# K3S Haproxy Ingress Controller

## Create Secret TLS Objects

```shell
k create secret tls my-tls-object --key privkey.pem --cert fullchain.pem --dry-run=client -o yaml \
    | tee controller/prep/cert.secret.yaml
```

## Create Controller

```shell
k apply -f controller/prep/
k apply -f controller/ingress-controller/
```