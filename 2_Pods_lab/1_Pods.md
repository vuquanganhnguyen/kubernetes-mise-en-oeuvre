# Manipulation des Resources Pods

Pour ce Lab et la suite, nous allons le faire sur un cluster Kubernetes composé de 1 Master node et 3 worker nodes.
Un cluster Kubernetes peut être déployé sur des machines physiques ou virtuelles. 

## Environnement Lab

Un des outils lees plus répandu pour l'installation d'un cluster Kubernetes en locale est [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).

Accédez au répertoire `0 - Lab Setup` en ligne de commande.
Lancez l'environment Vagrant `k8s-lab`

```bash
vagrant up
vagrant status
vagrant ssh k8s-master-1
```

## Exploration de notre cluster

Tout ce qui sera demandé devra être fait en ligne de commande avec l'outil `kubectl`

Lister les noeuds de votre cluster. Combien de master avez-vous et combien de worker nodes ?

<details><summary>Correction</summary>

```bash
  kubectl get nodes
```

</details>

Combien de namespaces avez vous dans votre cluster ? Listez les

<details><summary>Correction</summary>

  ```bash
  kubectl get namespaces 
  ```

</details>

Lister les Pods de tous les namespaces

<details><summary>Correction</summary>

```bash
kubectl get pods -A 
```

</details>

## Opérations sur les Pods & Containers


lancer un Pod du nom de `webserver` avec comme image `nginx:latest` de la manière impérative et visualiser :
* Sur quel noeud il a été déloyé  
* Dans quel namespaces ?
* Quel est son état ?

<details><summary>Correction</summary>

```bash
kubectl run webserver --image nginx

kubectl get pods -o wide

# cette commande affiche le nom des namespaces également
kubectl get pods -o wide -A

# describe
kubectl describe pod webserver
```

Il a été déployé dans le namespaces par défaut.

</details>

De la manière **Impérative**, Créer un namespace `k8s-lab` et lancez un Pod du nom de `tools`, `image:busybox` exécutant la commande `sleep 1000`

<details><summary>Correction</summary>

```bash
kubectl create ns k8s-lab
kubectl run tools --image busybox -n k8s-lab -- sleep 1000
```

</details>

Tentez de créer un pod du même nom avec une image de votre choix dans le même namespace précédent `k8s-lab`

Arrivez-vous à le faire ? Si non pourquoi ?

<details><summary>Correction</summary>

Dans un namespace, le nom d'une ressource doit être unique, un peu comme dans un espace de nom DNS

</details>

Faite une description du pod webserver et une description du pod tools

<details><summary>Correction</summary>

```bash
kubectl describe pods webserver
kubectl describe pods tools -n k8s-lab
```

</details>

Quels sont les IP des Pods précédents ?

Accédez à l'intérieur du Pod `tools` et emmettez une requete vers le Pod `Webserver`: un Ping, curl...

<details><summary>Correction</summary>

```bash
kubectl exec -it -n k8s-lab tools -- sh
/ # ping 192.168.118.29
PING 192.168.118.29 (192.168.118.29): 56 data bytes
64 bytes from 192.168.118.29: seq=0 ttl=63 time=0.268 ms
64 bytes from 192.168.118.29: seq=1 ttl=63 time=0.063 ms
```

</details>

Si le Ping passe, pourquoi selon vous c'est passé et si non pour quoi ? est-ce que le fait que les ressources soient dans differents namespaces empêchent les communications ?

<details><summary>Correction</summary>

Non, les namespaces ne limitent pas les communications réseaux.

</details>

Comment regarder le positionnement des Pods ?

<details><summary>Correction</summary>

Soit en faisant le Describe comme précédament ==>  il y'a une section `Node:             k8s-worker-1/192.168.56.12`
Soit en affichant les pods avec l'option `-o wide`

```bash
kubectl get pods -o wide
```

</details>

## Approche Déclarative

De la manière **Déclarative** lancez un Pod du nom de `tools2`, `image:busybox` exécutant la commande `sleep 1500` dans le namespace `k8s-lab`

<details><summary>Correction</summary>

Il faut ecrire le dichier YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: tools2
  name: tools2
  namespace: k8s-lab
spec:
  containers:
  - args:
    - sleep
    - "1500"
    image: busybox
    name: tools2
    resources: {}
```

Une astuce pour avoir ce fichier YAML est d'exécuter la commande impérative correspondante avec l'option `--dry-run=client -o yaml`

```bash
kubectl run tools --image busybox -n k8s-lab --dry-run=client -o yaml -- sleep 1500 > pod-tools2.yml
```

```bash
kubectl apply -f pod-tools2.yml 
```

```bash
kubectl get pods -n k8s-lab
                                                                                                                                                                                                                ─╯
NAME     READY   STATUS             RESTARTS      AGE
tools    0/1     CrashLoopBackOff   4 (86s ago)   3m2s
tools2   1/1     Running            0             18s
```

</details>