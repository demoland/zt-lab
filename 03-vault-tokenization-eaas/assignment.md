---
slug: vault-tokenization-eaas
id: be1zzr2rkt6g
type: challenge
title: Vault Transform secret engine (Tokenization/EaaS)
teaser: Protect critical customer data.
notes:
- type: text
  contents: |-
    The Transform secrets engine handles secure data transformation and tokenization against provided secrets.

    Transformation methods encompass NIST vetted cryptographic standards such as format-preserving encryption (FPE) via FF3-1 to encode your secrets while maintaining the data format and length.

    In addition, it can also perform pseudonymous transformations of the data through other means, such as masking.
- type: text
  contents: |-
    Our application will send plaintext customer data to Vault to be encrypted and returned before writing to Postgres.

    <img src="https://github.com/Andrew-Klaas/instruqt-zt-lab/raw/main/assets/diagrams/2-tokenization.png"
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
The Transform secret engine has already been mounted for you
```
vault secrets list
```
We first need to configure the secret engine to use tokenization.
Our golang application will be leveraging tokenization to protect customer data and meet PCI compliance of non-repudiation (Non reversible data).

Lets configure a named role in Vault for our secret engine. We also declare a transformation of name "ssn" we will define that in the next step.
```
vault write transform/role/vault_go_demo transformations=ssn
```
"Transformations" contain information about the type of data transformation that we want to perform, the template that it should use for value detection, and other transformation-specific values such as the tweak source or the masking characters to use.
In this case, we are using tokenization instead of format preserving encryption so the configuration is more simple. (No special formatting required)

Define the transformation.
```
vault write transform/transformations/tokenization/ssn \
  allowed_roles=vault_go_demo \
  max_ttl=24h
```
Now we can test tokenizing (called "encoding") a value.
```
export ENCODED_VAULT=$(vault write -format=json \
  transform/encode/vault_go_demo value=1111-2222-3333-4444 \
  transformation=ssn \
  | jq -r '.data.encoded_value')

echo $ENCODED_VAULT
```
Next, decode the tokenized value from above.
```
vault write transform/decode/vault_go_demo value="${ENCODED_VAULT}" transformation=ssn
```
You should see the original value in plaintext.

Next, we will test our web application.