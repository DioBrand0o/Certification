# TP Kubernetes avec Kind - Formation CKA

**Durée estimée :** 1 heure  
**Niveau :** Débutant à Intermédiaire

---

## 🎯 Objectifs du TP

À la fin de ce TP, vous serez capable de :
- Installer et configurer Kind pour créer un cluster Kubernetes local
- Créer et gérer des pods
- Utiliser les labels pour organiser et sélectionner des ressources
- Déployer des applications avec Deployments
- Scaler et mettre à jour des applications
- Exposer des services
- Maîtriser les commandes kubectl essentielles

---

## 📋 Prérequis

### Logiciels nécessaires

- **Docker** : Installé et fonctionnel
- **kubectl** : Outil CLI pour Kubernetes
- **Kind** : Kubernetes IN Docker

### Vérification

```bash
# Vérifier Docker
docker --version

# Vérifier kubectl
kubectl version --client

# Vérifier Kind
kind version
```

---

## Partie 1 : Installation et Configuration

### Étape 1 : Créer un cluster Kind

```bash
# Créer un cluster nommé "cka-training"
kind create cluster --name cka-training

# Vérifier que le cluster est bien créé
kind get clusters

# Vérifier le contexte kubectl
kubectl cluster-info --context kind-cka-training
```

**Astuce :** Le nom par défaut est `kind` si vous ne spécifiez pas `--name`.

### Étape 2 : Vérifier le cluster

```bash
# Lister les nœuds
kubectl get nodes

# Obtenir plus de détails
kubectl get nodes -o wide

# Voir les namespaces
kubectl get namespaces
```

**Résultat attendu :**  
Vous devriez voir un nœud `kind-cka-training-control-plane` avec le statut `Ready`.

---

## Partie 2 : Création et Gestion des Pods

### Étape 3 : Créer votre premier pod

```bash
# Créer un pod nginx
kubectl run nginx-pod --image=nginx:latest

# Vérifier le pod
kubectl get pods

# Voir plus de détails
kubectl get pods -o wide
```

### Étape 4 : Inspecter le pod

```bash
# Décrire le pod (événements, configuration, etc.)
kubectl describe pod nginx-pod

# Voir les logs du pod
kubectl logs nginx-pod

# Se connecter au pod (shell interactif)
kubectl exec -it nginx-pod -- /bin/bash
# (tapez 'exit' pour sortir)
```

**Exercice :** Créez un deuxième pod avec l'image `redis:alpine` et nommez-le `redis-pod`.

```bash
kubectl run redis-pod --image=redis:alpine
kubectl get pods
```

---

## Partie 3 : Les Labels

Les labels sont des paires clé/valeur utilisées pour organiser et sélectionner des ressources.

### Étape 5 : Ajouter des labels

```bash
# Ajouter un label au pod nginx
kubectl label pod nginx-pod app=frontend tier=web

# Ajouter un label au pod redis
kubectl label pod redis-pod app=cache tier=backend

# Voir les labels
kubectl get pods --show-labels
```

### Étape 6 : Sélectionner avec les labels

```bash
# Lister les pods avec le label app=frontend
kubectl get pods -l app=frontend

# Lister les pods avec le label tier=backend
kubectl get pods -l tier=backend

# Lister les pods avec plusieurs conditions
kubectl get pods -l 'app in (frontend,cache)'
```

**🎯 Exercice :** Créez un pod `busybox` avec les labels `app=debug` et `tier=tools`.

---

## 📊 Partie 4 : Les Deployments

Les Deployments gèrent automatiquement les pods et permettent la mise à l'échelle.

### Étape 7 : Créer un deployment

```bash
# Créer un deployment avec 3 réplicas
kubectl create deployment webapp --image=nginx:1.21 --replicas=3

# Voir le deployment
kubectl get deployments

# Voir les pods créés
kubectl get pods -l app=webapp
```

### Étape 8 : Scaler le deployment

```bash
# Augmenter à 5 réplicas
kubectl scale deployment webapp --replicas=5

# Vérifier
kubectl get deployment webapp
kubectl get pods -l app=webapp
```

### Étape 9 : Mise à jour (Rolling Update)

```bash
# Mettre à jour l'image vers nginx:1.22
kubectl set image deployment/webapp nginx=nginx:1.22

# Suivre le déploiement
kubectl rollout status deployment/webapp

# Voir l'historique
kubectl rollout history deployment/webapp
```

### Étape 10 : Rollback

```bash
# Revenir à la version précédente
kubectl rollout undo deployment/webapp

# Vérifier
kubectl rollout status deployment/webapp
kubectl describe deployment webapp | grep Image
```

