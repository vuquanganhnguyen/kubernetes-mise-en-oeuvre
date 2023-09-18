# Deployment


De la manière déclarative, créez un déploiement du nom de `robots`, contenant 2 pods, l'image est busybox et la commande qu'il doit exécuter est le script suivant `while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done`

<details><summary>Correction</summary>

Pour générer les fichier YAML correspondant, il existe une astuce simple qui est de combiner l'utilisation de kubectl avec l'option `--dry-run=client -o yaml` 
l'option `--dry-run=client`indique à l'PAI server que nous somme entrain de faire des simulations et qu'il ne devra pas créer la ressource
l'otion `- o yaml` est de dire que nous voulons un output de l'objet au format YAML. 

```bash
# Etape 1 => créer un fichier du script

$ cat << _EOF > script.sh
#!/bin/sh
while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done
_EOF 

# Etape 2 => créer le fichier YAML du déploiement

$ kubectl create deployment robots --replicas 2  --dry-run=client -o yaml --image busybox -- /bin/sh -c "`cat script.sh`" > robots.yml

# Etape 3 => Visualisez le manifest créé et apportez des modifications si nécessaires
$ cat robots.yml
```

Le fichier YAML créé :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: robots
  name: robots
spec:
  replicas: 2
  selector:
    matchLabels:
      app: robots
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: robots
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - |-
          #!/bin/sh
          while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done
        image: busybox
        name: busybox
        resources: {}
status: {}
```

Pour déployer donc nous ferons

```bash
kubectl apply -f robots.yml
```

Normalement si le déploiement est bien fait, vous pouvez voir les logs des pods


```bash
$ kubectl logs -f robots-577dbd8d5d-25mdb
F1buq9n5dhonu
QpcV5rcnf5yqm
eSn3raQVvBf0T
TA92AZM9Asjqr
```

</details>


## Autres notions variées

Créer un déployement du nom de `custom-web` de 3 replicas. Le Pod déployé doit avoir de deux conteneurs partageant le même espace de stockage temporaire `/usr/share/nginx/html/`
  
  * les deux conteneurs doivent monté un volume de type `EmptyDir: {}` mappé sur `/usr/share/nginx/html/`
  * Le premier conteneur est Nginx
  * le second conteneur est busybox qui exécute un script boucle infini qui écrira dans le fichier `/usr/share/nginx/html/index.html` les informations clée suivante : le nom de l'hôte sur lequel le Pod est exécuté
  * Appliquez le déploiement
  
<details><summary>Correction</summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: custom-web
  name: custom-web
spec:
  replicas: 4
  selector:
    matchLabels:
      app: custom-web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: custom-web
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - |-
          #!/bin/sh
                while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 >> /usr/share/nginx/html/index.html  ; echo ' running on : '>> /usr/share/nginx/html/index.html; echo  ${NODE_NAME} >> /usr/share/nginx/html/index.html && sleep 5; done
        image: busybox
        name: busybox
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        volumeMounts:
        - name: htmldir
          mountPath: /usr/share/nginx/html/
      - image: nginx
        name: nginx
        volumeMounts:
        - name: htmldir
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: htmldir
        emptyDir: {}
```

```bash
$ kubectl apply -f custom-web.yml
deployment.apps/custom-web created

$ kubectl logs custom-web-58f86dc44-8ghgq -c nginx                                                                  1 ✘ ╱ base  ╱ at mawaki-k8s-lab ⎈ ╱ at 16:48:19 
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/10/05 14:42:33 [notice] 1#1: using the "epoll" event method
2022/10/05 14:42:33 [notice] 1#1: nginx/1.23.1
2022/10/05 14:42:33 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/10/05 14:42:33 [notice] 1#1: OS: Linux 5.10.133+
2022/10/05 14:42:33 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/10/05 14:42:33 [notice] 1#1: start worker processes
2022/10/05 14:42:33 [notice] 1#1: start worker process 31
2022/10/05 14:42:33 [notice] 1#1: start worker process 32
10.128.0.13 - - [05/Oct/2022:14:45:46 +0000] "GET / HTTP/1.1" 200 2960 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36" "-"
10.128.0.13 - - [05/Oct/2022:14:45:46 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://35.223.150.115/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36" "-"
2022/10/05 14:45:46 [error] 31#31: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.128.0.13, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "35.223.150.115", referrer: "http://35.223.150.115/"
```

</details>


Créer un service de type `ClusterIp` qui expose ce déploiement et constatez que un curl depuis un Pod de test ( `image : curlimages/curl` ) affiche ce que le busybox écrit

<details><summary>Correction</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: custom-web
  labels:
    app: custom-web
spec:
  ports:
    - port: 80
  selector:
    app: custom-web
  type: ClusterIP
```

```bash
$ kubectl apply -f custom-web-service.yml
service/custom-web created
```

```bash
$ kubectl run client --rm  -it --image curlimages/curl -- sh
If you don't see a command prompt, try pressing enter.
/ $ curl http://custom-web
/ $
```

</details>


  * Modifier le déploiement pour configurer une antiafinité entre eux de sorte que deux pods ne se trouvent pas sur le même node
  * Appliquez à nouveau le déploiement et constatez bien que l'antiaffinité est respecté
  * Modifier le déploiement pour limiter les ressources RAM et CPU des conteneurs Nginx
  * Appliquez à nouveau le déploiement et constatez bien que les limites sont respecté

