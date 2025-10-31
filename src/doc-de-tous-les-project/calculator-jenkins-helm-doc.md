# Documentation Complète: Calculator Project
## Architecture Standard: Moyenne Entreprise Europe

---
feature-helm
## Table des matières

1. [Structure générale](#structure-générale)
2. [Infrastructure Kubernetes](#infrastructure-kubernetes)
3. [Flux complet jour par jour](#flux-complet-jour-par-jour)
4. [Structure des branches](#structure-des-branches)
5. [Rôles et responsabilités](#rôles-et-responsabilités)
6. [Fichiers importants](#fichiers-importants)
7. [Configuration ArgoCD](#configuration-argocd)
8. [Commandes pratiques](#commandes-pratiques)

---

## Structure générale

### Repository unique: calculator

```
calculator/
├─ src/
│  ├─ main/
│  │  ├─ java/
│  │  └─ resources/
│  └─ test/
│
├─ helm/
│  └─ calculator/
│     ├─ Chart.yaml
│     ├─ values.yaml (défaut)
│     ├─ values-dev.yaml         ← ArgoCD met à jour (env DEV)
│     ├─ values-staging.yaml     ← ArgoCD met à jour (env STAGING)
│     ├─ values-prod.yaml        ← ArgoCD met à jour (env PROD)
│     └─ templates/
│        ├─ deployment.yaml
│        ├─ service.yaml
│        ├─ ingress.yaml
│        ├─ configmap.yaml
│        └─ hpa.yaml
│
├─ Dockerfile
├─ Jenkinsfile                   ← La pipeline Jenkins
├─ pom.xml
└─ README.md
```

**Important:** Les fichiers `values-*.yaml` sont versionnés dans Git et mis à jour par Jenkins à chaque déploiement.

---

## Infrastructure Kubernetes

### 3 Clusters séparés

```
Cluster DEV (eu-central-1)
├─ Region: EU-Central
├─ Namespace: dev
├─ Nodes: 2 (t3.medium)
└─ ArgoCD Application: calculator-dev
   ├─ syncPolicy: automated
   ├─ Deploys: Automatiquement dès que values-dev.yaml change
   └─ Self-Heal: true (corrige les drifts)

Cluster STAGING (eu-central-1)
├─ Region: EU-Central
├─ Namespace: default
├─ Nodes: 2 (t3.large)
└─ ArgoCD Application: calculator-staging
   ├─ syncPolicy: manual
   ├─ Deploys: Après approbation manuel dans ArgoCD UI
   └─ Self-Heal: false (à cause du manuel)

Cluster PRODUCTION (eu-west-1 + eu-central-1)
├─ Regions: Multi-zone (haute disponibilité)
├─ Namespace: default
├─ Nodes: 5 (t3.xlarge)
├─ Replicas: 5 minimum
└─ ArgoCD Application: calculator-prod
   ├─ syncPolicy: manual
   ├─ Deploys: Après approbation manuel dans ArgoCD UI
   └─ Self-Heal: false (à cause du manuel)
```

### ArgoCD Instance

```
ArgoCD Server: http://109.176.198.187:30080
├─ Hébergé: Cluster DEV (économique)
├─ Surveille: Repository calculator (branche main)
│
└─ Applications configurées:
   ├─ calculator-dev
   ├─ calculator-staging
   └─ calculator-prod
```

---

## Flux complet jour par jour

### Jour 1: Feature branch (développeur)

**Matin 9h00**

Dev crée une branche feature:
```bash
$ git checkout -b feature/payment-gateway
$ # Fait des modifications
$ git add .
$ git commit -m "feat: Add payment gateway"
$ git push origin feature/payment-gateway
```

**Jenkins déclenche automatiquement** (webhook GitHub):

```
Pipeline Exécution:
├─ Stage: Initialize & Determine Action
│  └─ ACTION = 'test-only'
│  └─ IMAGE_TAG = 'feature-abc123'
│
├─ Stage: Checkout ✅
├─ Stage: Build ✅
├─ Stage: Code Linting (Checkstyle) ✅
├─ Stage: Unit Tests ✅
├─ Stage: Integration Tests ✅
├─ Stage: Code Coverage (JaCoCo) ✅
├─ Stage: SonarQube Analysis ✅
├─ Stage: Package ✅
├─ Stage: Non-Regression Tests ✅
│
└─ ❌ Image Prune (SKIPPED - feature/)
   ❌ Docker Build (SKIPPED - feature/)
   ❌ Docker Push (SKIPPED - feature/)
   ❌ Update Helm (SKIPPED - feature/)
   ❌ ArgoCD Sync (SKIPPED - feature/)

Résultat: PR GitHub avec ✅ Checks passed
```

**10h00** - Dev Lead approuve la PR dans GitHub

**Midi** - Dev merge PR:
```bash
$ git merge feature/payment-gateway into develop
$ git push origin develop
```

---

### Jour 2: Branche develop (après merge)

**Après-midi 14h00**

Merge vers develop déclenche Jenkins:

```
Pipeline Exécution:
├─ Stage: Initialize & Determine Action
│  └─ ACTION = 'build-and-deploy-dev'
│  └─ IMAGE_TAG = 'dev-234'
│
├─ Stage: Checkout ✅
├─ Stage: Build ✅
├─ Stage: Code Linting ✅
├─ Stage: Unit Tests ✅
├─ Stage: Integration Tests ✅
├─ Stage: Code Coverage ✅
├─ Stage: SonarQube Analysis ✅
├─ Stage: Package ✅
├─ Stage: Non-Regression Tests ✅
│
├─ Stage: Image Prune ✅
│  └─ docker stop calculator-dev
│  └─ docker rm -f calculator-dev
│
├─ Stage: Docker Build ✅
│  └─ docker build -t calculator-dev:dev-234 .
│
├─ Stage: Security Scan ✅
│  └─ docker scan calculator-dev:dev-234
│
├─ Stage: Docker Push ✅
│  └─ docker push user/calculator-dev:dev-234
│
├─ Stage: Update Helm Values (DEV) ✅
│  └─ sed -i "s|tag:.*|tag: dev-234|" helm/calculator/values-dev.yaml
│
├─ Stage: Commit & Push Helm Changes ✅
│  └─ git push origin develop
│
├─ Stage: Trigger ArgoCD Sync ✅
│  └─ curl -X POST http://argocd.../api/v1/applications/calculator-dev/sync
│
└─ Stage: Wait for ArgoCD Sync ✅
   └─ Status: Synced ✓
```

**14h20** - ArgoCD détecte le changement:

```
ArgoCD Sync (DEV):
├─ Application: calculator-dev
├─ syncPolicy: automated
├─ Détecte: Changement dans helm/calculator/values-dev.yaml
├─ Action: Auto-sync triggered
│
├─ Exécute: helm upgrade --install calculator \
│             helm/calculator \
│             --namespace dev \
│             --values values-dev.yaml \
│             --set image.tag=dev-234
│
├─ Résultat: Pods en DEV recréés avec nouvelle image
└─ Status: Synced ✓ (vert dans ArgoCD UI)
```

**14h30** - Stage: Health Check ✅
```bash
curl http://109.176.198.187:9003/actuator/health
→ {"status":"UP"}
```

**14h35** - Stage: Smoke Tests ✅
```
Test addition: 5 + 3 = 8 ✅
Test soustraction: 10 - 3 = 7 ✅
Test multiplication: 10 * 5 = 50 ✅
Test division: 10 / 5 = 2 ✅
```

**Résultat:** App en DEV après ~35 minutes
- Les devs peuvent tester immédiatement
- Feedback rapide sur le code

---

### Jour 3: Branche main (merge depuis develop)

**Matin 10h00**

Dev Lead merge develop → main:
```bash
$ git checkout main
$ git merge develop
$ git push origin main
```

**Jenkins déclenche automatiquement:**

```
Pipeline Exécution:
├─ Stage: Initialize & Determine Action
│  └─ ACTION = 'build-and-deploy-staging'
│  └─ IMAGE_TAG = '345'
│
├─ [Tous les stages de build/test comme develop]
│
├─ Stage: Update Helm Values (STAGING) ✅
│  └─ sed -i "s|tag:.*|tag: 345|" helm/calculator/values-staging.yaml
│
├─ Stage: Commit & Push Helm Changes ✅
│  └─ git push origin main
│
├─ Stage: Trigger ArgoCD Sync ✅
│  └─ curl -X POST http://argocd.../api/v1/applications/calculator-staging/sync
│
└─ Stage: Wait for ArgoCD Sync
   └─ Status: (non-bloquant, ArgoCD attend approbation)
```

**10h30** - ArgoCD détecte le changement:

```
ArgoCD Status (STAGING):
├─ Application: calculator-staging
├─ syncPolicy: manual
├─ Détecte: Changement dans helm/calculator/values-staging.yaml
├─ Action: Marque l'application comme "OutOfSync" (rouge)
└─ ArgoCD UI montre:
   ├─ Statut: OutOfSync ⚠️
   ├─ Diff visible: Image tag 234 → 345
   └─ Bouton "Sync": Disponible (attend approbation)
```

**10h35** - QA Lead approuve dans ArgoCD UI:

```
Actions du QA Lead:
1. Ouvre ArgoCD UI: http://109.176.198.187:30080
2. Navigue vers: Applications > calculator-staging
3. Click sur "Sync" dans l'interface
4. Review des changements (diff)
5. Confirme: "Sync now"

ArgoCD exécute:
├─ helm upgrade --install calculator \
│   helm/calculator \
│   --namespace default \
│   --values values-staging.yaml \
│   --set image.tag=345
│
└─ Résultat: Pods en STAGING recréés
   Status: Synced ✓
```

**11h00** - Stage: Health Check & Smoke Tests ✅

**Résultat:** App en STAGING
- QA peut commencer les tests fonctionnels
- Tests complets avant production

---

### Jour 4: Promotion vers production (tag)

**Après-midi 16h00**

QA finit les tests en STAGING: ✅ OK

Dev crée un tag:
```bash
$ git tag v1.2.0 main
$ git push origin v1.2.0
```

**Jenkins déclenche automatiquement:**

```
Pipeline Exécution:
├─ Stage: Initialize & Determine Action
│  └─ BRANCH_NAME = 'main'
│  └─ ACTION = 'build-and-deploy-staging'
│  └─ IMAGE_TAG = 'v1.2.0'
│
├─ [Tous les stages de build/test]
│
├─ Stage: Update Helm Values (STAGING) ✅
│  └─ Mise à jour values-staging.yaml (déjà à jour)
│
└─ [Pas de stage prod car pas de modification values-prod.yaml]
```

**Remarque importante:** Le tag n'autorise pas automatiquement le déploiement en PROD. Il faut une action manuelle du DevOps.

**16h30** - DevOps update manuellement pour prod:

```bash
# DevOps clone et met à jour
$ git checkout main
$ sed -i "s|tag:.*|tag: v1.2.0|" helm/calculator/values-prod.yaml
$ git add helm/calculator/values-prod.yaml
$ git commit -m "chore: Deploy v1.2.0 to production"
$ git push origin main
```

**16h35** - Jenkins re-déclenche (changement détecté):

```
Pipeline Exécution:
├─ Jenkins lance une nouvelle build
├─ Détecte: Changement dans values-prod.yaml
├─ Trigger ArgoCD pour calculator-prod
└─ ArgoCD Status: OutOfSync
```

**16h40** - ArgoCD détecte:

```
ArgoCD Status (PRODUCTION):
├─ Application: calculator-prod
├─ syncPolicy: manual
├─ Détecte: Changement dans values-prod.yaml
├─ Status: OutOfSync ⚠️
└─ ArgoCD UI montre: Prêt à déployer v1.2.0
```

**16h45** - DevOps approuve dans ArgoCD UI:

```
Actions du DevOps:
1. Ouvre ArgoCD UI
2. Navigue vers: Applications > calculator-prod
3. Review des changements
4. Click sur "Sync"
5. Confirme: "Sync now" (plus d'une fois si nécessaire)

ArgoCD exécute:
├─ helm upgrade --install calculator \
│   helm/calculator \
│   --namespace default \
│   --values values-prod.yaml \
│   --set image.tag=v1.2.0 \
│   --replicas 5 \
│   --cpu 1000m
│
└─ Résultat: Pods en PRODUCTION déployés progressivement
   Rolling update (préserve la disponibilité)
   Status: Synced ✓
```

**17h00** - Stage: Health Check & Smoke Tests ✅

**Résultat:** App en PRODUCTION ✅
- v1.2.0 accessible en production
- Historique complet dans Git
- Rollback facile si besoin

---

## Structure des branches

```
Git Branches Workflow:

feature/payment-gateway
     ↓ (PR → merge)
develop
     ↓ (merge après tests)
main
     ↓ (tag pour prod)
v1.2.0, v1.2.1, v2.0.0, etc.
```

### Branche: feature/*
- **Créée par:** Développeur
- **Merge vers:** develop (via PR)
- **Pipeline action:** test-only
- **Déploiement:** ❌ Non

### Branche: develop
- **Créée par:** Merge depuis feature/*
- **Utilisée pour:** Intégration continue
- **Pipeline action:** build-and-deploy-dev
- **Déploiement:** ✅ DEV (automatique)

### Branche: main
- **Créée par:** Merge depuis develop
- **Utilisée pour:** Versions stables
- **Pipeline action:** build-and-deploy-staging
- **Déploiement:** ✅ STAGING (manuel)

### Tags: v*.*.*
- **Créés sur:** main
- **Utilisés pour:** Production
- **Pipeline action:** Trigger manuel DevOps
- **Déploiement:** ✅ PRODUCTION (manuel)

---

## Rôles et responsabilités

| Rôle | Ce qu'il fait | Quand |
|------|---------------|-------|
| **Développeur** | Push code sur feature/* | À chaque changement |
| **Jenkins** | Build + Test + Docker | À chaque push (automatique) |
| **Dev Lead** | Code review + merge vers develop | Après PR approuvée |
| **ArgoCD (auto)** | Déploie en DEV | Après merge develop |
| **Développeurs** | Testent en DEV | Après déploiement auto |
| **QA Lead** | Tests en STAGING | Avant approbation |
| **QA Lead** | Approuve sync ArgoCD STAGING | Decision manuelle |
| **DevOps** | Update values-prod.yaml | Avant production |
| **DevOps** | Approuve sync ArgoCD PROD | Decision manuelle |
| **ArgoCD (manual)** | Déploie en STAGING/PROD | Après approbation |

---

## Fichiers importants

### Helm Chart Structure

```
helm/calculator/
│
├─ Chart.yaml
│  name: calculator
│  version: 1.0.0
│  appVersion: "1.2.0"
│
├─ values.yaml (défaut)
│  image:
│    repository: user/calculator
│    tag: latest
│    pullPolicy: IfNotPresent
│  replicaCount: 1
│  resources:
│    requests:
│      cpu: 100m
│      memory: 256Mi
│    limits:
│      cpu: 200m
│      memory: 512Mi
│
├─ values-dev.yaml (Dev)
│  replicaCount: 1
│  resources:
│    requests:
│      cpu: 100m
│      memory: 256Mi
│    limits:
│      cpu: 200m
│      memory: 512Mi
│
├─ values-staging.yaml (Staging)
│  replicaCount: 2
│  resources:
│    requests:
│      cpu: 250m
│      memory: 512Mi
│    limits:
│      cpu: 500m
│      memory: 1Gi
│
├─ values-prod.yaml (Production)
│  replicaCount: 5
│  autoscaling: true
│  resources:
│    requests:
│      cpu: 500m
│      memory: 1Gi
│    limits:
│      cpu: 1000m
│      memory: 2Gi
│
└─ templates/
   ├─ deployment.yaml
   ├─ service.yaml
   ├─ ingress.yaml
   ├─ configmap.yaml
   └─ hpa.yaml (Horizontal Pod Autoscaler)
```

### Jenkinsfile

- Chemin: `/calculator/Jenkinsfile`
- Détermine automatiquement l'action selon la branche
- Met à jour les values.yaml dans Git
- Déclenche ArgoCD

### Values files (versionné dans Git)

- `helm/calculator/values-dev.yaml` ← Mis à jour par Jenkins (develop)
- `helm/calculator/values-staging.yaml` ← Mis à jour par Jenkins (main)
- `helm/calculator/values-prod.yaml` ← Mis à jour manuellement par DevOps

---

## Configuration ArgoCD

### Applications Configurées

#### 1. calculator-dev

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: calculator-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/calculator.git
    targetRevision: main
    path: helm/calculator
    helm:
      valueFiles:
        - values-dev.yaml
  destination:
    server: https://dev-cluster.company.eu
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncInterval: 3m
```

#### 2. calculator-staging

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: calculator-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/calculator.git
    targetRevision: main
    path: helm/calculator
    helm:
      valueFiles:
        - values-staging.yaml
  destination:
    server: https://staging-cluster.company.eu
    namespace: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    # Manual sync - pas de automated
```

#### 3. calculator-prod

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: calculator-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/company/calculator.git
    targetRevision: main
    path: helm/calculator
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://prod-cluster.company.eu
    namespace: default
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    # Manual sync - pas de automated
```

---

## Commandes pratiques

### Git Workflow

```bash
# 1. Créer une feature branch
git checkout -b feature/mon-feature
git add .
git commit -m "feat: Ma feature"
git push origin feature/mon-feature

# 2. Créer une PR (via GitHub UI)
# 3. Merge dans develop
git checkout develop
git merge feature/mon-feature
git push origin develop

# 4. Merge dans main
git checkout main
git merge develop
git push origin main

# 5. Créer un tag pour production
git tag v1.2.0
git push origin v1.2.0

# 6. Update values-prod.yaml (DevOps)
sed -i 's|tag: .*|tag: "v1.2.0"|' helm/calculator/values-prod.yaml
git add helm/calculator/values-prod.yaml
git commit -m "chore: Deploy v1.2.0 to production"
git push origin main
```

### ArgoCD CLI

```bash
# Login to ArgoCD
argocd login 109.176.198.187:30080

# List applications
argocd app list

# Get application status
argocd app get calculator-dev

# Manually sync
argocd app sync calculator-staging

# Get application diff
argocd app diff calculator-staging

# Rollback (if needed)
argocd app rollback calculator-prod <revision>
```

### Jenkins UI

```
Jenkins URL: http://109.176.198.187:8080/
Pipeline: Calculator
Build Triggers: GitHub push (webhook)
Logs: Click on build number > Console Output
Artifacts: build.log, checkstyle.html, jacoco/
```

### Monitoring

```
ArgoCD UI: http://109.176.198.187:30080/
├─ Applications: Calculator-dev, calculator-staging, calculator-prod
├─ Sync Status: Synced / OutOfSync
└─ Resource Tree: Voir tous les pods/services

SonarQube: http://109.176.198.187:9000/
├─ Calculator project
├─ Code quality metrics
└─ Quality gate status

Kubernetes Cluster:
kubectl get deployments -n dev
kubectl get pods -n dev
kubectl logs -n dev deployment/calculator
```

---

## Avantages de cette approche

✅ **1 repo = plus simple**
- Tout dans un seul endroit
- Versioning cohérent

✅ **Helm charts versionnés**
- Historique complet
- Rollback facile

✅ **Jenkins responsabilité claire**
- Build + Test + Docker
- Pas de déploiement direct

✅ **ArgoCD fait les déploiements**
- GitOps pur
- Déclaratif

✅ **Auto-sync en DEV**
- Feedback rapide
- Itération rapide

✅ **Manual sync en STAGING/PROD**
- Contrôle total
- Approvals requises

✅ **Audit trail complet**
- Tout dans Git
- ArgoCD logs

✅ **Rollback facile**
- Juste revert le commit
- Automatique dans Git

