---
slug: consul-ui
id: 9mlvvplwymzx
type: challenge
title: Explore the Consul UI
teaser: Monitor Consul nodes & services via the UI.
notes:
- type: text
  contents: |-
    Consul is a service networking solution to automate network configurations, discover services, and enable secure connectivity across any cloud or runtime.

    We will first get to know Consul by exploring the UI.
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
Consul's UI allows you to view and interact with Consul via a graphical user interface, which can lower the barrier of entry for new users, and ease troubleshooting.
The Consul UI enables you to view all information about your Consul datacenter.

Access the Consul UI tab.

Consul UI Navigation is on the left hand side bar.

The "Nodes" section shows registered nodes, both Consul servers and clients. You can drill down further to find a nodes IP address, health checks, metadata, and other information.

The "Services" section shows registered services and their sidecar proxies. You can drill down further to find network topologies, assosciated intentions, the number of instances of an application, and other information.

As an example, click on the goapp service. The topology diagram should show a connection to the backend postgres service.

Additionally, you can view and update the following information through the Consul UI:

Consul Key-value pairs can be read/written in the "Key/Value" section

The "Intentions" section is where we manage network policy. We will cover that in a later section.

They are disabled in this demo, but Consul auth methods and Access Control List (ACL) tokens can also be managed in the UI.