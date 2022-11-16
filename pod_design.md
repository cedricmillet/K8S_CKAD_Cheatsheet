# Pod Design


## Labels, Selectors and annotations

* Les **labels** sont utilisés dans les **ReplicaSet**, les **Services** pour désigner des objets à controler
* Les **annotations** sont utilisées à titre descriptif seulement
* Les **selectors** permettent de matcher via différents champs tels que les **labels**

```
kind: Pod
metadata:
  name: <podname>
  labels:
    app: MyApplication
    function: frontend
  annotations:
    buildVersion: v1.25.4
    author: John Doe
```

```
kubectl get pods --selector app=MyApplication
```



## Rolling update & Rollbacks in Deployments

## Jobs

## Cronjobs

