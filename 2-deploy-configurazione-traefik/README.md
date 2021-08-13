# Deploy e configurazione [Traefik](https://traefik.io/)

> Se non hai eseguito l'installazione di k3s con il parametro --disable=traefik, Traefik è già attivo di default.
> Se non hai necessità di modificare il comportamento di default di traefik, puoi installare k3s senza il parametro --disable=traefik.



Per procedere all'installazione manuale di traefik verrà utilizzato [helm](https://helm.sh/docs/intro/install/) .

Procediamo quindi con l'aggiungere il repo

```
helm repo add traefik https://helm.traefik.io/traefik
```
e ad aggiornare 

```
helm repo update
```
A questo punto scarichiamo il file helm values per Traefik da [traefik-helm-chart](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml) . Con un editor di testo andiamo a modificare il file *values.yaml* appena scaricato, l'obiettivo è quello di modificare la porta http dalla 80 impostata di default alla 10080.

```
...
# Configure ports
ports:
  # The name of this one can't be changed as it is used for the readiness and
  # liveness probes, but you can adjust its config to your liking
  traefik:
    port: 9000
    # Use hostPort if set.
    # hostPort: 9000
    #
    # Use hostIP if set. If not set, Kubernetes will default to 0.0.0.0, which
    # means it's listening on all your interfaces and all your IPs. You may want
    # to set this value if you need traefik to listen on specific interface
    # only.
    # hostIP: 192.168.100.10

    # Override the liveness/readiness port. This is useful to integrate traefik
    # with an external Load Balancer that performs healthchecks.
    # healthchecksPort: 9000

    # Defines whether the port is exposed if service.type is LoadBalancer or
    # NodePort.
    #
    # You SHOULD NOT expose the traefik port on production deployments.
    # If you want to access it from outside of your cluster,
    # use `kubectl port-forward` or create a secure ingress
    expose: false # <--- impostare a true se si vuole esporre la dashboard di traefik
    # The exposed port for this service
    exposedPort: 9000
    # The port protocol (TCP/UDP)
    protocol: TCP
  web:
    port: 8000
    # hostPort: 8000
    expose: true
    exposedPort: 10080 # <--- porta modificata da 80 a 10080
    # The port protocol (TCP/UDP)
    protocol: TCP
...

```
Completata la modifica al file di configurazione, possiamo fare il deploy su k3s.
Creiamo un nuovo namespace:

```
kubectl create namespace test-traefik
```
e lanciamo l'installazione con helm

```
helm install --namespace=test-traefik --values=./values.yaml traefik traefik/traefik
```
```
❯ helm install --namespace=test-traefik --values=./values.yaml traefik traefik/traefik
NAME: traefik
LAST DEPLOYED: Fri Aug 13 17:15:57 2021
NAMESPACE: test-traefik
STATUS: deployed
REVISION: 1
TEST SUITE: None
```
Per verificarne lo stato è sufficiente lanciare

```
❯ kubectl -n test-traefik get services
NAME      TYPE           CLUSTER-IP     EXTERNAL-IP                PORT(S)                         AGE
traefik   LoadBalancer   10.43.136.10   192.168.0.1,192.168.1.86   10080:31309/TCP,443:30624/TCP   23m
```