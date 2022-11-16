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

```yaml
# single label selection
kubectl get pods --selector app=MyApplication
# multiple labels selection
kubectl get pod --selector env=prod,bu=finance,tier=frontend
```



## Rolling update & Rollbacks in Deployments

## Jobs

## Cronjobs

