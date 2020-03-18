# Tools and Automation

## Vault Secret Fetcher

A utility for automatically fetching secrets from Vault in IE infra. Written in Go and compiled statically to ensure no external dependencies are needed, especially in scratch and distroless images. This tool's target audience are teams looking to have maximum compatibility with minimal developer effort in a large infrastructure already deployed in Kubernetes.