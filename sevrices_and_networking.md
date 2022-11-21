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

## Ingress Networking

* Un **Ingress** est un point d'entrée du cluster qui redirige un trafic vers différents **Services** éventuellement à travers des règles de redirection.

On distingue les **Ingress Controller** et les **Ingress Resources**:
* Kubernetes n'intègre aucun **Ingress Controller** par défaut (Traefik, HA Proxy, NGINX...), il faut l'installer manuellement (sous la forme d'un **Deployment** par exemple (image: quay.io/nginx-ingressoui-controller) + Service NodePort + ConfigMap + ServiceAccount).
* **Ingress Resources** est un ensemble de configurations appliquées sur l'**Ingress Controller** (règles de redirections...)


Create an **Ingress Resource**
```yaml
#ingress-wear.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:            # vers quel service rediriger le trafic ?
    serviceName: wear-service
    servicePort: 80
```

* **Ingress Resource** avec filtrage par **URL**
```yaml
#ingress-wear.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              numner: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port:
              numner: 80
```

* **Ingress Resource** avec filtrage par **nom de domaine**
```
# idem avec le champ 'host:' pour chaque `spec.rules[]`
```

```
kubectl get ingress
kubectl create -f ingress-wear.yaml
kubectl create ingress <ingress-name> --rule="hostname/path=service:port"
kubectl describe ingress <ingress-name>
```


## Process mise en place ingress controller nginx

```
# Creation du namespace
k create ns ingress-space

# Creation donfigmap
k create configmap nginx-configuration -n ingress-space

# Creation compte de service
k create serviceaccount ingress-serviceaccount -n ingress-space

# TODO: Creation role + rolebinding

# Deploy nginx
k create -f deployment.yaml

# Exposer deployment with new service
k create -f service.yaml
```

deployment.yaml
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-space
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```


service.yaml
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```
