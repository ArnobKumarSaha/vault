# Installation

## Install cert-manager
```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true
```

## Install KubeVault
```bash
helm install kubevault oci://ghcr.io/appscode-charts/kubevault \
    --version v2024.3.12 \
    --namespace kubevault --create-namespace \
    --set-file global.license=/path/to/the/license.txt
```

# PKI enabled VaultServer
```bash
kubectl apply -f vaultserver.yaml
```

```bash
export VAULT_TOKEN=(kubectl vault root-token get vaultserver vault -n demo --value-only)
OR
kubectl view-secret -n demo vault-keys -a
export VAULT_TOKEN=<copy-the-token-from-above-output>
export VAULT_ADDR='http://127.0.0.1:8200'

kubectl port-forward -n demo svc/vault 8200
vault status
```


## Create PKI Engine
```bash
kubectl apply -f pki/engine.yaml

# Ensure engine path
vault read <secretEngine.status.path>/config/urls

If the above command gives "No value found", Need to set one manually.

vault write <secretEngine.status.path>/config/urls \
    issuing_certificates="http://vault.demo:8200/v1/<secretEngine.status.path>/ca" \
    crl_distribution_points="http://vault.demo:8200/v1/<secretEngine.status.path>/crl"
```

## Create PKI Role
```bash
kubectl apply -f pki/role.yaml

# Ensure role path
vault list <secretEngine.status.path>/roles
```

# Make the issuer ready
```bash
kubectl apply -f pki/rolebinding.yaml 
kubectl apply -f pki/policy.yaml # Edit the paths in VaultPolicy before applying

# Ensure the policy
vault policy read vault-issuer

kubectl apply -f pki/policy-binding.yaml
kubectl apply -f issue/clusterissuer.yaml # Edit the paths in clusterissuer before applying
``` 

## Create DB
```bash
kubectl apply -f mongo.yaml

mongo --tls --tlsCAFile /var/run/mongodb/tls/ca.crt --tlsCertificateKeyFile /var/run/mongodb/tls/client.pem --authenticationMechanism MONGODB-X509 --authenticationDatabase='$external'
# -u "CN=root,OU=client,O=kubedb"
```


# Utility commands

https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-cert-manager#configure-pki-secrets-engine



```bash
vault secrets enable pki
vault read pki/config/issuers
vault delete pki/config/issuers
vault delete pki/roles/example-dot-com
```

```bash
vault write -field=certificate pki/root/generate/internal \
     common_name="example.com" \
     issuer_name="root-2023" \
     ttl=87600h > root_2023_ca.crt
```


```bash
vault write pki/roles/example-dot-com \
    allowed_domains=example-dot-com \
    allow_subdomains=true \
    max_ttl=72h \
    issuer_ref=82797184-6706-1c35-3172-ec6d39781865
```