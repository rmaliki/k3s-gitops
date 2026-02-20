# Vault setup for `ledger-core`

Run these once (with a Vault token that can manage auth/policies).

## 1) Store secrets in Vault KV v2

```bash
export VAULT_ADDR="https://vault-hc.tools.datakushi.com"
# export VAULT_TOKEN="..."

vault kv put secret/ledger/core \
  QONTO_API_KEY="<your_qonto_api_key>" \
  QONTO_ORGANIZATION_ID="<your_org_id>"
```

## 2) Enable/configure Kubernetes auth (if not already enabled)

```bash
vault auth enable kubernetes || true

vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

> If you run this command outside a pod, provide the cluster CA and token reviewer JWT explicitly.

## 3) Policy for app read access

```hcl
# ledger-core-policy.hcl
path "secret/data/ledger/core" {
  capabilities = ["read"]
}
```

```bash
vault policy write ledger-core ledger-core-policy.hcl
```

## 4) Bind Vault role to Kubernetes ServiceAccount

```bash
vault write auth/kubernetes/role/ledger-core \
  bound_service_account_names=ledger-core \
  bound_service_account_namespaces=ledger-core \
  policies=ledger-core \
  ttl=24h
```

## 5) Prerequisite

Vault Agent Injector must be installed in the cluster for pod annotations to work.
