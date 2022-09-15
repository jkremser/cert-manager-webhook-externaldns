# cert-manager-webhook-externaldns

This project aims to provide [cert-manager](https://cert-manager.io) webhook plugin. This plugin on challenge request creates DNSEndpoint object, which is supposed to be picked by either [external-dns](https://github.com/kubernetes-sigs/external-dns) or CoreDNS [CRD plugin](https://github.com/k8gb-io/coredns-crd-plugin)

## Quick Start

```bash
# we assume k8s cluster at version 1.23 or higher
k3d cluster create --no-lb --k3s-arg "--disable=traefik,servicelb,metrics-server,local-storage@server:*" --image docker.io/rancher/k3s:v1.23.7-k3s1
```

```bash
# install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
```

```bash
# install the webhook
kubectl apply -f manifest/
```

## Example issuer
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: <enter your email address>
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-staging
    solvers:
    - dns01:
        webhook:
          groupName: acme.example.com
          solverName: externaldns 
          config:
            labels:
              cert-manager: true 
```
## external-dns variant
Run external-dns with source set to crd and optionally label filters

## coredns variant
Example `Corefile` for CoreDNS
```
zone example.net
k8s_crd {
  resources DNSEndpoint
  filter cert-manager=true
}
```

## Development

```bash
# after cloning the repo run
tilt up

# it assumes a k3s cluster to be up and running
```