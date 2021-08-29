# Architecture and Roadmap

## Architecture

The `kube-vip-dns` "controller" is a watcher of Kubernetes services and it's purpose is to update an external DNS provider with with a name from the service as a subdomain (where possible) and the IP address from the `service.spec.loadbalancerIP`. 

The `kube-vip-dns` "controller" will start and manage CRDs with the type DNS, it will also spawn a watcher as a `goroutine` for every provider that is enabled.

A `secret` will be required for the credentials/tokens for the external DNS providers in order for DNS information to be propogated to the various DNS providers.

## Roadmap

### PoC phase 1

- Basic controller with DNS CRD
- bind9 support

## PoC phase 2 

- Cloudflare support
