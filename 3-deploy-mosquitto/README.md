# Deploy broker Mosquitto su cluster

Deploy broker mosquitto, senza autenticazione, esposto su porta 31883, su cluster k3s nel namespace *iot* con 3 persistent volume su NAS con ip 192.168.1.100.

## configurazione NAS

```
  nfs:
    server: 192.168.1.100
    # Exported path of your NFS server
    path: "/volume1/k3s-data/mosquitto/{dir}"
```

* *config-persistent-volume* montato su `/mosquitto/config` e che mappa `/volume1/k3s-data/mosquitto/config`. In questo path è necessario creare un file mosquitto.conf [(esempio)](manifest/mosquitto-config/mosquitto.conf) 
* *data-persistent-volume* montato su `/mosquitto/data` che punta a `/volume1/k3s-data/mosquitto/data`
* *log-persistent-volume* montato su `/mosquitto/log` che punta a `/volume1/k3s-data/mosquitto/log` 


## deployment

Il deploy su k3s può essere eseguito facendo l'apply del contenuto della directory manifest

```
kubectl -n iot apply -f manifest/
```

