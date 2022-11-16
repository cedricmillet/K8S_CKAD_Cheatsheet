Un *Cluster* est constitué de *Nodes* (aka. minion).
Un *Node* 

Installler Kubernetes revient à installer les composants suivants :
* api-server : appelé par le cli
* etcd: key-value store distribué à travers les nodes du cluster
* scheduler: distribuer le travail à travers les différents nodes
* controller: cerveau derière orchestration, chargé d'avertir et de répondre lorsque les noeuds tombent
* container-runtaime: docker, rkt, cry-io
* kubelet: agent qui tourne sur chaque noeud du cluster, s'assure que les containers tournent

Master / worker setup
* Un *master* node embarque kube-apiserver, etcd, controller, scheduler
* Un *worker* node embarque kubelet + container runtime (docker)

## Common
```
kubectl get nodes
kubectl cluster-info
kubectl get all
# This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
kubectl create -f <file> --dry-run=client
```

## Namespaces
```
kubectl get ns
kubectl create ns <namespace_name> --dry-run=client -o yaml > new-namespace.yaml
```

## Pod
Un **pod** contient 1 container.

Un multi-containers pods consiste à intégrer un 1 ou plrs containers **helper** en pour accompagner le container principal.

```
# lancer un pod
kubectl run <podname> --image <image>
# générer le yaml d'un pod
kubectl run nginx-pod --image nginx:alpine --dry-run=client -o yaml > pod.yaml
# afficher les pods
kubectl get pods
kubectl describe pods <podname> 
kubectl edit po <podname>
kubectl replace -f /tmp/kubectl-edit-1960680826.yaml --force
kubectl replace -f pod.yaml --force
kubectl delete pod/<podname>
kubectl delete $(kubectl get pod -o name)
``` 


pod-definition.yml
```
apiVersion: v1
kind: pod
metadata:
 name: myapp-pod
 labels:
  app: myapp
  type: frontend
spec:
 containers:
  - name: nginx-container
  image: nginx:latest
# kubectl create -f pod-definition.yaml
```

## ReplicatSet

```
kubectl get rs
kubectl get rs <replica-set-name> -o yaml
kubectl create -f replicaset-definition.yaml
kubectl scale rs/<replica-set-name> --replicas=5 -f replicasetdefinition.yaml
kubectl replace -f replicaset-definition.yaml
```

replicaset.yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

## Deployment

Un **deployment** gère un/plrs **ReplicatSet**.

```
kubectl create -f deployment.yml
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3 --dry-run=client -o yaml
kubectl get deployments
kubectl get replicaset
```

## Service

### Create a Service named redis-service of type ClusterIP to expose pod redis on port 637
```
# This will automatically use the pod's labels as selectors
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml

# or this (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option.)
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

### Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes

```
# automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml


# or This will not use the pods labels as selectors
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

## ConfigMaps
```
kubectl get cm
kubectl get cm/<name> -o yaml
kubectl describe configmaps
kubectl create configmap <name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
kubectl create configmap <name> --from-file=example.properties
kubectl create -f
```

## Secrets

```
# View
kubectl get secrets
kubectl get secret <scret> -o yaml
kubectl describe secrets
# Create
kubectl create secret generic <secret-name> --from-literal=<key>=<value>
kubectl create secret generic <secret-name> --from-env-file=<path-to-file>
kubectl create -f <path-to-secret.yaml>
# /!\ values stored as b64 (echo -n 'value' | base64)
```

## ServiceAccount

```
kubectl get serviceaccount
kubectl describe serviceaccount <name>

kubectl create serviceaccount <service-account-name>
# if k8s < v1.24 un Secret est créé avec un token pour valeur
kubectl describe secret <serviceaccount-token-name>
# if k8s >= v1.24 aucun Secret n'est créé, on doit manuellement créer le token du ServiceAccount
kubectl create token <service-account-name>
# le token créé aura une date d'expiration, si on souhaite un token sans date d'expiration on génère un Secret (mauvaise pratique)

# decoder un token
jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64d | fromjson' <<< eYanshjkajhsdAjsjsdjTOKEN....
```

