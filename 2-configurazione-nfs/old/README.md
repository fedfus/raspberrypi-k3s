# aggiunta nfs-client-provisioner al cluster 

aggiungere il repository helm 
```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

modificare la configurazione di nfs.server e nfs.path e lanciare il comando

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=192.168.1.100 \
    --set nfs.path=/volume1/k3s-data
```