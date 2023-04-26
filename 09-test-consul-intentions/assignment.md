---
slug: test-consul-intentions
id: vcc8lypmv1b4
type: challenge
title: Testing Consul Intentions
teaser: Create deny & allow rules for accessing Postgres.
notes:
- type: text
  contents: You can define a service-intentions configuration entry to create and
    manage intentions, as well as manage intentions through the Consul UI. You can
    also perform some intention-related tasks using the API and CLI commands.
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
Switch to the "Client 1" tab.

First, let's deny access from the webapp to the database. we define the following service-intention configuration as code.

This intention specifices that connections from the source service "goapp" will be denied to the backend "Name" postgres.
```
cat << EOF > ~/config-intentions-postgres-deny.hcl
Kind = "service-intentions"
Name = "postgres"
Sources = [
  {
    Name   = "goapp"
    Action = "deny"
  },
  # NOTE: a default catch-all based on the default ACL policy will apply to
  # unmatched connections and requests. Typically this will be DENY.
]
EOF
consul config write config-intentions-postgres-deny.hcl
```
Consul Connect/Envoy will not affect existing connections so we will need to restart the application for the changes to take effect.

Restart the application from the Client 1 tab.
```
systemctl restart goapp
```
You should see that the webapp failed to start.
Note: try adding a record to see a failure as the webpage may be cached.

Switch back to the "Client 1" tab.

Now, let's enable traffic between the webapp and postgres so our connection succeeds.
```
cat << EOF > ~/config-intentions-postgres-allow.hcl
Kind = "service-intentions"
Name = "postgres"
Sources = [
  {
    Name   = "goapp"
    Action = "allow"
  },
  # NOTE: a default catch-all based on the default ACL policy will apply to
  # unmatched connections and requests. Typically this will be DENY.
]
EOF
consul config write config-intentions-postgres-allow.hcl
```
You can see we've changed the Action to "allow".

Consul Connect/Envoy will not affect existing connections so we will need to restart the application for the changes to take effect.

You may need to restart a second time.
```
systemctl restart goapp
```
You should see the webapp successfully start now as it was able to connect and initalize the database.

Great work! With our application and database talking over a secure connection, we can now move on to securing human access to these resources.