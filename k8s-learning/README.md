## Required

1. Install Docker
2. Install Node
3. Install Robot3T

## No SQL - MongoDB

1. Create a MongoDB with command:
```shell
docker run --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=root mongo
```
2. Open Robo3T and create our connection

## Start server

1. Open src and install packages
```shell
npm i
```

4. Start server
```shell
npm run server
```
or
```shell
node ./app.js
```

# Create Docker Image

```dockerfile
FROM node:14.15.5-alpine3.13

WORKDIR /app

COPY package*.json ./

RUN npm install --silent

COPY . .

EXPOSE 8080

CMD ["node", "./app.js"]

```

## Build Image
```shell
docker build -t andremorita/kubernets-api:v1 .
```

## Publish your image
```shell
docker login
```

```shell
docker push andremorita/kubernets-api:v1
```

## Install kubectl 
Example install mac using homebrew
```shell
brew install kubernetes-cli
```

Check installing
```shell
kubectl version --client
```

kubectl cluster-info

## Install K3S
Example install Mac with homeboew
```shell
brew install k3d
```

## Create cluster with K3D
```shell
k3d cluster create
```
List clusters

```shell
kubectl cluster-info

kubectl get nodes
```

Create without load balance
```shell
k3d cluster create meucluster --no-lb

k3d cluster list

k3d cluster delete meucluster 
```

```shell
k3d cluster create meucluster --servers 2 --agents 3
```

## Create pod.yml
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: meupod
spec:
  containers:
    - name: nginx-color
      image: kubedevio/nginx-color:blue
```

## Create Object
```shell
kubectl create -f ./pod.yaml

kubectl get pods

kubectl describe pod meupod

kubectl port-forward pod/meupod 8081:80

kubectl apply -f ./pod.yaml
```

Commands:
List
```shell
kubectl api-resources
```
```shell
kubectl get pods -l app=nginx-color
```


## Create Replicaset
```shell
kubectl apply -f ./replicaset.yaml
```

## Create cluster with bind NodePort
```shell
k3d cluster create meucluster --servers 1 --agents 2 -p 8081:30000@loadbalancer
```

## Prometeus 

Verifique se o MagicServer está instalado executando o comando abaixo. O k3d já vem com o Magic server no kind e k8s precisa instalar manualmente
```shell
kubectl top nodes
```

## Install Helme - Package manager the kubernets
Example Mac
```shell
brew install helm
```

## Recriar cluster
```shell
k3d cluster create meucluster --servers 1 --agents 2 -p 8080:30000@loadbalancer -p 8181:30001@loadbalancer -p 8282:30002@loadbalancer
```
```shell
kubectl apply -f ./k8s/api -f ./k8s/mongodb  
```

Add Prometheus
```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Create Prometheus file
```shell
helm show values prometheus-community/prometheus > prometheus.yaml
```

Create simples Prometheus values
```yaml
# Não instala o Alert Manager
alertmanager:
  enabled: false
# Não instala o Push Gateway
pushgateway:  
  enabled: false  
# Não instalar o State Metrics
kubeStateMetrics:
  enabled: false

server:
  global:
    ## Intervalo entre os scrapes     
    scrape_interval: 5s
    ## Tempo de timeout para um scrape
    scrape_timeout: 5s 

  persistentVolume:
    ## Desabilito a opção de volume para o Prometheus ( NÃO FAZER ISSO EM PRODUÇÃO !!!)    
    enabled: false

  service:
    ## Defino o tipo do Service 
    type: NodePort
    ## Definindo a porta que vai ser usada no Service NodePort
    nodePort: 30001
```

## Create and install prometheus
```shell
helm install prometheus prometheus-community/prometheus --values ./values.yaml
```
## Testing prometheus run while get products
```shell
while true; do curl http://localhost:8080/api/produto; sleep 1; done;
```

## Add grafana
```shell
helm repo add kube-ops https://charts.kube-ops.io
```

```shell
helm install grafana grafana/grafana --values ./values.yaml
```

```promql
sum(rate(http_request_duration_seconds_count[1m])) by (project_version)
```
