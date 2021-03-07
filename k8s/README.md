## Create cluster with K3D
```shell
k3d cluster create meucluster --servers 1 --agents 2 -p 8080:30000@loadbalancer -p 8181:30001@loadbalancer -p 8282:30002@loadbalancer
```

## Create or apply - Replicaset / Deployment / Pod
```shell
kubectl apply -f ./api -f ./mongodb
```

## Install Prometheus
```shell
helm install prometheus prometheus-community/prometheus --values ./prometheus/values.yaml
```

## Install Grafana
```shell
helm install grafana grafana/grafana --values ./grafana/values.yaml
```

## Testing prometheus run while get products
```shell
while true; do curl http://localhost:8080/api/produto; sleep 1; done;
```

## Bind Prometheus Pod
Add annotation in deployment template
```yaml
annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
```

## Add data Resource
1. Open grafana
2. Click Data resources in cog menu
3. Click Add data resource
3. Add URL. When is used Kubernets use the service name
```shell
kubectl get services
```

