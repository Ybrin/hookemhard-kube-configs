# Hook'em Hard kubernetes configuration files

## MySQL

### Production

Initial Password: `z6e9nCpOwvxCLoO1`

### Development

Initial Password: `ykd3kJ7ABb29pluc`

### Staging

Initial Password `elAcEq3eDDxmibyK`

## Kubernetes Setup

### helm

For dependency management.

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
```

### ingress-nginx

Should be installed like that:

```bash
helm install ingress-nginx stable/nginx-ingress --set rbac.create=true --set controller.service.nodePorts.http=30080 --set controller.service.nodePorts.https=30443 --set controller.service.type=NodePort
```

### cert-manager

```bash
kubectl apply \
    -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.6/deploy/manifests/00-crds.yaml

kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.14.1/cert-manager-legacy.crds.yaml

kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.14.1

kubectl apply -f clusterissuer.yaml
```

### Prometheus and Grafana

```bash
kubectl create namespace monitoring

helm install prometheus stable/prometheus \
      --namespace monitoring \
      --set server.persistentVolume.enabled=false \
      --set alertmanager.persistentVolume.enabled=false

kubectl apply -f monitoring/grafana/config.yaml
helm install grafana stable/grafana \
    -f monitoring/grafana/values.yaml \
    --namespace monitoring
```

```
Prometheus:

Get the PushGateway URL by running these commands in the same shell:
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $POD_NAME 9091

Grafana:

1. Get your 'admin' user password by running:

   kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

2. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   grafana.monitoring.svc.cluster.local

   Get the Grafana URL to visit by running these commands in the same shell:

     export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana" -o jsonpath="{.items[0].metadata.name}")
     kubectl --namespace monitoring port-forward $POD_NAME 3000

3. Login with the password from step 1 and the username: admin
```

Production password: `d5TXPjkzcHgJPhzWNFTmXM1lFuQDbwoCdeefvwfX`
Staging password: `AvK1jA7DhOmyUZdmXkRKLesXAGeSvNetmfacRzOB`
Development password: `GjBcigDucksZOWo7lMSoS4Ag5LMMCb9qD5IsNS9g`

### RethinkDB

To access the dashboard, run the following command:

```bash
kubectl -n rethinkdb port-forward rethinkdb-master-0 5000:8080
```

and use localhost:5000 to access it.
