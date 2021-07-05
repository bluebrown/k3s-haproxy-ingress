# K3S Haproxy Ingress Controller

## Start k3s

```shell
curl -sfL https://get.k3s.io \
    | sh -s - \
        --write-kubeconfig-mode "0644" \
        --disable traefik
```

If the service is already installed it will start again with the same command with which it was started the first time. This can also be modified in the `ExecStart` of /etc/systemd/system/k3s.service.

Otherwise provide the flags on the normal server command.

```shell
k3s server \
  --write-kubeconfig-mode "0644"    \
  --disable traefik
```

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

## Deploy Demo App

```shell
k create ns demo
k apply -f demo/ -n demo
k get all -n demo
curl <your-domain>
```
