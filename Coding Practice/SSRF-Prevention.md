# Server-Side Request Forgery (SSRF) Prevention

## Overview

> Server-side request forgery (also known as SSRF) is a web security vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing.
>
> In a typical SSRF attack, the attacker might cause the server to make a connection to internal-only services within the organization's infrastructure. In other cases, they may be able to force the server to connect to arbitrary external systems, potentially leaking sensitive data such as authorization credentials.
>
> -- PortSwigger, [Server-side request forget (SSRF)](https://portswigger.net/web-security/ssrf)

## Best Practices

In general, validate user submitted URLs with allow-list if your use case requires them.

If arbitrary URLs are expected:

- validate the submitted URL string with a regular expression or URL parsing library to ensure it fits the expected URL format.
- use host, cluster, or VPC egress firewalls to block access to internal resources.
- validate the URL protocol against an allow-list (e.g., only expected web URLs? only allow http and https)
- validate user submitted authentication tokens using a regular expression (e.g., [a-zA-Z0-9]{20})
- Return only the information needed by the frontend. Don't return the raw HTTP response from the destination web server.
- Block access to internal resources (see section below for details).
- Ensure that the HTTP client is not passing internal credentials to external resources.
- Prevent the HTTP client from following redirections (HTTP 301, 302, etc.)

## Block Access to Internal Resources

When you need to make server-side requests based on a user-submitted URL, such as with webhooks, prevention centers around rejecting requests aimed at internal resources. There are a couple of checks that need to be performed on a submitted URL before making the server-side request.

### Step 1. Check for submission of non-public IP address

Determine if the submitted URL is a non-public IP address using a block-list. If the submission uses a domain name, resolve the domain's IP addresses (A and AAAA records) and then perform this check on each of them. URLs containing local, APIPA, or Private IP addresses should be rejected.

Local Address Ranges:

- 127.0.0.0 – 127.255.255.255

Private IP Address Ranges:

- 10.0.0.0 - 10.255.255.255
- 172.16.0.0 - 172.31.255.255
- 192.168.0.0 - 192.168.255.255

APIPA Address Range:

- 169.254.0.1 - 169.254.255.254

```
func validateIPs(ips []net.IP) (bool, error) {
  if len(ips) == 0 {
    return false, errors.New("IP not found")
  }
 
  for _, ip := range ips {
    if ip.To16() == nil && ip.To4() == nil {
      log.Errorf("IP: %v is not valid", ip)
      return false, errors.New("IP is not valid")
    }
    // IsPrivate reports whether ip is a private address, according to
    // RFC 1918 (IPv4 addresses) and RFC 4193 (IPv6 addresses).
    if ip.IsPrivate() {
      log.Errorf("IP address: %v is a private address", ip)
      return false, errors.New("IP address is a private address")
    }
    // checks Local Address Range of 127.0.0.0 - 127.255.255.255
    if ip.IsLoopback() {
      log.Errorf("IP address: %v is a local address", ip)
      return false, errors.New("IP address is a local address")
    }
    // checks APIPA Address Range of 169.254.0.0 - 169.254.255.255
    if ip.IsLinkLocalUnicast() {
      log.Errorf("IP address: %v is a link-local unicast address", ip)
      return false, errors.New("IP address is a link-local unicast address")
    }
  }
 
  return true, nil
}

```

```
# Source: https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L214-L240
def validate_localhost(addrs_info)
    local_ips = ["::", "0.0.0.0"]
    local_ips.concat(Socket.ip_address_list.map(&:ip_address))
 
    return if (local_ips & addrs_info.map(&:ip_address)).empty?
 
    raise BlockedUrlError, "Requests to localhost are not allowed"
end
 
def validate_loopback(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_loopback? || addr.ipv6_loopback? }
 
    raise BlockedUrlError, "Requests to loopback addresses are not allowed"
end
 
def validate_local_network(addrs_info)
    return unless addrs_info.any? { |addr| addr.ipv4_private? || addr.ipv6_sitelocal? || addr.ipv6_unique_local? }
 
    raise BlockedUrlError, "Requests to the local network are not allowed"
end
 
def validate_link_local(addrs_info)
    netmask = IPAddr.new('169.254.0.0/16')
    return unless addrs_info.any? { |addr| addr.ipv6_linklocal? || netmask.include?(addr.ip_address) }
 
    raise BlockedUrlError, "Requests to the link local network are not allowed"
end
```

### Step 2. Prevent secondary name resolution

If the URL submitted uses a domain name, you need to protect against DNS rebinding attacks by preserving the DNS resolution done in Step 1. It is common for an HTTP client library to perform its own DNS resolution when passed a URL. If an attacker changes the DNS record to a local, APIPA, or Private IP address between the resolution in Step 1 and the DNS resolution performed by the HTTP client, this validation performed in Step 1 can be bypassed. This is called a DNS Rebinding Attack.

The video below explains how SSRF can be combined with DNS Rebinding to bypass IP checks.

There are a couple of ways prevent this.

#### Option 1

The first is the method used by Gitlab in the video above (found [here](https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L22)), it to replace the URL's domain with a validated IP address from Step 1. Pass this URL to your HTTP client to prevent a secondary DNS resolution.

For example, change the user submitted URL:

> https://www.mywebsite.com/validate

to the following URL:

> https://55.26.115.78/validate

Simplified code example of how Gitlab validates a submitted URI and transforms it. The `protected_uri_with_hostname` returned is used by an HTTP client. 

```
# Source: https://gitlab.com/gitlab-org/gitlab-foss/-/blob/eabd80f72f4f7d8e19b26526aa1f44c43d78e8b3/lib/gitlab/url_blocker.rb#L22
require 'ipaddress'
 
# Expects Addressable::URI
def validate_uri(uri)
    address_info = get_address_info(uri)
    ip_address = address_info.first&.ip_address
    # Replace domain with resolved IP address    
    protected_uri_with_hostname = enforce_uri_hostname(ip_address, uri)
    protected_uri_with_hostname
end
 
def enforce_uri_hostname(ip_address, uri)
    return [uri, nil] unless ip_address && ip_address != uri.hostname
    new_uri = uri.dup
    new_uri.hostname = ip_address
    [new_uri, uri.hostname]
end
 
def get_address_info(uri)
    Addrinfo.getaddrinfo(uri.hostname, get_port(uri), nil, :STREAM).map do |addr|
        addr.ipv6_v4mapped? ? addr.ipv6_to_ipv4 : addr
    end
rescue SocketError
    raise BlockedUrlError, "Host cannot be resolved or invalid"
rescue ArgumentError => error
    raise unless error.message.include?('hostname too long')
    raise BlockedUrlError, "Host is too long (maximum is 1024 characters)"
end
```

This is effective, but you can run into issues if the destination web server is using virtual hosts. Without a domain to parse, the request will fail. 

#### Option 2

The second way to DNS Rebinding Attacks is to override the destination IP address in transport configuration of the HTTP Library. Changing it to the validated IP address from Step 1 will ensure the request goes to the validated destination. 

```
func sendGetRequest(webIP, host, scheme, path) (string, error) {
    dialer := &net.Dialer{
        Timeout:   10 * time.Second,
        KeepAlive: 10 * time.Second,
    }
 
    // provide a custom Transport.DialContext function
    // to force requests to use a specific destination IP address
    http.DefaultTransport.(*http.Transport).DialContext = func(ctx context.Context, network, addr string) (net.Conn, error) {
        port := ":80"
        if scheme == "https" {
            port = ":443"
        }
 
        addr = webIP + port
        return dialer.DialContext(ctx, network, addr)
    }
 
    webURL := scheme + "://" + host + path
    resp, err := http.Get(webURL)
    if err != nil {
        return "", fmt.Errorf("error when doing a GET request to publisher webURL [%s]: %v", webURL, err)
    }
}
```

```
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://1.2.3.4");
request.Host = "www.example.com";
var response = request.GetResponse();
```

