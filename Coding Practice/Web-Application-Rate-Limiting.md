# Secure Design Principles [Coding Practice]

<font size="-1">_Author: Andrew Luke - Dec. 2021_</font>

## Overview

- [HAProxy](#haproxy)
- [NGINX](#nginx)
- [Apache](#apache)
- [Envoy](#envoy)
- [node.js](#nodejs)
- [Golang](#golang)
- [Ruby](#ruby)
- [Python](#python)
  - [Flask](#flask)
  - [Django](#django)
- [ASP.NET](#aspnet)
- [Java](#java)

"Rate limiting is the process of controlling traffic rate from and to a server or component. It can be implemented on infrastructure as well as on an application level. Rate limiting can be based on (offending) IPs, on IP blacklists, on geolocation, etc." – [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Denial_of_Service_Cheat_Sheet.html#:~:text=Rate limiting is the process,blacklists%2C on geolocation%2C etc.)

Rate limiting is a way to defend your web application against Denial of Service (DoS) attacks and scraping of data. DoS is commonly thought of as overwhelming a target system with network traffic to the point it is no longer accessible to other end users, which is typically done with a distributed network of thousands or millions of systems (DDoS). There are other ways to DoS a web application, such as consuming available system resources by repeatedly calling resource intensive endpoints. For example, flooding requests to endpoints that handle complex computation or file system operations could be used to exhaust available workers file system resources. Another example is to abuse a web application endpoint that includes calls to another rate-limited service to that point that the rate-limited is violated. The target web application could be temporarily blocked from accessing the rate-limited service, denying that endpoint functionality to every end user. This is where request rate-limiting comes in. Rate-limiting can be used to restrict the rate of requests from a source IP for either specific endpoints or the entire web application. Actions taken on users that violate the rate-limits can vary from throttling to temporary or permanent IP blacklisting. Rate-limiting is also an effective way to defend against data scraping, vulnerability scanning, and brute forcing login forms. 

There are several rate limiting algorithms that [Google Solutions does a good job of summarizing](https://cloud.google.com/solutions/rate-limiting-strategies-techniques#techniques-enforcing-rate-limits):

- **Token bucket**: A [token bucket](https://wikipedia.org/wiki/Token_bucket) maintains a rolling and accumulating budget of usage as a balance of *tokens*. This technique recognizes that not all inputs to a service correspond 1:1 with requests. A token bucket adds tokens at some rate. When a service request is made, the service attempts to withdraw a token (decrementing the token count) to fulfill the request. If there are no tokens in the bucket, the service has reached its limit and responds with backpressure. For example, in a GraphQL service, a single request might result in multiple operations that are composed into a result. These operations may each take one token. This way, the service can keep track of the capacity that it needs to limit the use of, rather than tie the rate-limiting technique directly to requests.
- **Leaky bucket**: A [leaky bucket](https://wikipedia.org/wiki/Leaky_bucket) is similar to a token bucket, but the rate is limited by the amount that can drip or leak out of the bucket. This technique recognizes that the system has some degree of finite capacity to hold a request until the service can act on it; any extra simply spills over the edge and is discarded. This notion of buffer capacity (but not necessarily the use of leaky buckets) also applies to components adjacent to your service, such as load balancers and disk I/O buffers.
- **Fixed window**: Fixed-window limits—such as 3,000 requests per hour or 10 requests per day—are easy to state, but they are subject to spikes at the edges of the window, as available quota resets. Consider, for example, a limit of 3,000 requests per hour, which still allows for a spike of all 3,000 requests to be made in the first minute of the hour, which might overwhelm the service.
- **Sliding window**: Sliding windows have the benefits of a fixed window, but the rolling window of time smooths out bursts. Systems such as Redis facilitate this technique with expiring keys.

Below is a non-comprehensive list of rate limiting options for common types of web servers and web application frameworks. The options and algorithms used vary and are annotated for each. Keep in mind these only cover rate limiting at the web server and web application level. This guide does not cover CDN, WAF, and cloud load balancer solutions, which are more effective measures against DDoS attacks. 

Resources: 

- [OWASP API Security Top 10 - Lack of Resources and Rate Limiting](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa4-lack-of-resources-and-rate-limiting.md) 
- [Google Solutions - Rate-limiting strategies and techniques](https://cloud.google.com/solutions/rate-limiting-strategies-techniques)

## HAProxy

Sliding window code example for a 10 requests per second limit with a 100 kilobyte stick-table size. Requests that surpass this limit will return and HTTP 429 response:

```
frontend website
    bind :80
    stick-table  type ip  size 100k  expire 30s  store http_req_rate(1s)
    http-request track-sc0 src
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 10 }
    default_backend servers
```

Fixed Window code example for 1000 requests per day:

```
frontend website
    bind :80
    stick-table type ip size 100k expire 24h store http_req_cnt
    http-request track-sc0 src
    http-request deny deny_status 429 if { sc_http_req_cnt(0) gt 1000 }
    default_backend servers
```

For rate limiting specific webapp endpoints, create a file that lists the endpoints and their rate limits named 'rates.map' in /etc/haproxy:

```
/search  10
/save  20
/update  30
```

Then reference the list in the HAProxy config:

```
frontend website
    bind :80
    stick-table  type binary  len 8  size 100k  expire 10s  store http_req_rate(10s)
 
    # Track client by base32+src (Host header + URL path + src IP)
    http-request track-sc0 base32+src
 
    # Check map file to get rate limit for path
    http-request set-var(req.rate_limit)  path,map_beg(/etc/haproxy/rates.map,20)
 
    # Client's request rate is tracked
    http-request set-var(req.request_rate)  base32+src,table_http_req_rate()
 
    # Subtract the current request rate from the limit
    # If less than zero, set rate_abuse to true
    acl rate_abuse var(req.rate_limit),sub(req.request_rate) lt 0  
 
    # Deny if rate abuse
    http-request deny deny_status 429 if rate_abuse
    default_backend servers
```

Source: https://www.haproxy.com/blog/four-examples-of-haproxy-rate-limiting/

## NGINX

NGINX uses the leaky bucket rate limiting algorithm. Code example for a 10 requests per second limit with a 10 megabyte zone size (16,000 IP addresses per megabyte). Bursts can be allowed by adding a burst limit (20 here):

```
http {
    #...
 
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
 
    server {
        #...
 
        location /search/ {
            limit_req zone=one burst=20;
        }
    }
}
```

Blog post: https://www.nginx.com/blog/rate-limiting-nginx/
Webinar: https://www.nginx.com/resources/webinars/rate-limiting-nginx/
Documentation: https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-http/

## Apache

The mod_ratelimit Apache module allows developers to specify rate limits on a location by location basis. Rather than limiting the rate by requests per second, mod_ratelimit instead limits bandwidth in kilobytes per second based on HTTP response.

Code example limiting bandwidth to 400 KiB/s with a burst rate of 512 KiB/s:

```
<Location "/search">
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 400
    SetEnv rate-initial-burst 512
</Location>
```

Source: https://documentation.help/httpd-2.4/mod_ratelimit.html

## Envoy

In order to implement rate limiting with Envoy, you need to point it to a gRPC rate limiting service. Lyft has an open source [Golang rate limiting service](https://github.com/envoyproxy/ratelimit) that utilizes Redis to track rates. Example for limiting individual users to 10 requests per second:

```
domain: edge_proxy_per_ip
descriptors:
  - key: remote_address
    rate_limit:
      unit: second
      requests_per_unit: 10
```

Service: https://github.com/envoyproxy/ratelimit

Documentation: https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting

## node.js

[Express-rate-limit](https://www.npmjs.com/package/express-rate-limit) offers application-wide, sliding window rate limiting and per-endpoint rate limiting of Express web applications. It uses an in-memory store by default, but also supports Redis, Memached, and Mongo.

Code example for rate limit of 10 requests per second:

```
const rateLimit = require("express-rate-limit");
  
const limiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minutes
  max: 600, // limit each IP to 600 requests per windowMs
  headers: false // don't send X-RateLimit-Limit, X-RateLimit-Remaining, and Retry-After headers in response
});
  
//  apply to all requests
app.use(limiter);
 
// only apply to requests that begin with /search/
app.use("/search/", limiter);
```

Library: https://www.npmjs.com/package/express-rate-limit

## Golang

[tollbooth](https://github.com/didip/tollbooth) offers per-handler rate limiting middleware. Shims for common web frameworks are linked in the README. Code example for rate limit of 10 requests per second:

```
package main
 
import (
    "github.com/didip/tollbooth"
    "net/http"
)
 
func HelloHandler(w http.ResponseWriter, req *http.Request) {
    w.Write([]byte("Hello, World!"))
}
 
func main() {
    // Create a request limiter per handler.
    http.Handle("/", tollbooth.LimitFuncHandler(tollbooth.NewLimiter(10, nil), HelloHandler))
    http.ListenAndServe(":12345", nil)
}
```

Library: https://github.com/didip/tollbooth
Blog Post on rolling your own per-user rate limiter in Golang: https://www.alexedwards.net/blog/how-to-rate-limit-http-requests

## Ruby

Kickstarter's rack-attack library uses fixed window rate throttling
Code example for universal rate limiting to 10 requests per second:

```
Rack::Attack.throttle('request per ip', limit: 50, period: 5) do |request|
  request.ip
end
```

Code example for rate limiting /search to 10 requests per second:

```
Rack::Attack.throttle "logins/ip", limit: 2, period: 1 do |req|
  req.path == "/search" && req.ip
end
```

Library: https://github.com/rack/rack-attack
Example: https://github.com/rack/rack-attack/blob/master/examples/rack_attack.rb

## Python

### Flask

Flask-limiter supports fixed window and sliding window rate limiting. Code example for rate limiting /search to 10 requests per second and universally to 100 per minute:

```
from flask import Flask
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
 
app = Flask(__name__)
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["100 per minute"],
)
 
@app.route("/search")
@limiter.limit("10 per second")
def slow():
    return "24"
```

Library: https://github.com/alisaifee/flask-limiter
Documentation: https://flask-limiter.readthedocs.io/en/stable/

### Django

Code example for rate limiting /search to 10 requests per second:

```
from ratelimit.decorators import ratelimit
 
@ratelimit(key='ip', rate='10/s')
def search(request):
    # ...
```

Library: https://github.com/jsocol/django-ratelimit
Documentation: https://django-ratelimit.readthedocs.io/en/stable/

# ASP.NET

The AspNetCoreRateLimit library offers rate limiting middleware to impliment into your web API/MVC application.
Code example:

```
public void ConfigureServices(IServiceCollection services)
{
    // needed to load configuration from appsettings.json
    services.AddOptions();
 
    // needed to store rate limit counters and ip rules
    services.AddMemoryCache();
 
    //load general configuration from appsettings.json
    services.Configure<IpRateLimitOptions>(Configuration.GetSection("IpRateLimiting"));
 
    // inject counter and rules stores
    services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
    services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
 
    // Add framework services.
    services.AddMvc();
 
    // https://github.com/aspnet/Hosting/issues/793
    // the IHttpContextAccessor service is not registered by default.
    // the clientId/clientIp resolvers use it.
    services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
 
    // configuration (resolvers, counter key builders)
    services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
}
 
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseIpRateLimiting();
 
    app.UseMvc();
}
```

In the appsettings.json file:

```
"IpRateLimiting": {
    "EnableEndpointRateLimiting": false, // limits apply globally
    "StackBlockedRequests": false,
    "RealIpHeader": "X-Real-IP",
    "ClientIdHeader": "X-ClientId",
    "HttpStatusCode": 429,
    "IpWhitelist": [ "127.0.0.1", "::1/10", "192.168.0.0/24" ],
    "EndpointWhitelist": [ "get:/api/license", "*:/api/status" ],
    "ClientWhitelist": [ "dev-id-1", "dev-id-2" ],
    "GeneralRules": [
        {
        "Endpoint": "*",
        "Period": "1m",
        "Limit": 100
        },
        {
        "Endpoint": "*:/search/*",
        "Period": "1s",
        "Limit": 10
        }
    ]
}
```

Library: https://github.com/stefanprodan/AspNetCoreRateLimit
Setup: https://github.com/stefanprodan/AspNetCoreRateLimit/wiki/IpRateLimitMiddleware#setup

## Java

The resilience4j library offers a lot of functionality, including rate limiting. Implementation is a bit more complex and is dependent on web framework used.
Library: https://github.com/resilience4j/resilience4j
Documentation: https://resilience4j.readme.io/docs/ratelimiter

## Contributors

* Andrew Luke - December 2021