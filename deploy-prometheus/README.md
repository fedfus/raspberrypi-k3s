# Installazione Prometheus su cluster k3s via helm

### aggiunta repository

``` 
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```
### deploy su cluster
```
kubectl create ns prometheus
helm install prometheus prometheus-community/kube-prometheus-stack --namespace prometheus
```

