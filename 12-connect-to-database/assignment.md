---
slug: connect-to-database
id: kpodwawau3po
type: challenge
title: Test Boundary secure access management.
teaser: Leverage Boundary to securely connect to PostgreSQL.
notes:
- type: text
  contents: We will now simulate what short-lived database access to PostgreSQL would
    look like for an end user.
- type: text
  contents: |-
    In this challenge, you will securely access PostgreSQL via Boundary.
    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/6-access.png" width=800px height=400px>
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
- title: Terraform Editor
  type: code
  hostname: hashistack-server
  path: /root/terraform-boundary/main.tf
difficulty: basic
timelimit: 20000
---
Login to Boundary via the CLI on the "Server" tab.
```
#Admin password is "password"
boundary authenticate password -auth-method-id ampw_1234567890 -login-name admin
```
Copy the resulting token to an env variable
```
export BOUNDARY_TOKEN=at_vmQqzJ03lB_s12m.....
```

If you haven't in the prior challenge, copy the postgres_server target from the corp_one org and set an environment variable.

```
export POSTGRES_TARGET=ttcp_uJw5GWOygZ
```
Next authorize a session to acess Postgres. This will create an authorization token and return dynamic database credentials from Vault.

```
boundary targets authorize-session -id  $POSTGRES_TARGET -token env://BOUNDARY_TOKEN
```
The Boundary CLI will automatically use the above information and the returned dynamic database credentials.
We can now connect to the database!

You may need to run this twice as it can error on the first call.
```
boundary connect postgres -target-id $POSTGRES_TARGET --dbname vault_go_demo -token env://BOUNDARY_TOKEN
```
Test a few database commands!
```
\l
\du
\q
SELECT * FROM vault_go_demo;
```
you can also test SSH'ing to the database host via Boundary. Switch back to the Boundary UI and copy the "ssh_server" target.

```
# SSH: ssh_server target (client2)
export SSH_TARGET=
```
Then connect to the host.
You may need to run this twice as it can error on the first call.
```
boundary connect ssh -target-id $SSH_TARGET -token=env://BOUNDARY_TOKEN
```

Congratulations! You've successfully learned how to secure application passwords, customer data, networking between services, and access to infrastructure resources!

To continue on your journey, please refer to the following docs for more information on use-cases, deployments, etc.

Vault: https://www.vaultproject.io/docs

Consul: https://www.consul.io/docs

Boundary: https://www.boundaryproject.io/docs