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

- Lancez votre pod et vérifiez qu’il est bien en cours d’exécution
    kubectl apply -f pod.yml
    kubectl get po
    kubectl describe po simple-webapp-color


