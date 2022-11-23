# State Persistence

## Volumes

* Il est possible d'atatcher un **Volume** à un **Pod** dans un pod-definition.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    # SI 1 seul noeud dans le cluster
    hostPath:
      path: /data             # creation du répertoire /data sur le node
      type: Directory
```


## Persistent Volumes

* Afin de ne pas avoir à préciser le volume sur chaque **pod** on utilise des **Persistent Volume**
* Les **Persistent Volume** sont créés par des Administrateurs K8S


persistent-volume.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  # How the volume should be mounted on the host ?
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

```
kubectl create -f persistent-volume.yaml
kubectl get persistenvolume
```

## Persistent Volume Claim

* Les **Persistent Volume Claim** sont créés par des Utilisateurs K8S
* Chaque **Persistent Volume Claim** est associé à un seul et unique **Persistent Volume**

Que se passe t-il lorsqu'un PVC est supprimé ?
* **Retain**: le PV reste en attente d'être manuellement supprimé par un administrateur
* **Delete**: le PV est supprimé lorsque le PVC est supprimé
* **Recycle**: le PV est vidé et ensuite disponiuble par un autre PVC

pvc-definition.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

```
kubectl get persistentvolumeclaim
kubectl delete persistenvolumeclaim <my-volume>
```


A ajouter danss la section **POD** d'un **ReplicatSet** ou **Deployment**.

````yaml
spec:
  containers:
    - iamge: nginx
  volumes:
    - name: mypod
      persistentVolumeClaim:
        claimName: pvc-myclaim
```
