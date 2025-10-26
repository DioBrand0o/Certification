# Préparation CKA (Certified Kubernetes Administrator)

Ce dossier contient des **travaux pratiques (TP)** et des **notions générales** pour préparer l'examen **CKA**.

## Contenu

- **TP** : Exercices pratiques pour s'entraîner aux tâches de l'examen
- **Notions générales** : Concepts clés de Kubernetes à maîtriser

## Prérequis

Les prérequis varient selon les TP. Pour une expérience optimale avec **Kind** ou un Vrais Cluster, il est recommandé de créer un cluster multi-nœuds :

- **1 master (control-plane)**
- **1 ou 2 workers** (selon les ressources disponibles)

### Exemple de configuration Kind multi-nœuds :

```bash
cat <<EOF > kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF

kind create cluster --config kind-config.yaml --name cka-cluster
```

> **Note :** Certains TP peuvent fonctionner avec un seul nœud, mais un cluster multi-nœuds permet de pratiquer des concepts comme le scheduling, les taints/tolerations, et les affinity rules.

## ⚠️ Disclaimer

> **Ce contenu est fait pour aider à la préparation, mais il peut contenir des erreurs.**  
> Vérifiez toujours les informations avec la [documentation officielle Kubernetes](https://kubernetes.io/docs/).

## Objectif

Se préparer efficacement à l'examen CKA en pratiquant régulièrement.

---

**Bon courage pour la certification ! 🚀**


**Zack Alias Dio Brando**
