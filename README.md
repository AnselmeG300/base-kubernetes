TP0: PRESENTATION ET CODE D'ACCES A LA PLATEFORME EAZYTRAINING

TP1: COMPOSANT ET INSTALLATION DE MINIKUBE

Il est important de noter que lors de cette formation nous n'allons pas faire l'installation d'un cluster complet, nous allons travailler sur du single node ie sur un seul noeud déployé sur une VM ayant comme OS centos 7

Alors nous allons installer minikube qui est une version miniaturise de Kubernetes utilise pour les cours de formation ou pour les tests  comme dans le cadre de ce cours (https://kubernetes.io/fr/docs/setup/learning-environment/minikube/)


ATTENTION : POUR INSTALLER MINIKUBE IL VOUS FAUT UNE VM AVEC AU MOINS 2 GO ET 2 CPU MINIMUM

# sudo yum -y update
# sudo yum -y install epel-release
EPEL (Extra Package for Entreprise Linux) est un dépôt qui fournit des paquets additionnels pour les distributions basées sur RedHat. En installant EPEL vous aurez un nombre conséquent de paquets disponibles via le gestionnaire de paquets yum

# sudo yum -y install git libvirt qemu-kvm virt-install virt-top libguestfs-tools bridge-utils 
Ensuite on installe les paquets libvirt qemu-kvm vont permettre d'installer la VM minikube qui contient
kubernetes
# sudo yum install socat -y  
qui permet de faire du fowarding
# sudo yum install -y conntrack 
qui permet de gerer l'utilisation du processeur sur notre machine


Ensuite nous allons installer Docker a partir de scon script d'installation (https://get.docker.com/)

Pourquoi installer docker? Le problème est que l'on ne peut pas faire de la virtualisation sur notre VM car
elle est interdit alors pour pouvoir avoir les composants de kubernetes sur notre PC nous allons utiliser la
technologie de conteneurisation pour dire à minikube de récupérer chaque composant de Kubernetes sous forme
de conteneur comme : DNS? API SERVER? kube SCHEDULER, MANAGER et ETCD

1. télécharger le script
# curl -fsSL https://get.docker.com -o install-docker.sh
2. vérifier le contenu du script
# cat install-docker.sh
3. exécuter le script avec --dry-run pour vérifier les étapes qu'il exécute
# sh install-docker.sh --dry-run
4. exécutez le script soit en tant que root, soit en utilisant sudo pour effectuer l'installation.
# sudo sh install-docker.sh
5. donner les droits d'excution sur le service docker a l'utlisateur centos
# sudo usermod -aG docker centos
7. lancer le service docker
# sudo systemctl start docker

pour récupérer les différents binaires que l'on souhaiterait avoir
# sudo yum -y install wget 
on techarge le binaire de minikube
# sudo wget https://storage.googleapis.com/minikube/releases/v1.28.0/minikube-linux-amd64
on le rend executable
# sudo chmod +x minikube-linux-amd64 
on le deplace dans un repertoire qui nous permettra de l'executer n'importe où dans notre VM
# sudo mv minikube-linux-amd64 /usr/bin/minikube 
on installe l'utlitaire de ligne de commande de minikube  kubectl
# sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.0/bin/linux/amd64/kubectl  
doc de kubectl (https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
on le rend executable 
# sudo chmod +x kubectl 
on le deplace dans un repertoire ou on pourra pour pour acceder a sa cmde depuis n'importe ou dans notre VM
# sudo mv kubectl  /usr/bin/ 

Configurer le forwarding au niveau des interfaces reseaux 
# sudo echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
Activer le service docker au demarrage
# sudo systemctl enable docker.service
nous installons minikube
# minikube start –driver=none –kubernetes-version v1.23.0
nous pouvons presenter les composants que minikube a telechargé en tapant 
# docker images
une fois l'installation terminé on peut lister les nodes en tapant 
Vérifier que le cluster est fonctionnel
# kubectl get nodes
# kubectl get pod -A
# systemctl status kubelet.service

pour l'autocompletion
# sudo yum install bash-completion -y


TP 2 DEPLOYEZ VOTRE PREMIERE APPLICATION :

se connecter en ssh avec VSCode

ssh -i 

- Créez un repertoire Kubernetes-training et un sous-dossier tp-2 et copiez vos manifests à l’interieur
    mkdir -p Kubernetes-training/tp-2
    cd Kubernetes-training/tp-2

- Ecrivez un manifest pod.yml pour déployer un pod avec l’image 
  mmumshad/simple-webapp-color  en précisant que la color souhaitée est la rouge
    => Le manifeste vous est fourni par le formatteur

apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    app: web
spec:
  containers:
  - name: web
    image: mmumshad/simple-webapp-color
    ports:
      - containerPort: 8080
    env:
      - name: APP_COLOR
        value: red

apiVersion: v1 et kind: Pod spécifient le type d'objet Kubernetes que nous créons.
metadata contient des informations sur le Pod, telles que son nom et ses étiquettes.
spec contient la spécification du Pod, qui définit les conteneurs qu'il contient et comment ils doivent être exécutés.
containers est une liste de tous les conteneurs à exécuter dans le Pod. Dans ce cas, il y a un seul conteneur nommé "web".
image spécifie l'image Docker à utiliser pour le conteneur. Dans ce cas, l'image est "mmumshad/simple-webapp-color".
ports spécifie les ports du conteneur à exposer. Dans ce cas, le port 8080 est exposé dans le cluster.
env définit les variables d'environnement à passer au conteneur. Dans ce cas, il y a une variable d'environnement nommée "APP_COLOR" avec une valeur "red".

lancez votre pod  
# kubectl apply -f pod.yml
vérifiez qu’il est bien en cours d’exécution
# kubectl get po
# kubectl describe po simple-webapp-color
exposez votre pod en utilisant la commande 
# kubectl port-forward <nom de votre pod> 8080:8080 –-address 0.0.0.0
# kubectl port-forward simple-webapp-color 8090:8080 --address 0.0.0.0
vérifiez que l’application est bien joignagle en ouvrant votre VM sur le port 8080
supprimez votre pod
# kubectl delete -f pod.yml
vérifiez que l’application est bien supprimer en actualisant votre VM sur le port 8080 
ou en tappant la cmde
# kubectl get po

Nous avons creer un objet de type pod, cependant nous allons passer à la creation d'un objet de type deploiement
2 replicas d’un pod nginx

creer le fichier nginx-deployment.yml
Mettre le contenu suivant 

apiVersion: apps/v1 spécifie la version de l'API Kubernetes utilisée pour créer cet objet
kind: Deployment # spécifie le type d'objet Kubernetes créé, ici un déploiement
metadata: # contient des informations sur le déploiement, telles que le nom et les étiquettes pour identifier notre ressource Kubernetes
  name: nginx-deployment
  labels:
    app: nginx
spec: # contient les détails du déploiement, tels que le nombre de réplicas, la stratégie de mise à jour, le sélecteur et le templ de pod
  replicas: 2 # Nous définissons deux réplicas pour l'application Nginx.
  strategy:
    type: RollingUpdate # Nous definissons la stratégie de mise à jour qui est une mise à jour en douceur (RollingUpdate) qui permet de mettre à jour nos pods un par un, sans interruption de service
    rollingUpdate: Nous definissons egalement les options de maxSurge et maxUnavailable pour contrôler le nombre maximal de pods mis à jour et inactif simultanément
      maxSurge: 1
      maxUnavailable: 1
  selector: La section selector est utilisée pour sélectionner les pods à mettre à jour en fonction des étiquettes. Ici, nous utilisons l'étiquette "app: nginx" pour sélectionner les pods Nginx.
    matchLabels:
      app: nginx
  template: Dans cette section nous allons définir le modèle de pod qui sera créé.
    metadata:
      labels:
        app: nginx # Nous définissons une étiquette "app: nginx" pour identifier les pods Nginx,
    spec:
      containers: nous configurons un conteneur Nginx avec un port d'écoute sur le port 80
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

notre script mis en place, lacons notre objet deployment 
# kubectl apply -f nginx-deployment.yml
verifions le nombre de nos objets de type deployment
# kubectl get deploy
verifions le nombre de nos objets de type replicaset
# kubectl get replicaset
verifions le nombre de nos objets de type pop
# kubectl get po

passons a la version latest de nginx
pour cela on va mettre a jour la version de nginx dans le container
ensuite tapons
# kubectl apply -f nginx-deployment.yml
on peut constater que le deployment reconfigure notre ressource

pour suivre l'evolution de la mise a jour de notre version on tape 
# kubectl get replicaset -o wide

pour visualiser l'évolution de notre cluster
# watch kubectl get all

pour visualiser l'historique de notre deploiement 
# kubectl rollout history deployment/nginx-deployment

pour faire un rollback
# kubectl rollout undo deployment/nginx-deployment

Nous avons terminé avec l'approche déclarative, nous allons continué avec l'approche impérative pour pouvoir créé nos ressources

Supprimez toutes les ressources créées et recréez les en utilisant 
  les commandes impératives

    + lancement du pod :    
# kubectl run --image=mmumshad/simple-webapp-color --env="APP_COLOR=red"  --restart=Never simple-webapp-color
    + Suppression du pod :  
# kubectl delete pod simple-webapp-color
    + Lancement du deploy:  
# kubectl create deployment --image=nginx:1.18.0 nginx-deployment
    + Scaling du replicas:  
# kubectl scale --replicas=2 deployment/nginx-deployment
    + upgrade de version :  
# kubectl set image deployment/nginx-deployment nginx=nginx 

Enfin, poussez ce de tp sur votre github afin de conservez tous vos fichiers
    cd ..
    git init
    git add . 
    git commit -m "message de commit personnalisé"
    git remote add origin http://url_repos_git
    git push origin main