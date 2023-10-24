# TOS-CALICO

![Simplified Networking, Security and Observability for Rancher Kubernetes  Engine with Calico | SUSE Communities](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ-J2g78-RncNtklzNcfSHSvhQM_hbghq42Hg&usqp=CAU)

# Introduction à la Mise en Place de Calico sur Kubernetes
Calico, un outil open-source, est essentiel pour la gestion de réseaux robustes et sécurisés dans les clusters Kubernetes. Ce tutoriel détaillé vous guide dans l'installation, la configuration, et les tests de Calico, vous permettant de renforcer la connectivité et la sécurité de vos applications. Peu importe votre niveau d'expérience, ce guide vous fournira les clés pour maximiser les avantages de Calico au sein de votre infrastructure Kubernetes. Plongeons dès maintenant dans ce processus essentiel pour votre environnement conteneurisé.

# Prérequis

1.  **Cluster Kubernetes fonctionnel** : Assurez-vous que votre cluster Kubernetes est correctement configuré et fonctionne sans problème. Vous pouvez utiliser des solutions telles que kubeadm, kops, ou GKE pour créer votre cluster.
    
2.  **kubectl** : Vérifiez que l'outil `kubectl` est installé et configuré pour interagir avec votre cluster Kubernetes. Vous pouvez le vérifier en exécutant la commande `kubectl version`.
    
3.  **Accès Internet** : Les nœuds de votre cluster Kubernetes doivent avoir un accès à Internet pour télécharger  les manifestes nécessaires à l'installation de Calico.
    
4.  **Plan d'adressage IP** : Définissez un plan d'adressage IP pour les pods de votre cluster. Calico utilise cet espace d'adressage pour assigner des adresses IP aux pods. Assurez-vous qu'il n'y a pas de collision avec d'autres sous-réseaux utilisés dans votre environnement..

**IMPORTANT** : Configurer Calico avant d'ajouter d'autres nœuds au nœud master.
    

En respectant ces prérequis, vous serez prêt à déployer et à configurer Calico sur votre cluster Kubernetes de manière efficace et sans problème.

## Étape 1: Installation de Calico

### 1.1. Téléchargement du manifeste
```shell
wget https://docs.projectcalico.org/manifests/calico.yaml
```
### 1.2. Appliquer le manifeste

`kubectl apply -f calico.yaml`

Attendez que tous les pods Calico soient en cours d'exécution :

`kubectl get pods -n kube-system`

## Étape 2 : Configuration de Calico

### 2.1. Personnalisation de la configuration (optionnel)
Vous pouvez personnaliser la configuration de Calico en modifiant le ConfigMap. Par exemple, pour changer le sous-réseau IP par défaut, éditez le ConfigMap :

`kubectl edit configmap -n kube-system calico-config`

Vérifiez que les ressources Calico sont correctement configurées :

`kubectl get felixconfigurations.crd.projectcalico.org `
`kubectl get bgpconfigurations.crd.projectcalico.org `
`kubectl get ippools.crd.projectcalico.org`

## Étape 3 : Test de Calico

Créez un exemple de déploiement pour tester le réseau Calico. Par exemple, un déploiement simple de Nginx :

```shell
apiVersion:  apps/v1  
kind:  Deployment  
metadata:  
	name:  nginx-deployment  
spec:  
	replicas:  2  
	selector:  
		matchLabels:  
			app:  nginx  
	template:  
		metadata:  
			labels:  
			app:  nginx  
	spec:  
		containers:  
		-  name:  nginx  
		- image:  nginx:latest
```
Enregistrez le fichier YAML ci-dessus (par exemple, `nginx-deployment.yaml`) et déployez-le avec `kubectl` :

`kubectl apply -f nginx-deployment.yaml`

Vérifiez que les pods Nginx sont en cours d'exécution :
```shell
kubectl get pods -A
```
## Étape 4 : Configuration de la politique réseau
Calico permet de définir des politiques réseau pour contrôler la communication entre les pods. Par exemple, pour autoriser le trafic HTTP :
```shell
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: allow-http
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
    - ports:
      - port: 80
      from:
      - podSelector:
          matchLabels:
            role: web
```
Enregistrez le fichier YAML ci-dessus (par exemple, `allow-http.yaml`) et appliquez la politique réseau :
`kubectl apply -f allow-http.yaml`

## Étape 5 : Vérification des politiques réseau
Vérifiez que la politique réseau est correctement configurée :

`kubectl get networkpolicies`

C'est tout ! Vous avez configuré et testé Calico avec Kubernetes.

