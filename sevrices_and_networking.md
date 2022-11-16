# Services & Networking

## Services

* Un **Node Port** permet de rendre accessible un **pod** sur un port donné du **node**.
* Un **ClusterIP** crée une IP virtuelle à l'intérieur du **cluster** pour permettre la communication entre les services
* Un **Load Balancer** équilibre la charge à traverrs différents services

### Node Port 

Permet d'exposer le port d'un **pod** (TargetPort) à travers un port sur le **node** (NodePort).

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: <service-name>
spec:
  type: NodePort
  ports:
  - targetPort: 80    # port du pod
    port: 80          # port du service (obligatoire)
    nodePort: 3008    # port du node (entre 30000 et 32767)
  selector:           # identifier le/les pod(s) à mapper via ses labels
    app: my-application
```

```
kubectl create -f service;yaml
kubectl get services
curl http://<ip-of-node>:3008
```

### ClusterIP

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: <service-name>
spec:
  type: ClusterIP     # Default service type
  ports:
  - targetPort: 80    # port du pod
    port: 80          # port du service
  selector:           # identifier le/les pod(s) à mapper via ses labels
    app: my-application
```
