---
slug: test-app
id: tlui9ygu2kiq
type: challenge
title: Test the application
teaser: Write an example record to the application.
notes:
- type: text
  contents: With our application integrated with Vault, we can now test creating a
    customer record.
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
- title: Client 1
  type: terminal
  hostname: hashistack-client-1
- title: Client 2
  type: terminal
  hostname: hashistack-client-2
difficulty: basic
timelimit: 600
---
Switch to the "Client 1" terminal and restart the application.

NOTE: You may need to run this command twice due to a DNS issue.
```
systemctl start goapp
```
Switch to the "App UI" tab.

Now test the App UI by adding a customer Record.

Navigate to the "Add Record" button and fill in example data. Once submitted, you will be redirected to the applications view of all detokenized data.

Next, switch to the "Database View" tab. You should see that your new record's SSN has been tokenized. This is what a logged in database user would see if inspecting Postgres.
If anyone where to compromise your database, they now would be unable to reverse this data.

Further Reading: Our application should be leveraging TLS certificates for secure connections.
We could have integrated it with the Vault PKI secrets engine to do so: https://learn.hashicorp.com/tutorials/vault/pki-engine