Par défaut, lorsqu'un pod est créé un volume est monté dans le pod afin de partager le token du compte de service `default` du namespace courrant. Cela permet au pod d'effectuer des requêtes basiques auprès de l'API K8S.

```
spec:
 containers: ...
 serviceAccountName: <name>
 automountServiceAccountToken: true/false
```

K8S v1.22 introduit `TokenRequestAPI` afin de de gérer l'accès, l'expiration et la sécurité des comptes de service.


## Resource Limits
* K8S gère 3 types de ressources: CPU, Memory, Disk
* Il est possible de déclarer les ressources nécéssaire ET maximales de chaque **container** d'un **pod**.
* Il est possible de déclarer les ressources nécéssaires par défaut à l'échelle du **namespace**.
```
spec:
 containers:
  - name: my-application
    image: my-image
    resources:
     # Resource requirements
     requests:
      memory: "1Gi"
      cpu: 1
     # Resource limits
     limits:
      memory: "2Gi"
      cpu: 2
```

* La consommation **CPU** d'un container est bridée par K8S. 
* Par contre un container peut excèder la consommation **memoire** maximale, dans ce cas le **pod** est tué.

 ## Taints and tolerations
 
Les **teintes** et les **tolérances** ne servent qu'à empêcher les **noeuds** d'accepter certains **pods**.
Cela ne force pas les pods d'aller sur des noeuds particuliers (!= **Affinity**).

* On place une **taint** sur un **node**.
* On place une **toleration** sur un **pod**.

___Remarque___: Les master nodes ont une teinte appliquée empechant le déploiement de pods, ne laissant que les worker nodes pour cela (`kubectl describe node kubemaster | grep Taint`). 


 
 ### Taint
 ```
kubectl taint nodes <node-name> key=value:<taint-effect>
kubectl taint nodes node1 app=blue:NoSchedule
 ```

\<taint-effect\> permet de définir ce qui arrive aux pods qui ne sont pas tolérants à la teinte appliquée:
* **NoSchedule**: ne pas placer le pod sur le noeud
* **PreferNoSchedule**: eviter si possible
* **NoExecute**: ne pas placer ET ejecter les pods qui ne repondent pas à la teinte

### Toleration
 
 pod.yaml
```
spec:
 containers:
  - name: my-application
    image: my-image
 tolerations:
 - key: "app"
   operator: "Equal"
   value: "blue"
   effect: "NoSchedule"
 ```
 
## Node Selectors

### 1. Label node
```
kubectl label nodes <node-name> <label-key>=<label-value>
```

### 2. Create pod with nodeSelector
pod.yaml
```
spec:
 containers:
  - name: my-application
    image: my-image
 nodeSelectors:
  size: large # node key/balue labels
 ```
 

## Node Affinity

pod.yaml
```
spec:
 containers:
  - name: my-application
    image: my-image
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: size
        operator: In     # d'autres operateurs du type "NotIn" ou "Exists"
        values:
        - Large
 ```

Différents type d'affinity :
 * `requiredDuringSchedulingIgnoredDuringExecution`
 * `preferredDuringSchedulingIgnoredDuringExecution`
 * `requiredDuringSchedulingRequiredDuringExecution` (pas encore implémenté en v1.24)

## Multi-Container PODs

Partage le meme espace réseau (localhost), meme filesystem

```
spec:
 containers:
  - name: my-application
    image: my-application:v1.0
    ports:
     - containerPort: 8080
  - name: log-agent
    image: log-agent:v2.0.0
 ```
 
 Plusieurs Design Patterns:
 * `Sidecar` : Etendre les fonctionnalités du container principal (ex: envoyer des logs à un système externe)
 * `Adapter` : Transformer / modifier la sortie du container principal
 * `Ambassador` : Proxy vers un sytème externe (ex: un service qui contient un token API ou une connexion à une DB)
 
 
 ## Readiness Probe / Liveness Probe / Stratup Probe

* **Readiness** est utilisé pour detecter à partir de quand un container est prêt à accepter le traffic.
* **Liveness** est utilisé pour detecter quand redémarrer un container (healthcheck) 
* **Startup** est utilisé pour detecter quand un container est demarré, bloquant Liveness et Readiness

pod.yaml
```
spec:
  containers:
  - name: liveness
    image: <image>
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
```
