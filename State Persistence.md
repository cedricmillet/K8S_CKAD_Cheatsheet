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

Afin de ne pas avoir à préciser le volume sur chaque **pod** que l'on créé on utilise **Persistent Volume**

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
