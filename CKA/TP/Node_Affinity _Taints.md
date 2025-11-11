# Node Affinity & Taints - Formation Interactive Kubernetes ENV TALOS ! ! ! 

> **Node Affinity & Taints : Décider où tournent tes Pods** 


---

## 📋 Table des matières

1. [Exercice 1 : Node Affinity / Anti-Affinity](#exercice-1--node-affinity--anti-affinity)
2. [Exercice 2 : Taints & Tolerations](#exercice-2--taints--tolerations)
3. [Récapitulatif et bonnes pratiques](#-récapitulatif-final)

---

## Exercice 1 : Node Affinity / Anti-Affinity

**Difficulté** : 🟡 Moyen  
**Objectif** : Contrôler sur quel node un pod doit être schedulé en utilisant les Node Affinity.

---

### 🤔 Comprendre avant d'agir : Node Affinity, c'est quoi ?

**Node Affinity** = Les **préférences/règles** pour placer un pod sur certains nodes spécifiques.

#### 📦 Cas d'usage

- Placer des pods sur des nodes avec GPU pour du machine learning
- Isoler des workloads sur des nodes dédiés (prod vs dev)
- Placer des pods proche de certaines ressources (stockage local, etc.)

#### Différence avec nodeSelector

| Critère | nodeSelector | nodeAffinity |
|---------|--------------|--------------|
| Simplicité | ✅ Simple | ⚠️ Plus complexe |
| Puissance | ❌ Limité (matching exact) | ✅ Opérateurs avancés (In, NotIn, Exists, etc.) |
| Cas d'usage | Labels simples | Règles complexes |

---

### Ton Cluster

Tu as **3 nodes** dans ton cluster :

- 🎛️ **c0** : Control plane (avec taint NoSchedule par défaut)
- 🖥️ **w0** : Worker node 1
- 🖥️ **w1** : Worker node 2

---

### 🤔 Réfléchis d'abord !

#### Question 1 : Voir les labels des nodes

Avant de créer une affinity, tu dois savoir quels **labels** sont disponibles sur tes nodes.

À ton avis, quelle commande permet de voir les labels d'un node ?

- `kubectl get nodes` → Affiche les nodes, mais pas les labels détaillés
- `kubectl describe node w0` → Affiche tout (trop d'infos)
- `kubectl get nodes --show-labels` → Affiche les nodes avec leurs labels

**Indice** : Il y a aussi une option pour voir UNIQUEMENT certains labels...

#### Question 2 : Ajouter un label custom

Par défaut, Kubernetes ajoute des labels comme `kubernetes.io/hostname`, mais pour cet exercice, on va créer nos propres labels.

Tu veux labelliser :
- **w0** avec le label `workload=web`
- **w1** avec le label `workload=batch`

Comment fait-on pour ajouter un label à un node ?

#### Question 3 : Affinity obligatoire vs préférée

Il existe 2 types de Node Affinity :

- **requiredDuringSchedulingIgnoredDuringExecution** → OBLIGATOIRE (hard constraint)
- **preferredDuringSchedulingIgnoredDuringExecution** → PRÉFÉRÉE (soft constraint)

À ton avis, quelle est la différence ?

- Si "required" et qu'aucun node ne match → Que se passe-t-il ?
- Si "preferred" et qu'aucun node ne match → Que se passe-t-il ?

---

### Étape 1 : Explorer les labels actuels

```bash
# Voir tous les nodes avec leurs labels
kubectl get nodes --show-labels

# Output attendu
NAME   STATUS   ROLES           AGE   VERSION   LABELS
c0     Ready    control-plane   20d   v1.33.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=c0,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
w0     Ready    <none>          20d   v1.33.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=w0,kubernetes.io/os=linux
w1     Ready    <none>          20d   v1.33.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=w1,kubernetes.io/os=linux

# Voir uniquement le label hostname
kubectl get nodes -L kubernetes.io/hostname

# Output attendu
NAME   STATUS   ROLES           AGE   VERSION   HOSTNAME
c0     Ready    control-plane   20d   v1.33.3   c0
w0     Ready    <none>          20d   v1.33.3   w0
w1     Ready    <none>          20d   v1.33.3   w1
```

> [!NOTE]
> **Labels par défaut intéressants :**
> - `kubernetes.io/hostname` → Nom du node (c0, w0, w1)
> - `kubernetes.io/arch` → Architecture (amd64, arm64)
> - `kubernetes.io/os` → Système d'exploitation (linux, windows)
> - `node-role.kubernetes.io/control-plane` → Node control plane

---

### Étape 2 : Ajouter des labels custom

```bash
# Ajouter le label workload=web sur w0
kubectl label node w0 workload=web
# Output: node/w0 labeled

# Ajouter le label workload=batch sur w1
kubectl label node w1 workload=batch
# Output: node/w1 labeled

# Vérifier les nouveaux labels
kubectl get nodes -L workload

# Output attendu
NAME   STATUS   ROLES           AGE   VERSION   WORKLOAD
c0     Ready    control-plane   20d   v1.33.3
w0     Ready    <none>          20d   v1.33.3   web
w1     Ready    <none>          20d   v1.33.3   batch
```

> [!TIP]
> **Labels ajoutés avec succès !** Maintenant on peut créer des affinity basées sur ces labels.

---

### Étape 3 : Créer un pod avec Node Affinity OBLIGATOIRE

On veut créer un pod qui DOIT absolument tourner sur **w1** (le node avec `workload=batch`).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-required
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - batch
  containers:
  - name: nginx
    image: nginx
```

#### Décryptage du YAML

| Champ | Description |
|-------|-------------|
| `affinity.nodeAffinity` | Section affinity pour les nodes |
| `requiredDuringSchedulingIgnoredDuringExecution` | OBLIGATOIRE (hard constraint) |
| `nodeSelectorTerms` | Liste de conditions (au moins UNE doit matcher) |
| `matchExpressions` | Conditions de matching |
| `key: workload` | Le label à vérifier |
| `operator: In` | Opérateur (In, NotIn, Exists, DoesNotExist, Gt, Lt) |
| `values: [batch]` | Valeurs acceptées |

**Traduction** : "Le pod doit être placé sur un node ayant le label `workload` avec la valeur `batch`"

```bash
# Créer le pod
kubectl apply -f pod-affinity-required.yaml
# Output: pod/pod-affinity-required created

# Vérifier sur quel node il est placé
kubectl get pod pod-affinity-required -o wide

# Output attendu
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-affinity-required   1/1     Running   0          5s    10.244.2.5   w1     <none>           <none>
```

> [!TIP]
> **Le pod est bien sur w1 !** La Node Affinity obligatoire a fonctionné.

---

### Étape 4 : Tester avec une affinity impossible

Que se passe-t-il si on demande un label qui n'existe sur AUCUN node ?

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-impossible
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: In
            values:
            - gpu  # ❌ Aucun node n'a workload=gpu
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f pod-affinity-impossible.yaml
# Output: pod/pod-affinity-impossible created

# Vérifier l'état du pod
kubectl get pod pod-affinity-impossible

# Output attendu
NAME                      READY   STATUS    RESTARTS   AGE
pod-affinity-impossible   0/1     Pending   0          30s

# Voir pourquoi il est Pending
kubectl describe pod pod-affinity-impossible

# Events:
#   Type     Reason            Age   From               Message
#   ----     ------            ----  ----               -------
#   Warning  FailedScheduling  20s   default-scheduler  0/3 nodes are available: 3 node(s) didn't match Pod's node affinity/selector.
```

> [!WARNING]
> **Pod en Pending !** Avec une affinity **required**, si aucun node ne match, le pod reste **Pending** indéfiniment.

```bash
# Nettoyer
kubectl delete pod pod-affinity-impossible
```

---

### Étape 5 : Node Affinity PRÉFÉRÉE (soft constraint)

Avec **preferred**, Kubernetes **tente** de placer le pod sur le node demandé, mais si impossible, il le place ailleurs.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: workload
            operator: In
            values:
            - web
  containers:
  - name: nginx
    image: nginx
```

#### Nouveauté

- `preferredDuringSchedulingIgnoredDuringExecution` → PRÉFÉRÉE (soft constraint)
- `weight: 100` → Poids de la préférence (1-100, plus élevé = plus prioritaire)
- Si plusieurs préférences, Kubernetes calcule un score total pour chaque node

```bash
kubectl apply -f pod-affinity-preferred.yaml

kubectl get pod pod-affinity-preferred -o wide

# Output attendu
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-affinity-preferred   1/1     Running   0          5s    10.244.1.8   w0     <none>           <none>
```

> [!TIP]
> **Le pod est placé sur w0 !** Kubernetes a respecté la préférence (`workload=web`).

#### Maintenant testons avec une préférence impossible

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-preferred-fallback
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: workload
            operator: In
            values:
            - gpu  # ❌ Aucun node n'a ce label
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f pod-affinity-preferred-fallback.yaml

kubectl get pod pod-affinity-preferred-fallback -o wide

# Output attendu
NAME                              READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-affinity-preferred-fallback   1/1     Running   0          5s    10.244.1.9   w0     <none>           <none>
```

> [!TIP]
> **Le pod est placé quand même !** Avec `preferred`, si aucun node ne match, Kubernetes place le pod sur n'importe quel node disponible.

---

### Étape 6 : Node Anti-Affinity (NotIn)

On peut aussi dire "NE PAS placer le pod sur certains nodes".

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-anti-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: workload
            operator: NotIn
            values:
            - batch
  containers:
  - name: nginx
    image: nginx
```

> [!NOTE]
> **Opérateur NotIn :** Le pod sera placé sur un node qui **n'a PAS** le label `workload=batch`.

```bash
kubectl apply -f pod-anti-affinity.yaml

kubectl get pod pod-anti-affinity -o wide

# Output attendu
NAME                READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-anti-affinity   1/1     Running   0          5s    10.244.1.10  w0     <none>           <none>
```

> [!TIP]
> **Le pod évite w1 !** Il est placé sur w0 (ou potentiellement c0 si le taint était enlevé).

---

### Étape 7 : Nettoyer

```bash
# Supprimer tous les pods
kubectl delete pod pod-affinity-required pod-affinity-preferred pod-affinity-preferred-fallback pod-anti-affinity

# (Optionnel) Supprimer les labels custom
kubectl label node w0 workload-
kubectl label node w1 workload-
```

---

### ASTUCE CKA

**Commande utile :**

```bash
# Générer le squelette YAML avec kubectl explain
kubectl explain pod.spec.affinity.nodeAffinity
```

Cette commande affiche la documentation intégrée sur la structure de nodeAffinity !

---

---

## Exercice 2 : Taints & Tolerations

**Difficulté** : Difficile  
**Objectif** : Contrôler quels pods PEUVENT être schedulés sur certains nodes en utilisant les Taints et Tolerations.

---

### 🤔 Comprendre avant d'agir : Taints & Tolerations, c'est quoi ?

**Taint** = Une "répulsion" appliquée sur un **node** qui **empêche** les pods d'y être placés.

**Toleration** = Une "tolérance" déclarée dans un **pod** qui lui permet de **supporter** un taint.

**Analogie :** Le taint est comme une barrière, et la toleration est le badge qui permet de passer.

#### Différence avec Node Affinity

| Concept | Comportement | Modèle |
|---------|--------------|--------|
| **Node Affinity** | Les pods **choisissent** où aller | Pull model |
| **Taints** | Les nodes **repoussent** les pods | Push model |

---

### Les 3 effets de Taint

Quand un node a un taint, il existe 3 comportements possibles :

| Effet | Description | Impact pods existants | Impact nouveaux pods |
|-------|-------------|----------------------|---------------------|
| **NoSchedule** | Aucun nouveau pod ne peut être placé | ✅ Restent | ❌ Bloqués |
| **PreferNoSchedule** | Évite de placer, mais accepte si pas d'autre choix | ✅ Restent | ⚠️ Évités |
| **NoExecute** | Aucun pod ne peut rester | ❌ Éjectés | ❌ Bloqués |

---

###  Réfléchis d'abord !

#### Question 1 : Le taint du control plane

Par défaut, le node **c0** (control plane) a un taint. C'est pour ça que les pods applicatifs ne sont jamais placés dessus.

À ton avis :
- Quel est ce taint ?
- Quel est son effet ?
- Comment voir les taints d'un node ?

#### Question 2 : Ajouter un taint

Tu veux réserver le node **w0** uniquement pour des pods "production".

Comment ajouter un taint sur w0 avec :
- Key: `environment`
- Value: `production`
- Effect: `NoSchedule`

#### Question 3 : Toleration syntax

Pour qu'un pod puisse être placé sur w0, il doit avoir une **toleration** matching le taint.

À ton avis, où se place la section `tolerations` dans le manifest du pod ?
- Dans `spec.containers[].tolerations` ?
- Dans `spec.tolerations` ?
- Dans `metadata.tolerations` ?

---

### Étape 1 : Voir les taints existants

```bash
# Voir les taints de tous les nodes
kubectl describe nodes | grep -i taint

# Output attendu
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>

# Ou voir un node spécifique
kubectl describe node c0 | grep -i taint

# Output attendu
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

> [!NOTE]
> **Explication :**
> - Le control plane **c0** a le taint `node-role.kubernetes.io/control-plane:NoSchedule`
> - Cela empêche les pods utilisateur d'y être placés
> - Seuls les pods système (kube-system) ont des tolerations pour ce taint
> - Les workers **w0** et **w1** n'ont aucun taint par défaut

---

### Étape 2 : Ajouter un taint sur w0

On va "réserver" le node w0 uniquement pour les pods de production.

```bash
# Ajouter le taint sur w0
kubectl taint nodes w0 environment=production:NoSchedule
# Output: node/w0 tainted

# Vérifier
kubectl describe node w0 | grep -i taint
# Output: Taints:             environment=production:NoSchedule
```

> [!NOTE]
> **Syntaxe du taint :** `kubectl taint nodes <NODE> <KEY>=<VALUE>:<EFFECT>`
> - `KEY` : `environment`
> - `VALUE` : `production`
> - `EFFECT` : `NoSchedule`

---

### Étape 3 : Tester avec un pod SANS toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-toleration
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f pod-no-toleration.yaml

# Vérifier sur quel node il est placé
kubectl get pod pod-no-toleration -o wide

# Output attendu
NAME                 READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-no-toleration    1/1     Running   0          5s    10.244.2.15  w1     <none>           <none>
```

> [!TIP]
> **Le pod est sur w1 !** Il évite automatiquement w0 (qui a un taint) et c0 (qui a aussi un taint).

> [!WARNING]
> **Important :** Sans toleration, le pod ne peut PAS aller sur w0, même si on utilise nodeSelector ou nodeAffinity !

---

### Étape 4 : Créer un pod AVEC toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-toleration
spec:
  tolerations:
  - key: environment
    operator: Equal
    value: production
    effect: NoSchedule
  containers:
  - name: nginx
    image: nginx
```

#### Décryptage de la toleration

| Champ | Description |
|-------|-------------|
| `key: environment` | Doit matcher la key du taint |
| `operator: Equal` | Vérifie l'égalité (ou `Exists` pour ignorer la value) |
| `value: production` | Doit matcher la value du taint |
| `effect: NoSchedule` | Doit matcher l'effect du taint |

**Traduction :** "Ce pod tolère le taint `environment=production:NoSchedule`"

```bash
kubectl apply -f pod-with-toleration.yaml

kubectl get pod pod-with-toleration -o wide

# Output attendu
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-with-toleration    1/1     Running   0          5s    10.244.1.20  w0     <none>           <none>
```

> [!TIP]
> **Le pod PEUT être placé sur w0 !** (Ou sur w1, car la toleration ne force PAS le placement, elle l'autorise)

> [!WARNING]
> **Attention :** Une toleration seule ne **garantit PAS** que le pod ira sur w0. Elle dit juste "je peux y aller si besoin". Pour forcer, il faut combiner avec nodeSelector ou nodeAffinity !

---

### Étape 5 : Combiner Toleration + nodeSelector

Pour **garantir** que le pod soit placé sur w0, on combine toleration + nodeSelector.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-guaranteed-w0
spec:
  nodeSelector:
    kubernetes.io/hostname: w0
  tolerations:
  - key: environment
    operator: Equal
    value: production
    effect: NoSchedule
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f pod-guaranteed-w0.yaml

kubectl get pod pod-guaranteed-w0 -o wide

# Output attendu
NAME                 READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-guaranteed-w0    1/1     Running   0          5s    10.244.1.21  w0     <none>           <none>
```

> [!TIP]
> **Placement garanti sur w0 !**
> - `nodeSelector` → Force le placement sur w0
> - `tolerations` → Autorise le pod à supporter le taint de w0

---
### Étape 6 : Tester l'effet NoExecute

L'effet **NoExecute** est plus radical : il **éjecte** les pods existants qui n'ont pas de toleration.

```bash
# D'abord, créer un pod sur w1 (sans taint)
kubectl run test-pod --image=nginx

# Vérifier qu'il est sur w1
kubectl get pod test-pod -o wide

# Output attendu
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
test-pod   1/1     Running   0          5s    10.244.2.22  w1     <none>           <none>

# Ajouter un taint NoExecute sur w1
kubectl taint nodes w1 maintenance=true:NoExecute
# Output: node/w1 tainted

# Observer immédiatement
kubectl get pod test-pod -o wide

# Output attendu
NAME       READY   STATUS        RESTARTS   AGE   IP       NODE   NOMINATED NODE   READINESS GATES
test-pod   1/1     Terminating   0          20s   <none>  w1     <none>           <none>
```

> [!WARNING]
> **Le pod est éjecté !** L'effet `NoExecute` termine les pods existants qui n'ont pas de toleration matching.

```bash
# Après quelques secondes
kubectl get pod test-pod
# Output: Error from server (NotFound): pods "test-pod" not found
```

> [!NOTE]
> **Cas d'usage de NoExecute :** Maintenance d'un node, évacuation de pods avant shutdown, isolation temporaire.

---

### Étape 7 : Toleration avec operator Exists

L'opérateur `Exists` permet de tolérer un taint **sans vérifier sa value**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-tolerate-any-maintenance
spec:
  tolerations:
  - key: maintenance
    operator: Exists
    effect: NoExecute
  containers:
  - name: nginx
    image: nginx
```

> [!NOTE]
> **operator: Exists :** "Je tolère le taint avec cette `key`, peu importe sa `value`."

```bash
kubectl apply -f pod-tolerate-any-maintenance.yaml

kubectl get pod pod-tolerate-any-maintenance -o wide

# Output attendu
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE   NOMINATED NODE   READINESS GATES
pod-tolerate-any-maintenance  1/1     Running   0          5s    10.244.2.25  w1     <none>           <none>
```

> [!TIP]
> **Le pod peut rester sur w1 !** Malgré le taint `maintenance=true:NoExecute`, ce pod a une toleration qui le protège.

---

### Étape 8 : Nettoyer les taints et pods

```bash
# Supprimer tous les pods de test
kubectl delete pod pod-no-toleration pod-with-toleration pod-guaranteed-w0 pod-tolerate-any-maintenance

# Supprimer les taints (noter le '-' à la fin)
kubectl taint nodes w0 environment=production:NoSchedule-
# Output: node/w0 untainted

kubectl taint nodes w1 maintenance=true:NoExecute-
# Output: node/w1 untainted

# Vérifier que les taints sont bien supprimés
kubectl describe nodes | grep -i taint

# Output attendu
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>
```

> [!TIP]
> ** Nodes nettoyés !** Seul le taint du control plane reste (c'est normal).

---

### 💡 ASTUCE : Toleration spéciale pour TOUT tolérer

```yaml
tolerations:
- operator: Exists  # Tolère TOUS les taints
```

Utilisé par certains DaemonSets (monitoring, logs) qui doivent tourner sur TOUS les nodes.

---


---

## 🎓 Récapitulatif Final

### Quand utiliser quoi ?

| Besoin | Solution |
|--------|----------|
| Placer un pod sur un node spécifique | **nodeSelector** ou **nodeAffinity** |
| Réserver un node pour certains pods | **Taint** sur le node + **Toleration** sur les pods autorisés |
| Isoler le control plane | **Taint** (déjà présent par défaut) |
| Évacuer un node pour maintenance | **Taint NoExecute** temporaire |
| DaemonSet sur tous les nodes | **Toleration** avec `operator: Exists` |

---

### Comparaison Node Affinity vs Taints

| Critère | Node Affinity | Taints & Tolerations |
|---------|---------------|----------------------|
| **Approche** | Pull (pods choisissent) | Push (nodes repoussent) |
| **Placement** | Force le placement | Autorise le placement |
| **Use case** | "Je veux aller sur ce node" | "Ce node n'accepte que certains pods" |
| **Garantie** | Obligatoire avec `required` | Besoin de combiner avec nodeSelector |

---

### Commandes essentielles

```bash
# Labels
kubectl get nodes --show-labels
kubectl label node <NODE> <KEY>=<VALUE>
kubectl label node <NODE> <KEY>-  # Supprimer un label

# Taints
kubectl describe node <NODE> | grep -i taint
kubectl taint nodes <NODE> <KEY>=<VALUE>:<EFFECT>
kubectl taint nodes <NODE> <KEY>=<VALUE>:<EFFECT>-  # Supprimer un taint

# Debug
kubectl get pod <POD> -o wide
kubectl describe pod <POD>
kubectl explain pod.spec.affinity.nodeAffinity
```

---

## 📚 Ressources supplémentaires

- [Documentation officielle Kubernetes - Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Documentation officielle Kubernetes - Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [CKA Exam Tips](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

## 🤝 Contribution

Tu as trouvé une erreur ou tu veux améliorer cette formation ? N'hésite pas à ouvrir une issue ou une pull request !

---

## 📄 Licence

Ce contenu est disponible sous licence [MIT](LICENSE).

---
