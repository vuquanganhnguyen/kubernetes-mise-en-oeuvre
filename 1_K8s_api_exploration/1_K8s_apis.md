# Exploration des API Kubernetes

Dans ce Lab, nous allons nous servir de `kubectl`

#### Rappel:
**kubectl** est un outil en ligne de commande, fournit par Kubernetes pour communiquer avec le plan de contrôle d'un cluster Kubernetes (API-Server)
**kubectl** se réfère à un fichier nommé `config` dans le répertoire personnel de l'utilisateur `$HOME/.kube/config` dans lequel se trouve les utilisateurs, les contexts, les clusters auxquels l'utilisateur peut se connecter.
_Vous pouvez spécifier d'autres fichiers kubeconfig en définissant la variable d'environnement `$KUBECONFIG` ou en définissant l'option `--kubeconfig` ._



## API Groups & resources

Les groupes d'API facilitent l'extension de l'API Kubernetes.
Il existe plusieurs groupes d'API dans Kubernetes :
* Le `core group` (groupe de base, également appelé `legacy`) se trouve dans le chemin REST `/api/v1.`
  Dans les fichier YAML, On lui fait référence avec `apiVersion : v1` 
* Les `named group` se trouvent dans le chemin REST `/apis/$GROUP_NAME/$VERSION` 
  Dans les fichier YAML, On leur fait référence avec `apiVersion : $GROUP_NAME/$VERSION` example `apiVersion : apps/v1`

Lister les groupes d'API Kubernetes

<details><summary>Correction</summary>

```bash
kubectl api-versions
```

</details>

Lister les resources de Kubernetes avec les Groups API correspondant

<details><summary>Correction</summary>

```bash
kubectl api-resources
```

</details>

Quel sont les apiVersion des resources suivantes : `StorageClass` & `RoleBinding`

<details><summary>Correction</summary>
Il sont respectivement `storage.k8s.io/v1` et `rbac.authorization.k8s.io/v1`

```bash
kubectl api-resources | grep -i storageClass
kubectl api-resources | grep -i RoleBinding
```

</details>

Decriver la structure des objets `Pod` , `Deployment`, `PersistentVolume`

<details><summary>Correction</summary>

```bash
kubectl explain pods
kubectl explain pods --recursive
```

</details>

## Contexts des utilisateurs

Lister les contexts de votre environnement kubernetes. Combien de context avez vous ? quel est le context par défaut ?

<details><summary>Correction</summary>

```bash
kubectl config get-contexts
```

</details>

Visualisez la configuration de ce context par défaut. Comment est structuré cette configuration ? Afficher la version courte de la configuration.

<details><summary>Correction</summary>

```bash
kubectl config view 
```

