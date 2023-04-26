---
slug: consul-services
id: zamgwiwnsyjv
type: challenge
title: Defining Consul Services
teaser: Create a simple service definition to declare the availability of a service
  in the Consul catalog.
notes:
- type: text
  contents: |-
    One of the major use cases for Consul is called "service discovery". Consul provides a DNS interface that downstream services can use to find the IP addresses of their upstream dependencies.
    Consul knows where these services are located because each service registers with its local Consul client.

    Operators can register services manually, configuration management tools can register services when they are deployed, or container orchestration platforms can register services automatically via integrations.
- type: text
  contents: |-
    In this challenge we will leverage Consul for registering our application and database services.

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/3-connect.png"
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
Switch to the "Client 1" terminal. Let's check the Consul Agent Config first.
```
cd /etc/consul.d/
cat consul.hcl
```
This consul agent is running in "client" mode. The configuration is used to set several options such as which address to bind to, where the Consul servers are that we want to join, and other info like log levels.
```
datacenter = "dc1"
retry_join = ["hashistack-server"]
retry_interval = ["5s"]
data_dir = "/tmp/consul"
server = false
log_level = "DEBUG"
node_name = "hashistack-client-1"
client_addr = "10.132.0.181 127.0.0.1"
bind_addr = "10.132.0.181"
ui = true
ports {
  grpc = 8502
}
```
Next lets check our webapp's service definition.
```
cat goapp.hcl
```
This file is used to define the name/ports of the service we want Consul to register & monitor, how to configure a sidecar proxy, health checks, etc.

We will discuss the "connect" stanza in more detail in the next section.
```
service {
  name = "goapp"
  tags = [ "goapp" ]
  port = 9090
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
}
```
Switch to the Client 2 terminal to check our postgres service definition.
```
cd /etc/consul.d/
cat postgres.hcl
```
We've included an example health check under the "check" stanza. Consul will run a tcp check against the local postgres database on 10 second intervals.
You can also configure Consul to execute a script or perform HTTP checks as well.
```
service {
  name = "postgres"
  tags = [ "postgres" ]
  port = 5432
  connect {
    sidecar_service {
      proxy {
        config {
          protocol = "tcp"
        }
      }
    }
  }
  check {
    id       = "postgresql-check"
    tcp      = "127.0.0.1:5432"
    interval = "10s"
  }
}
```
You can view all services registered in Consul via the following command.
```
consul catalog services
```