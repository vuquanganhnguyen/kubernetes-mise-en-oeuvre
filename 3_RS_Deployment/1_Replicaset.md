# ReplicaSets

## Exploration

Visualisez tous les ReplicaSets actuels dans le `default` namespace et ensuite dans tous les namespaces.

<details><summary>Correction</summary>

```bash
kubectl get rs
kubectl get rs -A
```

</details>


Créez un fichier YAML pour un ReplicaSet simple avec les carrectéristiques suivantes :
* image : `nginx:1.14.2`
* nom : frontend
* Nombre de replica : 3
* Labels : `tier : frontend` , `app: web-server`
* Quel est le paramètre `apiVersion` que vous aller utiliser ? 

<details><summary>Correction</summary>

D'une manière général et ce pour tous les objets, le paramètre `apiVersion` que nous allons utiliser dépend de la version de Kubernetes que nous utilisons
Pour connaitre la version : `kubectl api-resources| grep -i replicaset` ==> `apps/v1`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
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

```

</details>


Créez le replicaset

<details><summary>Correction</summary>

```bash
kubectl create -f rs-frontend.yaml
```

</details>

Visualisez le nouvel objet créé

<details><summary>Correction</summary>


```bash
kubectl describe rs frontend
```

</details>

Visualisez les Pods créés avec le ReplicaSet. À partir du fichier `yaml` créé, il devrait y avoir 3 Pods. Vous devez voir des noms de Pods aux patterns suivants : `frontend-<suite de caractère aléatoire>` (Ex :  `frontend-fkgg4`)

<details><summary>Correction</summary>

```bash
kubectl get pods
```

</details>

Supprimez un des Pods. Que constatez vous


<details><summary>Correction</summary>

```bash
kubectl delete pods frontend-fkgg4
```

On constate q'un autre est recréé automatiquement. c'est l'oeuvre du control Loup du Replica Controller

</details>

Nous allons tentr d'isoler un POD:
* Choisissez un Pod et et éditer les Labels de ce pod. Remplacez le par `tier: FrontendIsole`

<details><summary>Correction</summary>

```bash
kubectl edit pod frontend-fkgg4

(...)
    labels:
        tier: FrontendIsole #<-- Changer 
managedFields:
(...)

```

</details>

Visualisez le nombre de pods dans le ReplicaSet. Vous devriez en voir 3 en cours d'exécution.

<details><summary>Correction</summary>

```bash
kubectl get rs
```

</details>

Regardez les Pods avec affichage des Labes. Que constatez vous ? 

<details><summary>Correction</summary>

```bash
kubectl get pods --show-labels

# ou

kubectl get pods -L tier
```

On constate qu'il y en a 4, l'un étant plus récent que les autres.
Le ReplicaSet a veillé à conserver 3 réplicas, en remplaçant le pod qui a été isolé.

</details>

Supprimez le ReplicaSet et lister les Pods restant

<details><summary>Correction</summary>

```bash
kubectl delete -f rs-frontend.yaml

# ou

kubectl delete rs frontend
```

```bash
kubectl get po
```

</details>