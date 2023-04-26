---
slug: define-consul-intentions
id: rf6iouyrjwls
type: challenge
title: Defining Consul Intentions (Network Policy)
teaser: Define access control for services via Consul Connect.
notes:
- type: text
  contents: |-
    Consul Intentions define access control for services via Connect and are used to control which services may establish connections or make requests.

    Intentions are enforced on inbound connections or requests by the proxy or within a natively integrated application.

    Depending upon the protocol in use by the destination service, you can define intentions to control Connect traffic authorization either at networking layer 4 (e.g. TCP) and application layer 7 (e.g. HTTP)
- type: text
  contents: |-
    In this challenge, we configure Consul Service Mesh to connect and authorize connections between services

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/4-intention.png"
    width=800px height=400px>
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

Let's look at our webapp and database service definitions to understand how Connect is configured.

Switch to the "Client 1" tab. We will review the service configuration again with a focus on connect.
```
cd /etc/consul.d/
cat goapp.hcl
```
The Connect stanza holds configuration for this service's sidecar proxy.

First, note the sidecar stanza within the connect stanza.

```
connect {
  sidecar_service {
    proxy {
      upstreams = [
        {
          destination_name = "postgres"
          local_bind_port  = 5432
        }
      ]
    }
  }
}
```

Connect proxies are typically deployed as "sidecars" that run on the same node as the service instance that they handle traffic for. They might be on the same VM or running as a separate container in the same network namespace.
In this case, we will be leveraging Envoy as the sidecar proxy.

Next we define proxy "upstreams". These upstreams determine which backend services this local webapp/proxy is able to direct traffic too.
Within the upstreams stanza we define the destination service (regsistred in Consul) and a local port to bind too.

Then our service will connect to that configured local port instead of a remote address.
Note: Consul also supports Transparent Proxying - https://www.consul.io/docs/connect/transparent-proxy

In our frontend webapp's case, we configured the proxy to direct traffic to the backend postgres database on the local port 5432.

Next, switch to the "Client 2" tab
```
cd /etc/consul.d/
cat postgres.hcl
```
This backend service does not have any upstreams, so the configuration is a bit more simple.

```
connect {
sidecar_service {
  proxy {
      config {
        protocol = "tcp"
      }
    }
  }
}
```
The only real difference is that we specified the protocol. Consul also supports http/http2/tcp/gRPC.

With an understanding of our sidecar/connect configurations. Let's now test allowing and denying traffic.