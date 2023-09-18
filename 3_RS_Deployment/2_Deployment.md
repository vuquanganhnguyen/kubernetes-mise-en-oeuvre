# Deployement

## Mode Imperatif

Créez un déploiement de 3 pods nginx avec la méthode impérative, le nom du déploiement sera `frontend`

<details><summary>Correction</summary>

```bash
kubectl create deployment frontend --image nginx --replicas 3
```

</details>

Quelles sont les ressources qui ont été créés par cette commande précédente ?

<details><summary>Correction</summary>

Lorsqu'un déploiement est créé, en dehors de la ressource deployment elle même, derrière une ressource Replicaset du même nom dérivé est créé

```bash
kubectl get deployment
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
frontend          3/3     3            3           1m13s
```

```bash
kubectl get replicaset
NAME                     DESIRED   CURRENT   READY   AGE
frontend-7f9649877f      3         3         3       2m40s
```

Ce replicat va engendrer le nombre de pods exprimé dont les noms sont à leur tour dérivés du nom replicaset

```bash
kubectl get pods
NAME                           READY   STATUS             RESTARTS        AGE
frontend-7f9649877f-29c6c      1/1     Running            0               5m36s
frontend-7f9649877f-fq5v7      1/1     Running            0               5m36s
frontend-7f9649877f-pkjst      1/1     Running            0               5m36s
```

</details>

Modifier le deploiement en changeant le nombre de réplicas à 4
 
<details><summary>Correction</summary>

```bash
kubectl edit deployment frontend
```

=> ça s'ouvre avec votre éditeur de texte par défaut. par défaut c'est l'éditeur `vi`
Pour modifier => les touches [ESC] [i]
Pour quitter l'éditeur sans le modifier => [ESC] [:] q!
Après l'avoir modifier, pour enregistrer et quitter => [ESC] [:] wq

</details>

visualiser les événements du déploiement frontend. Quel mécanisme constatez-vous après edition du deploiement au nombre de replicas = 4 ? 

<details><summary>Correction</summary>

```bash
kubectl describe deployment frontend
```

Un pod supplémentaire s'ajoute au replicas

</details>


## Mode Déclaratif - YAML

Dans cette partie tout ce qu'on fera sera fait avec des manifest YAML Kubernetes

Recréez un déploiement du nom de frontend
* nom: frontend
* Image: `<votre login Docker>/lab1-simpleapp:1.0` => nous allons utiliser la première version de l'image que nous avions créé dans le Lab 3. Si vous n'avez pas fait cette partie prenez tout simplement l'image `nginx:1.14.2`
* replicas : 3
* Labels : `app: web-server`, `tier: frontend`
* Méthode de création: Créez le déploiement de sorte à enregistrer vos modifications (ajout de l'option `--record`)

<details><summary>Correction</summary>

Pour avoir les infirmation sur les option d'une resources Kubernetes on peut se servir de `kubectl explain`

```bash
kubectl explain deployment 
kubectl explain deployment --recursive
```

Pour générer les fichier YAML correspondant, il existe une astuce simple qui est de combiner l'utilisation de kubectl avec l'option `--dry-run=client -o yaml` 
l'option `--dry-run=client`indique à l'PAI server que nous somme entrain de faire des simulations et qu'il ne devra pas créer la ressource
l'otion `- o yaml` est de dire que nous voulons un output de l'objet au format YAML. 

```bash
kubectl create deployment frontend --image nginx:1.14.2 --replicas 3 --dry-run=client -o yaml > dep-frontend.yaml

# ou

kubectl create deployment frontend --image mpakoupete/lab1-simpleapp:1.0 --replicas 3 --dry-run=client -o yaml > dep-frontend.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: web-server
    tier: frontend
spec:
  replicas: 3
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```


```bash
kubectl create -f dep-frontend.yaml --record=true
```

L'option records enregistrera vos modifications pour le déploiement frontend.

</details>

Quel est l'avantage de déployer de avec la méthode déclarative ?

<details><summary>Correction</summary>

La manière déclarative a l'avantage de pouvoir définir notre infrastructure applicative en code (IaC) => l'état **désiré** de notre Cluster est sauvegardé et toute modification autres que le Apply n'est pas accecpé.
Vous pouvez également utiliser kubectl apply pour mettre à jour une ressource existante en modifiant les paramètres de configuration dans le fichier manifeste donné et le Re-apply.

</details>

Quel est la stratégie de mise à jour par défaut de ce déployement ?

<details><summary>Correction</summary>

Faites un describe pour voir
La stratégie par défault c'est `RollingUpdate`

</details>

Changer la stratégie de mise à jour par `Recreate` dans le manifest YAML et faite un reapply. Constatez que la stratégie a changé


<details><summary>Correction</summary>

```yaml
(...)
spec:
 replicas: 3
 strategy:
    type: Recreate
 selector:
   matchLabels:
(...)

```

```bash
kubectl apply -f dep-frontend.yaml
```

Faites un describe

</details>

Vous avez mis à jour votre application à la version 2. Vous souhaitez mettre à jour le déploiement => Changez l'image pour la mettre à la dernière version disponible `nginx:1.22.1` ou `<votre login Docker>/lab1-simpleapp:2.0`
Que constatez-vous dans les Events du déployement frontend ?
Que constatez-vous au niveau des ReplicaSet ?

<details><summary>Correction</summary>

Modifier le manifest YAML en mettant la nouvelle version de l'image 

ou bien

```bash
 kubectl set image deployment frontend nginx=nginx:1.22.1 
```

On constate qu'il a supprimer tous les anciens Pods et remplacer par les nouveaux

On constate au niveau des replicaSet que l'ancien est vide et un nouveau est créé

</details>

Afficher l'historique du rollout

<details><summary>Correction</summary>

```bash
kubectl rollout history deployment frontend
```

</details>


