---
slug: vault-aws-authentication
id: yunk3e8nwajx
type: challenge
title: Vault's AWS authentication method
teaser: Authenticate applications to Vault using AWS IAM.
notes:
- type: text
  contents: |-
    "Secure Introduction" or "how to introduce an authentication token to applications" is often one of the most difficult challenges in application & infrastructure deployments.

    Auth methods are components in Vault that perform authentication and are responsible for assigning identity and a set of policies to a user.
    Using the right auth method for the application's underlying platform/cloud helps us solve secure introduction.

    In this lab, we will use the AWS auth method to authenticate a Golang application so it can retrieve database credentials from Vault.
- type: text
  contents: |-
    This diagram shows our overall lab archtecture. There are 3 nodes: Server, Client-1, and Client-2.

    Client-1 contains our Go application. Client-2 hosts a Postgres Database. The Server contains the Consul, Vault, and Boundary Servers.

    We will first work on authenticating our application to Vault so it can retrieve a login token.

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/0-auth.png" width=800px height=400px>
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
- title: Goapp - Vault code
  type: code
  hostname: hashistack-server
  path: /tmp/aws-vault-go-demo-tokenization/config/db.go
difficulty: basic
timelimit: 600
---
Make sure you are on the server tab.

First, we need to authenticate to Vault as an admin user. This will set your vault token at "~/.vault-token".
```
vault login -method=userpass username=vault password=vault
```
Once authenticated, view currently mounted auth methods with the following command.
```
vault auth list
```
NOTE: the AWS auth method has already been mounted for you. You would use this command to mount it yourself.
```
#The AWS Secret Engine has already been mounted for you. This will return a 400 error.
vault auth enable aws
```

The AWS auth method's documentation can be found here: https://www.vaultproject.io/docs/auth/aws

Next, read the AWS auth mount config.
```
vault read auth/aws/config/client
```
The mount's already been configured for you as well.

To do so in your own environment, you need to configure the auth method with suitable AWS credentials to perform actions on IAM Users.
Our recommended AWS IAM policy is here: https://www.vaultproject.io/docs/auth/aws#recommend

You DO NOT need to run this step.
```
vault write auth/aws/config/client \
  access_key=$AWS_ACCESS_KEY_ID \
  secret_key=$AWS_SECRET_ACCESS_KEY
```
Next, configure a role for our application. The argument are explained in detail here: https://www.vaultproject.io/api/auth/aws#create-role

A role can be used to set time-to-lives for tokens, associate policies for Vault access, and set parameters on allowed IAM prinicpal ARNs.
```
vault write auth/aws/role/my-role-iam \
  auth_type=iam \
  bound_iam_principal_arn=arn:aws:iam::$AWS_ACCOUNT_ID:* \
  policies=go-app \
  token_ttl=30m \
  token_max_ttl=30m
```
Check your created role.
```
vault read auth/aws/role/my-role-iam
```
Test the login via vault CLI.

```
vault login -method=aws role=my-role-iam
```
You should see a new token created with the default and go-app policies attached.

How does our application perform this login? Vault programming libraries allow you to natively integrate your application for improved security.

By having your applciation call Vault directly, you can ensure that credentials only live in memory and not on disk or envirnoment variables.

Lets take at the "Goapp - Vault code" tab. (Click on the file on the left hand side)

The "AWSLogin" function at line 91 contains the code used by our application to login to Vault's AWS auth method.

Next, let's learn how to create short lived database credentials.