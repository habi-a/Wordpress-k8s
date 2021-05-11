---
title: 'Better secret with Vault'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 4
summary: Kubernetes offers basic secret management, Hasicorp's vault can help you.
---

## Install vault with helm

For installing helm, see the previous section of this doc

add the Hashicorp helm repository and install vault
```
$> helm repo add hashicorp https://helm.releases.hashicorp.com
$> helm install vault hashicorp/vault --namespace=vault
```

Next go to the vault dashboard via traefik ingress route: `https://vault.etna.student`

Create the master token and key
Select 1 key share and 1 key threshold, then download the key

Next return to the CLI of your kube master
Unseal Vault running on the vault-0 pod
```
$> kubectl exec --namespace=vault vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
```

First, start an interactive shell session on the vault-0 pod.
```
$> kubectl exec --namespace=vault --stdin=true --tty=true vault-0 -- /bin/sh
```

Login with the root token when prompted.
```
$> vault login
```

Enable kv-v2 secrets at the path secret.
```
$> vault secrets enable -path=secret kv-v2
```

Create a secret at path secret/wordpress/config with a username and password.
```
$> vault kv put secret/wordpress/config username="static-user" password="static-password" root_password="static-root-password"
```

Enable the Kubernetes authentication method.
```
$> vault auth enable kubernetes
```

Configure the Kubernetes authentication method to use the service account token, the location of the Kubernetes host, and its certificate.
```
$> vault write auth/kubernetes/config \
        token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
        kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
        kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Write out the policy named `wordpress` that enables the read capability for secrets at path secret/data/wordpress/config.
```
$> vault policy write wordpress - <<EOF
path "secret/data/wordpress/config" {
  capabilities = ["read"]
}
EOF
```

Create a Kubernetes authentication role, named webapp, that connects the Kubernetes service account name and webapp policy.
```
$> vault write auth/kubernetes/role/wordpress \
        bound_service_account_names=vault \
        bound_service_account_namespaces=vault \
        policies=wordpress \
        ttl=24h
```

