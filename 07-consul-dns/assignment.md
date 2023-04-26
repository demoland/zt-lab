---
slug: consul-dns
id: xkdoddbbe8ao
type: challenge
title: Testing Consul DNS
teaser: Use Consul DNS to locate services.
notes:
- type: text
  contents: |-
    One of the primary query interfaces for Consul is DNS. The DNS interface allows applications to make use of service discovery without any high-touch integration with Consul.

    For example, instead of making HTTP API requests to Consul, a host can use the DNS server directly via name lookups like redis.service.us-east-1.consul. This query automatically translates to a lookup of nodes that provide the redis service, are located in the us-east-1 datacenter, and have no failing health checks. It's that simple!
tabs:
- title: Server
  type: terminal
  hostname: hashistack-server
- title: App UI
  type: service
  hostname: hashistack-client-1
  port: 9090
- title: Vault UI
  type: service
  hostname: hashistack-server
  port: 8200
- title: Consul UI
  type: service
  hostname: hashistack-server
  port: 8500
- title: Client 1
  type: terminal
  hostname: hashistack-client-1
- title: Client 2
  type: terminal
  hostname: hashistack-client-2
difficulty: basic
timelimit: 600
---
Switch to the Client 1 tab.

We've already configured the nodes to leverage Consul for resolving any "*.consul" domains.

Use the dig command to find our postgres database registered in Consul.
```
dig postgres.service.consul
```
You should see the IP address of the postgres database returned!

Example:
```
;; ANSWER SECTION:
postgres.service.consul. 0      IN      A       10.132.0.184
```