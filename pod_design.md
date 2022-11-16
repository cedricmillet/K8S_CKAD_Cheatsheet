# Pod Design


## Labels, Selectors and annotations

* Les **labels** sont utilisés dans les **ReplicaSet**, les **Services** pour désigner des objets à controler
* Les **annotations** sont utilisées à titre descriptif seulement
* Les **selectors** permettent de matcher via différents champs tels que les **labels**

```yaml
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

Lorsque l'on lance un **Deployment** un **Rollout** est créé, et versionnée avec une nouvelle révision. 

```yaml
# Afficher le status d'un déploiement
kubectl rollout status deployment/<deployment-name>

# Afficher l'historique (les révisions) d'un déploiement
kubectl rollout history deployment/<deployment-name>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>

# Afficher la revision#1
kubectl rollout history deployment <deployument-name> --revision=1

# Afficher la stratégie de déploiement
kubectl describe deployment <deployment-name> | grep StrategyType:
```

Deux stratégies de déploiement :
* **Recreate** Détruire tous les pods, puis lancer tous les pods de la nouvelle version => engendre une période d'indisponibilité
* **Rolling Update** Détruire et relancer chacun des pods un à un, les uns après les autres


Montée de version d'un déploiement:
* Editer le `deployment.yaml`, puis `kubectl apply -f deployment.yaml`
* ou `kubectl set image deployment/<deployment-name> nginx=nginx:v2.0.0 [--record]` => engendre une différence de config entre le .yaml et ce qui tourne

## Jobs

Les **ReplicatSet** s'assurent qu'une flotte de **pods** tournent **constamment** (pour exposer un serveur web par exemple).
Contraitement aux **ReplicaSet**, les **Jobs** sont des workloads destinés à exécuter une tâches à travers 1/plrs **pods** et à se **terminer** une fois la tache accomplie.

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: <jobname>
spec:
  comletions: 3 # nbre de fois que le job doit réussir
  parallelism: 3 # nbre de jobs paralleles autorisés (1 pour les exec sequentiellement)
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

```
kubectl create -f job.yaml
kubectl get jobs # will create required pods
kubectl get pods
kubectl delete jon <jobname> # will delete pods
```


## Cronjobs

```yaml
# cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: <cronjobname>
spec:                       # cronjob spec
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:                   # job spec
      comletions: 3 # nbre de fois que le job doit réussir
      parallelism: 3 # nbre de jobs paralleles autorisés (1 pour les exec sequentiellement)
      template:
        spec:               # pod spec
          containers:
            - name: math-add
              image: ubuntu
              command: ['expr', '3', '+', '2']
          restartPolicy: Never
```


```
kubectl create -f cronjob.yaml
kubectl get cronjob
```
