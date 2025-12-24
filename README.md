# DevSecOps_Kubernetes

Repo pédagogique pour apprendre **Kubernetes** puis les **bonnes pratiques** de **sécurités**.
## Environnement

Utilisation de Rancher Desktop. \
https://rancherdesktop.io/

## Commande de base

```shell-session
❯ kubectl version
Client Version: v1.33.6
Kustomize Version: v5.6.0
Server Version: v1.33.6+k3s1
```

```shell-session
❯ kubectl get nodes
NAME                   STATUS   ROLES                  AGE   VERSION
lima-rancher-desktop   Ready    control-plane,master   17h   v1.33.6+k3s1
```

Nous avons que le node "control-plane" puisque rien n'a encore été installé.
```shell-session
❯ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS        AGE
kube-system   coredns-6d668d687-xw8sh                   1/1     Running     2 (4h15m ago)   21h
kube-system   helm-install-traefik-4cccr                0/1     Completed   2               21h
...
```
-A : all namespaces

```shell-session
❯ kubectl run <pod_name> --image=<image>
```

### Exemple :

```shell-session
❯ kubectl run my-nginx --image=nginx
pod/my-nginx created
```

```shell-session
❯ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
my-nginx   1/1     Running   0          37s
```

```shell-session
❯ kubectl describe pod my-nginx
Name:             my-nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             lima-rancher-desktop/192.168.5.15
Start Time:       Tue, 23 Dec 2025 14:50:07 +0100
Labels:           run=my-nginx
Annotations:      <none>
Status:           Running
IP:               10.42.0.20 
```

Logs : ```kubectl logs <pod_name>```

Exécuter une commande dans le container :

```
kubectl exec -it <pod_name> -- <program>
```

**kubectl exec** : exécute une commande dans un conteneur d’un Pod.

**-i** : garde l’entrée standard (stdin) ouverte pour que tu puisses taper des commandes.

**-t** : alloue un pseudo-TTY, ce qui rend l’expérience similaire à un terminal interactif.

**--** : sépare les options de kubectl de la commande à exécuter dans le conteneur.

​**program** : exemple un shell Bash, /bin/bash.

Supprimer pod :

```
kubectl delete pod <pod_name>
```

## Deployment (Déploiements)

```
kubectl create deployment <deployment_name> --image=<image> --replicas=3
```

### Exemple :

```
kubectl create deployment my-app --image=nginx --replicas=3
```

```
❯ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
my-app-56f7689b79-7phhf   1/1     Running   0          8m42s
my-app-56f7689b79-km5mg   1/1     Running   0          8m42s
my-app-56f7689b79-t72j2   1/1     Running   0          3m30s
```

```
❯ kubectl get deployment my-app
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   3/3     3            3           9m26s
```

#### Self-Healing

```
❯ kubectl delete pod my-app-56f7689b79-4444x
pod "my-app-56f7689b79-4444x" deleted
```

```
❯ kubectl get pods --watch
NAME                      READY   STATUS    RESTARTS   AGE
my-app-56f7689b79-4444x   1/1     Running   0          4m38s
my-app-56f7689b79-7phhf   1/1     Running   0          4m38s
my-app-56f7689b79-km5mg   1/1     Running   0          4m38s
my-app-56f7689b79-4444x   1/1     Terminating   0          5m12s
my-app-56f7689b79-t72j2   0/1     Pending       0          0s
my-app-56f7689b79-t72j2   0/1     ContainerCreating   0          0s
my-app-56f7689b79-4444x   0/1     Completed           0          5m13s
my-app-56f7689b79-t72j2   1/1     Running             0          3s
```

#### Scaling

```
kubectl scale deployment my-app --replicas=2
```

Rajoute ou réduit au nombre de replicas (pods) souhaité. \ 
**A voir : Autoscaler**

#### Replicasets

```
❯ kubectl get replicaset
NAME                DESIRED   CURRENT   READY   AGE
my-app-56f7689b79   2         2         2       21m
```

Un Deployment est un concept de plus haut niveau qui gère les ReplicaSets.
```
​Deployment
└── ReplicaSet
	└── Pods
```

#### Deployment Rollouts

```
❯ kubectl set image deployment/my-app nginx=nginx:1.25
deployment.apps/my-app image updated
```

```
❯ kubectl rollout status deployment/my-app
deployment "my-app" successfully rolled out
```

```
❯ kubectl rollout history deployment/my-app
deployment.apps/my-app 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```
❯ kubectl rollout undo deployment/my-app
deployment.apps/my-app rolled back

❯ kubectl rollout history deployment/my-app
deployment.apps/my-app 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

## Ressources

https://kubernetes.io/ \
https://blog.stephane-robert.info/docs/conteneurs/orchestrateurs/kubernetes/ \
https://go.kubecraft.dev/k8s-quickstart