**Exercice :** Créez un deployment `api` avec 2 réplicas de l'image `httpd:alpine`, puis scalez-le à 4 réplicas.

---

## Partie 5 : Les Services

Les Services exposent les pods et permettent la communication.

### Étape 11 : Créer un service ClusterIP

```bash
# Créer un service pour le deployment webapp
kubectl expose deployment webapp --port=80 --target-port=80 --name=webapp-service

# Voir le service
kubectl get services

# Voir les endpoints (pods derrière le service)
kubectl get endpoints webapp-service
```

### Étape 12 : Tester le service

```bash
# Obtenir l'IP du service
kubectl get svc webapp-service

# Tester depuis un pod temporaire
kubectl run test-pod --rm -it --image=busybox -- sh
# Dans le pod, taper :
wget -qO- http://webapp-service
# Tapez 'exit' pour sortir
```

### Étape 13 : Service NodePort

```bash
# Créer un service NodePort
kubectl expose deployment webapp --port=80 --type=NodePort --name=webapp-nodeport

# Voir le port assigné
kubectl get svc webapp-nodeport
```

**Note :** Avec Kind, il faut utiliser `kubectl port-forward` pour accéder depuis votre machine :

```bash
kubectl port-forward service/webapp-nodeport 8080:80
# Ouvrez http://localhost:8080 dans votre navigateur
```

---

## Partie 6 : Nettoyage et Gestion

### Étape 14 : Supprimer des ressources

```bash
# Supprimer un pod
kubectl delete pod nginx-pod

# Supprimer un deployment (supprime aussi les pods)
kubectl delete deployment webapp

# Supprimer un service
kubectl delete service webapp-service

# Supprimer toutes les ressources avec un label
kubectl delete all -l app=frontend
```

### Étape 15 : Supprimer le cluster

```bash
# Lister les clusters
kind get clusters

# Supprimer le cluster
kind delete cluster --name cka-training

# Vérifier
kind get clusters
```

---

## Commandes kubectl Essentielles

### Commandes de base

| Commande | Description | Exemple |
|----------|-------------|---------|
| `kubectl get` | Lister les ressources | `kubectl get pods` |
| `kubectl describe` | Détails d'une ressource | `kubectl describe pod nginx` |
| `kubectl logs` | Voir les logs | `kubectl logs nginx-pod` |
| `kubectl exec` | Exécuter une commande | `kubectl exec -it nginx -- sh` |
| `kubectl delete` | Supprimer une ressource | `kubectl delete pod nginx` |
| `kubectl apply` | Appliquer un fichier YAML | `kubectl apply -f pod.yaml` |

### Commandes avancées

| Commande | Description | Exemple |
|----------|-------------|---------|
| `kubectl edit` | Éditer une ressource | `kubectl edit deployment webapp` |
| `kubectl scale` | Changer le nombre de replicas | `kubectl scale deploy webapp --replicas=3` |
| `kubectl rollout` | Gérer les déploiements | `kubectl rollout status deploy/webapp` |
| `kubectl label` | Gérer les labels | `kubectl label pod nginx app=web` |
| `kubectl port-forward` | Forward un port | `kubectl port-forward pod/nginx 8080:80` |

### Options utiles

| Option | Description | Exemple |
|--------|-------------|---------|
| `-o yaml` | Sortie en YAML | `kubectl get pod nginx -o yaml` |
| `-o wide` | Plus de détails | `kubectl get pods -o wide` |
| `--show-labels` | Afficher les labels | `kubectl get pods --show-labels` |
| `-l` | Filtrer par label | `kubectl get pods -l app=web` |
| `--all-namespaces` | Tous les namespaces | `kubectl get pods --all-namespaces` |

---

## Ressources et Prochaines Étapes

### Documentation officielle

- **Kubernetes :** [kubernetes.io/docs](https://kubernetes.io/docs)
- **Kind :** [kind.sigs.k8s.io](https://kind.sigs.k8s.io)
- **kubectl Cheat Sheet :** [kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### Ce que vous avez appris

- Installer et configurer Kind pour créer un cluster Kubernetes local
- Créer et gérer des pods
- Utiliser les labels pour organiser et sélectionner des ressources
- Déployer des applications avec Deployments
- Scaler et mettre à jour des applications
- Exposer des services
- Maîtriser les commandes kubectl essentielles

---

## ⚠️ Avertissement

> Ce document est créé à des fins pédagogiques pour la préparation de la certification CKA.  
> Vérifiez toujours les informations avec la [documentation officielle Kubernetes](https://kubernetes.io/docs/).

---
