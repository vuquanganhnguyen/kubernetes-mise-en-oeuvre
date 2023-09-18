# Installation de Minikube

Un cluster Kubernetes peut être déployé sur des machines physiques ou virtuelles. 
Pour commencer à développer Kubernetes ou test vos déploiement en environnement de Test/Dev, Minikube est l'une des solutions les plus répandues de nos jours. 
Minikube est une implémentation Kubernetes légère qui crée une VM sur votre machine locale et déploie un cluster simple contenant un seul nœud. 
Il est disponible pour les systèmes Linux, macOS et Windows.
Il possède une commande CLI (`minikube`) pour des opérations de démarrage de base pour travailler avec votre cluster : démarrage, arrêt, suppression, ajout de fonctionnalités... 
Notre Lab va consister à le mettre en place.

## Environnement Lab

Accédez au répertoire `0_lab_minikube/lab_setup` en ligne de commande.
Lancez la VM `minikube-server`

```bash
vagrant up
vagrant status
vagrant ssh minikube-server
```

## Installation de Minikube

La suite de ce Lab se fera sur le serveur `minikube-server`

Se réferrer à [la documentation officielle](https://kubernetes.io/fr/docs/tasks/tools/install-minikube/)

Mettre à jour l'index des paquets apt existant

```bash
sudo apt update -y
```

Installez les dépendances minikube suivantes

```bash
$ sudo apt install -y curl wget apt-transport-https
```

Télécharger la dernière version du binaire Minikube et installez le

```bash
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo cp minikube-linux-amd64 /usr/local/bin/minikube
sudo chmod +x /usr/local/bin/minikube
```

Vérifier la version de minikube

```bash
minikube version
```

## Installation de Kubectl

Se réferrer à [la documentation officielle](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

**kubectl** est un outil en ligne de commande, fournit par Kubernetes pour communiquer avec le plan de contrôle d'un cluster Kubernetes (API-Server)
**kubectl** se réfère à un fichier nommé `config` dans le répertoire personnel de l'utilisateur `$HOME/.kube/config` dans lequel se trouve les utilisateurs, les contexts, les clusters auxquels l'utilisateur peut se connecter.


```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Vérifier la version de kubectl installé

```bash
kubectl version --client --output=yaml 
```

## Installation de Docker

Se réferrer à [la documentation officielle](https://docs.docker.com/engine/install/ubuntu/)

Mettre à jour l'index des paquets apt

```bash
sudo apt-get update
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Ajouter la clé GPG officielle de Docker

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Configurer le Dépot 

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Installation de Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

```

Vérifiez que l'installation de Docker Engine est réussie en exécutant l'image `hello-world`

```bash
docker version
sudo systemctl status docker
sudo docker run hello-world
```



## Démarrer Minikube

Se réferrer à [la documentation officielle](https://minikube.sigs.k8s.io/docs/start/)

En tant qu'utilisateur actuel, nous allons nous ajouter au group d'utilisateurs `docker` qui ont le droit d'exécuter Docker Engine

```bash
sudo usermod -aG docker $USER && newgrp docker
```

Démarrer MiniKube

```bash
minikube start --driver=docker
```

## Accès au cluster MiniKube & test

Afficher le contenu du fichier `$HOME/.kube/config`, il devrait y avoir du contenu. Nous le décrypterons plus tard
Pour l'instant vérifiez bien que vous pouvez : 

Voir un context minikube présent 

```bash
kubectl config get-contexts
```

lister les ressources du nouveau cluster Minikube

```bash
kubectl get nodes
```
![Screenshot 2022-11-06 at 02 07 59](https://user-images.githubusercontent.com/59444079/200149425-e95099ee-43bc-43de-82bf-82cb14f7d8ba.png)

## Pour aller plus loin

Par défaut, seuls quelques fonctionnalités (`addons`) sont activés lors de l'installation de minikube, pour voir les addons de minikube:

```bash
minikube addons list
```

Si nous voulions activer et accéder au tableau de bord de Kubernetes

```bash
minikube dashboard
```
```bash

```