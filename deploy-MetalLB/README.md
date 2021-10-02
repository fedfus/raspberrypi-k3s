# [MetalLB](https://metallb.universe.tf/) 

MetalLB è un network load balancer per bare-metal clusters. 

## installazione
È possibile seguire più strade per installare MetalLB (helm, kustomize, manifest) ma, per coerenza, procederemo all'installazione tramite file manifest.

Recuperiamo quindi dal [repository ufficiale](https://github.com/metallb/metallb) i seguenti manifest:

* [namespace.yaml](https://github.com/metallb/metallb/blob/main/manifests/namespace.yaml)
* [metallb.yaml](https://github.com/metallb/metallb/blob/main/manifests/metallb.yaml)

ed applichiamoli:

```
kubectl apply -f namespace.yaml
```
```
kubectl apply -f metallb.yaml
```
oppure è possibile dare l'apply direttamente a partire dai file presenti nel repo di MetalLB senza doverli scaricare

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/namespace.yaml
```
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/manifests/metallb.yaml
```
## [configurazione](https://metallb.universe.tf/configuration/)
Dopo aver fatto l'apply di namespace.yaml e di metallb.yaml, il servizio rimane in attesa fino a che non verrà creata e deployata una *config map* nello stesso namespace.

Le configurazioni possibili sono: 

* layer 2 configuration
* BGP configuration

In questo caso procederemo con una configurazione layer 2.
Di seguito un [esempio di config map](./manifest/metalLb-configmap.yaml):

```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.0.240-192.168.0.250
```
con questa configurazione, MetalLB avrà il controllo degli indirizzi dal 192.168.0.240 al 192.168.0.250 .


