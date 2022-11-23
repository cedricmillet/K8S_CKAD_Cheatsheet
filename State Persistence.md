# State Persistence

## Volumes

* Il est possible d'atatcher un **Volume** à un **Pod**

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
