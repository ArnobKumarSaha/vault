
## Install cert-manager
```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true
```

## Install vault
```bash
helm install kubevault oci://ghcr.io/appscode-charts/kubevault \
    --version v2024.1.31 \
    --namespace kubevault --create-namespace \
    --set-file global.license=/path/to/the/license.txt
```

## Make an issuer ready

```bash
kubectl apply -f server.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f policy.yaml
kubectl apply -f clusterissuer.yaml
```

## Utility commands

https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-cert-manager#configure-pki-secrets-engine

```bash
export VAULT_TOKEN=(kubectl vault root-token get vaultserver vault -n demo --value-only)
export VAULT_ADDR='http://127.0.0.1:8200'
```

```bash
vault secrets enable pki
```


```bash
vault write -field=certificate pki/root/generate/internal \
     common_name="example.com" \
     issuer_name="root-2023" \
     ttl=87600h > root_2023_ca.crt
```

```bash
vault read pki/config/issuers
```

```bash
vault delete pki/roles/example-dot-com
```

## Create DB
`kubectl apply -f mongo.yaml`