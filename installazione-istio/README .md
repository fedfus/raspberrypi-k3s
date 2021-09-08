# [Istio](https://istio.io/latest/) 

## Prerequisiti
Gli step che seguiranno richiedono di aver installato sul proprio pc [kubectl](https://kubernetes.io/docs/tasks/tools/) 
#### creazione cluster
Per semplicità, verrà utilizzato un cluster virtualizzato su docker con [K3D](https://k3d.io/) composto da un server e due agent, che espone le api di k3s sulla porta 6550 (non necessario) e mappa la porta 80 del container che corrisponde al nodefilter "loadbalancer" alla 8081.
Creiamo quindi il cluster con il seguente comando:

```
k3d cluster create provacluster --agents 2 --api-port 6550 -p "8081:80@loadbalancer"
```
#### deploy di nginx
Una volta creato il cluster, facciamo il deploy di nginx e rendiamolo accessibile via ingress

```
kubectl create deployment nginx --image=nginx
```
```
kubectl create service clusterip nginx --tcp=80:80
```
creiamo quindi il file manifest nginx-ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```
e facciamo l'apply

```
kubectl apply -f nginx-ingress.yaml
```

## Installazione istioctl
Per installare istioctl è possibile seguire direttamente i tre semplici passaggi illustrati nella [documentazione ufficiale](https://istio.io/latest/docs/setup/getting-started/#download) oppure, se si usa macos con homebrew installato, è sufficiente dare il comando 

```
 brew install istioctl
```
Una volta installato istioctl e dopo aver controllato che il cluster sia attivo 

```
❯ kubectl get nodes
NAME                        STATUS   ROLES                  AGE   VERSION
k3d-provacluster-server-0   Ready    control-plane,master   52s   v1.20.10+k3s1
k3d-provacluster-agent-1    Ready    <none>                 24s   v1.20.10+k3s1
k3d-provacluster-agent-0    Ready    <none>                 39s   v1.20.10+k3s1
``` 
procediamo con [l'installazione](https://istio.io/latest/docs/setup/getting-started/#install) vera e propria di istio sul cluster utilizzando il profilo *demo*, ovvero una configurazione apposita che permette di illustrare le funzinalità di istio:

```
istioctl install --set profile=demo -y
```
Una volta completata l'installazione, lanciando il comando 
```
kubectl get namespace
```
vedremo che sarà attivo un nuovo namespace *istio-system* .

Per completare l'installazione, andremo a lanciare il comando 

```
kubectl label namespace default istio-injection=enabled
```
che istruirà istio a fare automaticamente l'inject del sidecar *Envoy* quando faremo il deploy dell'applicazione.

### Deploy dell'applicazione Bookinfo 
Una volta installato e configurato istioctl possiamo procedere al deploy dell'applicazione demo Bookinfo

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/platform/kube/bookinfo.yaml 
```
Attendiamo fino a che tutti i pod non saranno in stato running 

```
❯ kubectl get pods

NAME                              READY   STATUS    RESTARTS   AGE
ratings-v1-b6994bb9-bzn82         2/2     Running   0          6m
details-v1-79f774bdb9-6llc5       2/2     Running   0          6m
productpage-v1-6b746f74dc-l6tzx   2/2     Running   0          6m
reviews-v1-545db77b95-fqdp2       2/2     Running   0          6m
reviews-v3-84779c7bbc-8bkl7       2/2     Running   0          6m
reviews-v2-7bf8c9648f-8dzd6       2/2     Running   0          6m
```

Anche se i pod sono in stato running, l'applicazione non è ancora accessibile dall'esterno. Per renderla accessibile, dobbiamo associare l'applicazione al gateway di Istio. 

Di seguito un estratto del file [bookinfo-gateway.yaml](https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/networking/bookinfo-gateway.yaml) dove è possibile vedere come verrà definito il gateway:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"

```
Procediamo quindi con l'apply 

```
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/networking/bookinfo-gateway.yaml
```
dopo qualche istante, lanciando il comando 

```
kubectl get svc istio-ingressgateway -n istio-system
```
controlliamo se è disponibile uno o più EXTERNAL-IP . Se così non fosse, può darsi che la configurazione iniziale di nginx non è andata a buon fine oppure non si sta utilizzando k3d. In entrambi i casi è possibile proseguire recuperando INGRESS_HOST e INGRESS_PORT seguendo la [documentazione ufficiale istio](https://istio.io/latest/docs/setup/getting-started/#determining-the-ingress-ip-and-ports).


 
