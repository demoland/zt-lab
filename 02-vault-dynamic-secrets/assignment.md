---
slug: vault-dynamic-secrets
id: wladtgpnvjul
type: challenge
title: Vault dynamic secrets
teaser: Create least priviledge & just-in-time database credentials.
notes:
- type: text
  contents: |-
    Vault's database secrets engine generates short-lived credentials dynamically based on configured roles.

    Services no longer need hardcoded or static credentials. They can request the secret from Vault, and use Vault's leasing mechanism to keep those credentials renewed or to pull new ones after expiration.

    Since every service is accessing the database with unique credentials, this makes auditing easier when questionable data access is discovered. You can track it down to the specific instance of a service based on the SQL username.
- type: text
  contents: |-
    In this challenge we will configure Vault to create lease priviledged Postgres credentials

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/1-dbsecrets.png"
    width=800px height=400px>
tabs:
- title: Server
  type: terminal
  hostname: hashistack-server
- title: Vault UI
  type: service
  hostname: hashistack-server
  port: 8200
- title: Client 1
  type: terminal
  hostname: hashistack-client-1
- title: Client 2
  type: terminal
  hostname: hashistack-client-2
difficulty: basic
timelimit: 600
---
First, log back into Vault as our admin user on the "Server" tab.
```
vault login -method=userpass username=vault password=vault
```
List currently mounted secret engines.
```
vault secrets list
```
The PostgreSQL database secret engine has already been mounted and configured for you

To configure the secret engine we need to give Vault admin level credentials that enable creation and deletion of database users.
We also set the connection_url in this setup so Vault can locate the database.

Run the following command.
```
vault write database/config/my-postgresql-database \
  plugin_name=postgresql-database-plugin \
  allowed_roles="my-role, vault_go_demo" \
  connection_url="postgresql://{{username}}:{{password}}@postgres.service.consul:5432/vault_go_demo?sslmode=disable" \
  username="postgres" \
  password="password"
```
Next, we configure a secret engine role. In this step we can customize the creation_statements used by Vault for creating users.

Creation_statements are used to grant different permissions for users to read, write, etc. from databases.

We also set the database credential time-to-live and max time-to-live. After the TTL expires, the credential will be deleted from the database by Vault unless your applicaiton renews it. Credentials can be renewed up until the max-ttl expires.
```
vault write database/roles/vault_go_demo \
  db_name=my-postgresql-database \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
  ALTER USER \"{{name}}\" WITH SUPERUSER;" \
  default_ttl="1h" \
  max_ttl="24h"
```
Test the secret engine.
```
vault read database/creds/vault_go_demo
```
You should see Vault's dynamically generated username and password for PostreSQL. Every read to this secret engine will create a unique credential.

Let's learn how Vault can be used to encrypt customer PII data such as credit card or social security numbers.