On reconnait la structure générale des fichier YAML de Kubernetes
- Section `apiVersion : v1`
- Section `Kind : Config`
- Puis les autres sections:
  - `clusters` : qui fait la liste des clusters, les IP des API-Server 
  - `users` : qui fait la liste des utilisateurs avec leur crédentials (clés d'authentification)
  - `contexts` : qui fait la liste des context : c-a-d qui fait un matching entre user et cluster
  
Version courte

```bash
 kubectl config view --minify
```

</details>

Changer de context par defaut et choisisser un autre 

<details><summary>Correction</summary>

```bash
kubectl config use-context kubernetes-admin@k8s-lab
```

</details>

## Exploration des composants K8s déployés

Lister les Pods contenus dans le namespace `kube-system`

<details><summary>Correction</summary>

```bash
kubectl get pods -n kube-system 
```

</details>

Quel sont les composants que vous avez identifiés ?

<details><summary>Correction</summary>

Le Pod de l'API server kubernetes : Tous les autres composants sont reliés à l'[API server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
```bash
kube-apiserver-k8s-master-1                1/1     Running   1 (7d1h ago)   16d
```

[etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/) est une base de donnée clés-valeurs, cohérent et hautement disponible utilisé par Kubernetes (API Server) pour le stockage de toutes les données du cluster.
```bash
etcd-k8s-master-1                          1/1     Running   1 (7d1h ago)   16d
```

Pod du [Controller Manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) qui implémente une boucle de contrôle qui surveille l'état partagé du cluster par l'intermédiaire de l'API server et effectue des changements pour tenter de faire évoluer l'état actuel vers l'état souhaité.
```bash
kube-controller-manager-k8s-master-1       1/1     Running   1 (7d1h ago)   16d
```

Pod de l'ordonanceur [kube scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) qui détermine quels nœuds sont des placements valides pour chaque pod dans la file d'attente d'ordonnancement en fonction des contraintes et des ressources disponibles. 
```bash
kube-scheduler-k8s-master-1                1/1     Running   1 (7d1h ago)   11d
```

Pod [CoreDNS](https://coredns.io/). CoreDNS est un serveur DNS flexible et extensible qui peut servir de DNS de cluster Kubernetes. Il a donc remplacé `kube-dns`
```bash
coredns-565d847f94-9hhhz                   1/1     Running   1 (7d1h ago)   16d
coredns-565d847f94-zv8q4                   1/1     Running   1 (7d1h ago)   16d
```

les pods [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) qui s'exécutes sur chaque nœud. Responsable de la publication des Services Kubernetes sur chaque noeud du cluster. 
```bash
kube-proxy-888qt                           1/1     Running   1 (7d1h ago)   16d
kube-proxy-kc2t8                           1/1     Running   1 (7d1h ago)   16d
kube-proxy-sc44m                           1/1     Running   1 (7d1h ago)   16d
kube-proxy-z4jsr                           1/1     Running   1 (7d1h ago)   16d
```

Des Pods [Calico](https://projectcalico.docs.tigera.io/about/about-calico). Calico met en œuvre l'interface réseau pour conteneurs ([CNI](https://github.com/containernetworking/cni)) de Kubernetes en tant que plug-in et fournit des agents pour Kubernetes afin de mettre en réseau les conteneurs et les pods.
```bash
calico-kube-controllers-566654d67d-gs2nl   1/1     Running   1 (7d1h ago)   16d
calico-node-4nvnl                          1/1     Running   1 (7d1h ago)   16d
calico-node-rts9x                          1/1     Running   1 (7d1h ago)   16d
calico-node-shw7b                          1/1     Running   1 (7d1h ago)   16d
calico-node-tcr4s                          1/1     Running   1 (7d1h ago)   16d
```

</details>

Placez-vous sur le serveur master et visualisez les options avec lesquelles Kubelet est en cours d'exécution

_Si vous explorez avec `Minikube` étant données que tout cela tourne à l'intérieur d'un noeud virtual, pour y accéder, exécutez la commande suivante_

```bash
minikube ssh 
```
<details><summary>Correction</summary>

```bash
ps aux | grep -i kubelet

root        1847  6.1  5.0 1935700 101908 ?      Ssl  01:02  10:33 /var/lib/minikube/binaries/v1.25.3/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/var/run/cri-dockerd.sock --hostname-override=minikube --image-service-endpoint=/var/run/cri-dockerd.sock --kubeconfig=/etc/kubernetes/kubelet.conf --node-ip=192.168.49.2 --runtime-request-timeout=15m
```

</details>

A l'aide de la commande `kubectl describe pods -n kube-system <NOM_DU_POD>`, visualisez les options de lancement de chacun des Pods du Control Place : etcd, kube-scheduler, api-server...

<details><summary>Correction</summary>

```bash
kubectl describe pods -n kube-system etcd-minikube
```

```yaml
Name:                 etcd-minikube
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 minikube/192.168.49.2
Start Time:           Sun, 06 Nov 2022 01:02:17 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.49.2:2379
                      kubernetes.io/config.hash: bd495b7643dfc9d3194bd002e968bc3d
                      kubernetes.io/config.mirror: bd495b7643dfc9d3194bd002e968bc3d
                      kubernetes.io/config.seen: 2022-11-06T01:02:17.008978752Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   192.168.49.2
IPs:
  IP:           192.168.49.2
Controlled By:  Node/minikube
Containers:
  etcd:
    Container ID:  docker://ff6f7e04548eac6f4075cf8f293e9bc6693457f53689b16de0eea6ed4ac387ca
    Image:         registry.k8s.io/etcd:3.5.4-0
    Image ID:      docker-pullable://registry.k8s.io/etcd@sha256:6f72b851544986cb0921b53ea655ec04c36131248f16d4ad110cb3ca0c369dc1
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.49.2:2379
      --cert-file=/var/lib/minikube/certs/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/minikube/etcd
      --experimental-initial-corrupt-check=true
      --experimental-watch-progress-notify-interval=5s
      --initial-advertise-peer-urls=https://192.168.49.2:2380
      --initial-cluster=minikube=https://192.168.49.2:2380
      --key-file=/var/lib/minikube/certs/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.49.2:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.49.2:2380
      --name=minikube
      --peer-cert-file=/var/lib/minikube/certs/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/var/lib/minikube/certs/etcd/peer.key
      --peer-trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
      --proxy-refresh-interval=70000
      --snapshot-count=10000
      --trusted-ca-file=/var/lib/minikube/certs/etcd/ca.crt
    State:          Running
      Started:      Sun, 06 Nov 2022 01:02:09 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health%3Fexclude=NOSPACE&serializable=true delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health%3Fserializable=false delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /var/lib/minikube/certs/etcd from etcd-certs (rw)
      /var/lib/minikube/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/certs/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/minikube/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>
```

</details>
