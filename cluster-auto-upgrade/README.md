# K3s auto upgrade

## [Installazione system upgrade controller](https://rancher.com/docs/k3s/latest/en/upgrades/automated/)

``` 
kubectl apply -f https://github.com/rancher/system-upgrade-controller/releases/latest/download/system-upgrade-controller.yaml 
```

Una volta fatto l'apply del controller, è necessario definire un plan di aggiornamento per il server e per gli agent. È possibile specificare la versione da aggiornare 
``` 
apiVersion: upgrade.cattle.io/v1
kind: Plan
metadata:
  name: server-plan
  namespace: system-upgrade
spec:
 ...
  upgrade:
    image: rancher/k3s-upgrade
  version: v1.17.4+k3s1  
  ```
oppure omettere la versione e puntare direttamente ad un channel 
```
apiVersion: upgrade.cattle.io/v1
kind: Plan
...
spec:
  ...
  channel: https://update.k3s.io/v1-release/channels/stable
```
