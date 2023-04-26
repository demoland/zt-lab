---
slug: boundary-ui
id: lx7uw5vlnfk8
type: challenge
title: Explore the Boundary UI
teaser: Get familiar with Boundary projects and targets.
notes:
- type: text
  contents: |-
    Boundary supports CLI, API, and a GUI for interacting with the service. In this exercise you will explore the GUI and find available targets to securely access.

    Boundaries domain model is explained in detail here -> https://www.boundaryproject.io/docs/concepts/domain-model
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
- title: Boundary UI
  type: service
  hostname: hashistack-server
  port: 9200
- title: Client 1
  type: terminal
  hostname: hashistack-client-1
- title: Client 2
  type: terminal
  hostname: hashistack-client-2
difficulty: basic
timelimit: 600
---
Login to Boundary in the "Boundary UI" tab
```
username: admin
password: password
```
Boundary's top level domain unit is called an "Org" or "Organization" (they are also called Scopes).

Scopes are a foundational part of Boundary. They allow users to partition resources and assign ownership of resources to principals.

Find more details at https://learn.hashicorp.com/tutorials/boundary/manage-scopes?in=boundary/basic-administration

First, navigate to the corp_one Org. This is where our resources were defined and configured in the last challenge.

Within Scopes are Projects. Projects contain Targets and Host Catalogs (the actual resources/VMs/databases/APIs we are securing)

Click on "core_infra". Then click "Targets" on the left hand side.

Copy the Postgres_server Target. We will access this in the next challenge so save the value.
It should look like:
```
ttcp_tGJJUmCsrP
```

Set an environment variable on the "Server" tab with your copied target id.
```
#Copy the postgres_server target from the corp_one org
export POSTGRES_TARGET=
```
We can now move on to testing Boundary for secure access.