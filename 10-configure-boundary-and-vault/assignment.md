---
slug: configure-boundary-and-vault
id: z29yjel01uza
type: challenge
title: Configure Boundary and Vault
teaser: Configure Vault to issue credentials for Boundary.
notes:
- type: text
  contents: |-
    Boundary provides a secure way to access hosts and critical systems without having to manage credentials or expose your network.

    Boundary 0.4.0 adds a Vault integration for the brokering of Vault secrets to Boundary clients via the command line and desktop clients for use in their Boundary sessions.

    This feature enables Boundary as a credential broker for infrastructure targets by binding credentials with user sessions, and surfacing those credentials during session initialization.
- type: text
  contents: |-
    We will configure Boundary to retrieve dynamic credentials from Vault in this challenge.

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/5-boundary-vault.png"
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
timelimit: 600
---
Login to Vault on using the "Server" tab
```
vault login -method=userpass username=vault password=vault
```
Next, we will configure Boundary using HashiCorp Terraform.

Terraform is an open-source infrastructure as code software tool that enables you to safely and predictably create, change, and improve infrastructure.

Change into the terraform-boundary directory where our Terraform config is stored
```
cd terraform-boundary/
cat boundary-controller-policy.hcl
```
Boundary needs to lookup, renew, and revoke tokens and leases in order to broker credentials from Vault properly.

Next, we will create a Vault token for the Boundary Server to use.

```
vault policy write boundary-controller boundary-controller-policy.hcl
vault token create \
  -no-default-policy=true \
  -policy="boundary-controller" \
  -policy="go-app" \
  -orphan=true \
  -period=20m \
  -renewable=true
```

Save the "token" from the above output. You will pass this value to Terraform in the "apply" step below.
```
#example: Save this from above. You will pass it to Terraform apply.
s.5DyL8vhy0AA3ke42WbWG4r5A
```

Initialize terraform.
```
#feed token to terraform
terraform init;
```
Once satisified, apply the configuration. You will need to pass the Vault token from above to the Terraform CLI command.
```
terraform apply --auto-approve;
```

Once Terraform successfully configures Boundary, we can move on to the next challenge.
```
apply complete! Resources: 20 added, 0 changed, 0 destroyed.
```