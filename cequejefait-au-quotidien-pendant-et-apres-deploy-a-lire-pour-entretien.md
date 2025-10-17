# DevOps 2-4 ans: Équipe de 3 + Lead Senior

## Structure de l'équipe

**Lead DevOps Senior (6-8 ans)**
- Définit l'architecture
- Prend les décisions stratégiques
- Valide les pull requests
- Mentoring

**DevOps Mid-level (2-4 ans) ← VOUS**
- Exécution autonome
- Design de solutions moyennes
- Support technique principal

**DevOps Junior (0-2 ans)**
- Tâches simples guidées
- Apprentissage

## Vos responsabilités concrètes

### Docker
- Créer/optimiser des Dockerfiles pour applications
- Debugging images : taille, sécurité, performances
- Registry management : nettoyer les vieilles images, tagging
- Tests docker-compose en développement
- Résoudre les problèmes de build/push

### Kubernetes
- Déploiements quotidiens : pods, services, ingress
- Debugging avancé : logs, exec, port-forward, describe
- Networking : services, network policies, ingress
- Troubleshooting : CrashLoopBackOff, resource limits
- Scaling : HPA, réplicas, resource management
- Sécurité : RBAC, secrets, security policies

### Helm
- Créer des Charts simples à complexes
- Adapter Charts existants pour nouvelles apps
- Templating : values.yaml, conditions, loops
- Testing : lint, dry-run, validation
- Versionning des Charts
- Helm hooks : tests, pre/post déploiement

### Jenkins
- Créer des pipelines declaratives et scriptées
- CI/CD complet : build, test, push, deploy
- Kubernetes plugin : agents dynamiques
- Intégrations : SonarQube, Docker, Helm, ArgoCD
- Credentials management : secrets, tokens
- Debugging : logs pipelines, stages échoués

### ArgoCD (GitOps)
- Créer des Applications ArgoCD
- Configurer les sync policies
- Debugging sync issues
- Gestion des dépendances : sync waves
- AppProjects et RBAC ArgoCD

### Prometheus & Grafana
- Créer des dashboards personnalisés
- Configurer les alertes
- PromQL queries : métriques pod, service, node
- Troubleshoot : metriques manquantes, collecte
- ServiceMonitors pour Prometheus

## Distribution du travail avec votre Lead

### Responsabilités du Lead Senior
- Approuver architectures
- Décider technologies/versions
- Valider designs complexes
- Escalation production
- Planning stratégique
- Code reviews stratégiques

### Ce que vous faites SANS demander
- Créer une pipeline Jenkins pour une nouvelle app
- Adapter un Helm Chart existant
- Déboguer un pod en crash
- Créer des dashboards Grafana
- Configurer des alertes Prometheus
- Déployer en dev/staging

### Ce que vous faites AVEC validation Lead
- Decisions architecturales majeures
- Changements de cluster
- Modifications de sécurité RBAC
- Déploiements en production complexes
- Modifications Kubernetes critiques

### Escalade au Lead (obligatoire)
- Incidents critiques production
- Perte de données potentielle
- Changements irréversibles
- Problèmes non résolus en 1h
- Questions de sécurité

## Semaine type : répartition

### Lundi
- Daily standup (15 min)
- Créer une pipeline Jenkins pour la nouvelle app "payment-service" (2h)
- Adapter un Helm Chart existant (1.5h)
- Review code d'un collègue (1h)
- Support développeurs (30 min)

### Mardi
- Déboguer incident : Pod CrashLoopBackOff en staging (2h)
  - Voir les logs
  - Vérifier les resources
  - Identifier la cause
  - Corriger et redéployer
- Optimiser un deployment : resource limits incorrects (1.5h)
- Mettre à jour des Charts : nouvelles versions (1h)
- Monitoring ad-hoc (30 min)

### Mercredi
- Créer des dashboards Grafana : monitoring microservices (2h)
- Configurer des alertes Prometheus : sur-consommation CPU/RAM (1.5h)
- Déployer en staging : version candidate production (1h)
- Pair programming avec junior (1h)

### Jeudi
- Vérifier ArgoCD : synchronisation des apps (30 min)
- Déboguer sync issue : dépendances manquantes (1h)
- Teste de performance : scaling sous charge (1.5h)
- Documenter un runbook : procédure de redémarrage pod (1h)
- Valider la pipeline avec le lead avant prod (30 min)

### Vendredi
- Script d'automatisation : cleanup docker registry (1h)
- Amélioration continue : proposition d'optimisation (1.5h)
- Documentation/knowledge sharing (1h)
- Déploiement production hebdo (1.5h)
- Retrospective équipe (30 min)

## Tâches quotidiennes vs. exceptionnelles

### Quotidiennes (40% du temps)
- Support développeurs
- Debugging pods/services
- Monitoring dashboards
- Vérifier les déploiements
- Répondre aux tickets

### Régulières (30% du temps)
- Créer pipelines Jenkins
- Adapter/optimiser Helm Charts
- Configurer alertes
- Déploiements planifiés

### Occasionnelles (20% du temps)
- Incidents majeurs
- Architecture improvements
- Scripts d'automatisation
- Formation/documentation

### Rares (10% du temps)
- Décisions stratégiques
- Migrations/changements majeurs
- Escalade production

## Ce que vous NE faites PAS
- Décisions d'infrastructure au niveau cluster
- Budget/coûts cloud (lead fait ça)
- Négociations vendors
- On-call niveau 3 (escalation finale)
- Security audits complets
- Disaster recovery planning seul
- Architecture multi-région
- Décisions stratégiques long terme

## Autonomie attendue à 2-4 ans

| Situation | Votre action |
|-----------|--------------|
| Pipeline échoue en dev | Debugger, corriger, relancer |
| Pod crash en staging | Investigation complète, résoudre |
| Question dev sur Kubernetes | Expliquer et aider |
| Nouvelle app à déployer | Design du chart, création pipeline |
| Alert Prometheus sensibilité | Ajuster sans demander |
| Problème production | Essayer rapidement (5-10 min), puis escalade lead |
| Choix architecture simple | Décider et implémenter |
| Choix architecture complexe | Proposer au lead, discussion |

## Progression vers Senior (4+ ans)

Pour passer à Senior dans 1-2 ans:
- Mentorer le junior régulièrement
- Prendre des décisions architecturales plus souvent
- Valider les solutions des autres
- Proposer des améliorations stratégiques
- Être autonome 95% du temps sur production
- Déboguer des problèmes très complexes
- Documenter et partager les best practices
- Être on-call niveau 2 (escalade intermédiaire)

---

# Workflow réel: Kubernetes + Helm automatisé dans Jenkins

## Le malentendu courant

**Croyance fausse:**
"Jenkins déploie automatiquement avec Helm, donc les DevOps n'ont rien à faire sur Kubernetes/Helm"

**Réalité:**
Jenkins automatise le processus, mais vous gérez, debuggez, optimisez Kubernetes et Helm constamment.

## Architecture réelle en entreprise

```
┌─────────────────────────────────────────────────────────────────┐
│                        Git Repository                           │
│  (Code app + Helm Charts + Jenkinsfile)                         │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ (Webhook: push sur main)
                 ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Jenkins Pipeline                           │
│  Stage 1: Build (Maven/Go/Python)                               │
│  Stage 2: Test                                                   │
│  Stage 3: Docker Build & Push                                    │
│  Stage 4: ⭐ HELM DEPLOY (automatisé)                            │
│  Stage 5: Smoke Tests                                            │
│  Stage 6: Notify                                                 │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ (helm upgrade --install ...)
                 ↓
┌─────────────────────────────────────────────────────────────────┐
│              Kubernetes Cluster (Dev/Staging/Prod)              │
│  ├─ Deployments                                                  │
│  ├─ Services                                                     │
│  ├─ Ingress                                                      │
│  ├─ ConfigMaps/Secrets                                           │
│  ├─ HPA (Horizontal Pod Autoscaler)                              │
│  └─ PVC (Persistent Volumes)                                     │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────────────┐
│  DevOps: Monitoring, Debugging, Optimisation, Gestion           │
│  ├─ Grafana/Prometheus (alertes, dashboards)                     │
│  ├─ Logs (ELK, Loki)                                             │
│  ├─ kubectl debugging (logs, exec, describe)                     │
│  └─ Helm Chart maintenance                                       │
└─────────────────────────────────────────────────────────────────┘
```

## Scenario réel: Déploiement d'une app

### Timeline

**Jour 1 - Avant le déploiement**

```
9h00  Dev finit la feature, crée un commit
      → Pousse sur git dans branche "feature/payment-v2"

9h30  Pull Request créée
      → Pipeline Jenkins démarre AUTOMATIQUEMENT
      
10h00 Jenkins a fini les tests
      → Code review par Dev Lead + DevOps Lead
      
10h30 PR approuvée, merge sur "main"
      → Webhook Jenkins déclenché
```

**10h45 - Jenkins pipeline démarre (AUTOMATISÉ)**

```groovy
// Stage 1-3: Build + Test + Docker Build
✅ Maven build (3 min)
✅ Tests unitaires (2 min)
✅ Docker build & push (2 min)
   → Image: registry.company.com/payment-service:abc123

// Stage 4: HELM DEPLOY (C'EST ICI QUE VOUS INTERVENEZ)
helm upgrade --install payment-service ./helm/payment-service \
  --namespace production \
  --values values-prod.yaml \
  --set image.tag=abc123 \
  --wait \
  --timeout 10m

✅ Helm chart installé (2 min)

// Stage 5: Smoke Tests
✅ Vérifier les pods (1 min)

// Résultat final
✅ DEPLOYMENT SUCCESS à 11h02
```

Mais en réalité, Jenkins a juste exécuté des commandes.
**Le vrai travail commence après...**

## Ce que vous FAITES VRAIMENT après le déploiement automatisé

### 1. Pendant le déploiement (Jenkins run en cours)

Vous regardez activement :

```bash
# Terminal 1: Watcher les pods
kubectl get pods -n production -l app=payment-service -w

# Terminal 2: Voir les logs en temps réel
kubectl logs -n production -l app=payment-service -f

# Terminal 3: Avoir Grafana ouvert sur le dashboard
# Regarder CPU, Memory, Request rate pendant le rollout
```

**Pourquoi?** Parce que même si Jenkins déploie, les choses peuvent échouer silencieusement:

- Pod démarre mais crash après 30 secondes
- Pas assez de ressources → OOMKilled
- Database connection échoue → CrashLoopBackOff
- Image n'existe pas → ImagePullBackOff

**Votre rôle: Détecter et corriger en direct**

```bash
# ❌ Problème détecté: Pod crash
kubectl logs -n production payment-service-xyz-123

# Output:
# Error: Cannot connect to database: Connection refused

# 🔍 Investigation:
kubectl describe pod payment-service-xyz-123 -n production

# Output:
# Events:
#   - Normal   Created    2m
#   - Normal   Started    2m
#   - Warning  BackOff    1m    (back-off restarting failed container)

# ✅ Solution: Les variables d'env ne sont pas passées correctement
# Vérifier le Helm Chart
kubectl get deployment payment-service -n production -o yaml | grep env

# Correction: Edit values-prod.yaml ou patch direct
kubectl set env deployment/payment-service \
  DB_HOST=postgres-prod.internal \
  DB_PORT=5432 \
  -n production

# Redémarrer
kubectl rollout restart deployment/payment-service -n production
```

### 2. Après le déploiement (Post-deployment)

**Les heures qui suivent: Surveillance active**

```
11h15 - Helm a déployé, pods en running
       Vous: Regarder les logs pour erreurs
       
11h30 - Vérifier Grafana
       Vous: Request rate normal? Error rate 0%? 
       
11h45 - Tester les endpoints critiques
       You: 
       curl https://api.company.com/payments/health
       curl https://api.company.com/payments/list?limit=10
       
12h00 - Vérifier les alertes Prometheus
       Vous: Aucune alerte déclenchée?
       
12h30 - Tout semble bon
       Vous: Créer un ticket pour documenter
       
14h00 - Re-check les logs (4h après déploiement)
       Vous: Aucune erreur intermittente?
```

### 3. Entre les déploiements (Maintenance quotidienne)

Vous allez faire BEAUCOUP de travail en Kubernetes/Helm

#### A. Debugging et troubleshooting

```bash
# Lundi matin: Incident Slack
# "@devops Service payment est lent en production"

# Étape 1: Vérifier les pods
kubectl get pods -n production | grep payment

# Output:
# payment-service-xyz    1/1    Running    0    3d
# payment-service-abc    1/1    Running    0    3d
# payment-service-def    0/1    CrashLoop  4    5m  ⚠️

# Étape 2: Voir les logs du pod crash
kubectl logs payment-service-def -n production --tail=50

# Output:
# Exception: OutOfMemoryError

# Étape 3: Vérifier les resources allouées
kubectl get deployment payment-service -n production -o jsonpath='{.spec.template.spec.containers[0].resources}'

# Output:
# {"limits":{"memory":"512Mi"},"requests":{"memory":"256Mi"}}

# Problème: Memory limit trop basse!

# Étape 4: Corriger le Helm Chart
vim helm/payment-service/values-prod.yaml

# Changer:
# resources:
#   limits:
#     memory: "512Mi"   → "2Gi"
#   requests:
#     memory: "256Mi"   → "1Gi"

# Étape 5: Redéployer avec les valeurs corrigées
helm upgrade payment-service ./helm/payment-service \
  --namespace production \
  --values values-prod.yaml \
  --wait

# Étape 6: Vérifier
kubectl get pods -n production | grep payment
# Tous en 1/1 Running ✅
```

#### B. Optimisation des Helm Charts

```bash
# Jeudi après-midi: Amélioration continue

# Problème identifié: Helm chart manque de health checks
# Solution: Éditer le Helm Chart

vim helm/payment-service/templates/deployment.yaml

# Ajouter:
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 60
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5

# Tester en dev d'abord
helm upgrade payment-service ./helm/payment-service \
  --namespace dev \
  --values values-dev.yaml \
  --dry-run --debug

# Puis en prod
helm upgrade payment-service ./helm/payment-service \
  --namespace production \
  --values values-prod.yaml
```

#### C. Gérer les ressources et le scaling

```bash
# Lundi: HPA déclenchée, CPU à 85%
# Vérifier si c'est normal ou si besoin d'ajuster

kubectl top pods -n production -l app=payment-service

# Output:
# payment-service-1   450m   800Mi    (82% CPU)
# payment-service-2   460m   750Mi    (84% CPU)
# payment-service-3   440m   720Mi    (79% CPU)
# payment-service-4   470m   760Mi    (85% CPU) ← HPA vient d'ajouter ce pod

# Décision: C'est OK (charge temporaire)
# Mais si ça persiste, augmenter les limits dans Helm

helm get values payment-service -n production | grep cpu

# Vérifier si besoin d'ajuster le CPU
# resources.limits.cpu: 600m → augmenter à 800m?
```

#### D. Configurer les Helm Charts pour chaque environnement

```bash
# Dev: Peu de ressources
values-dev.yaml:
  replicas: 1
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
  autoscaling.enabled: false

# Staging: Configuration test
values-staging.yaml:
  replicas: 2
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
  autoscaling.enabled: false

# Prod: Haute disponibilité
values-prod.yaml:
  replicas: 5
  resources:
    limits:
      cpu: 1000m
      memory: 2Gi
  autoscaling.enabled: true
  minReplicas: 5
  maxReplicas: 20
```

## Répartition réelle du travail

### Jenkins (30% du temps)
- Créer/maintenir les Jenkinsfiles
- Fixer les pipelines cassées
- Ajouter des stages (tests, security scans)
- Debug des erreurs de build
- Les déploiements eux-mêmes: ~5 minutes d'exécution automatique

### Kubernetes (40% du temps)
- Debugging pods en crash
- Vérifier les logs en production
- Troubleshooting connectivity
- Watcher les déploiements en temps réel
- Vérifier les resources
- Gérer les persistent volumes
- Troubleshooting networking

### Helm (20% du temps)
- Créer/adapter des Charts
- Gérer les values par environnement
- Tester les Charts (helm lint, dry-run)
- Optimiser les configurations
- Versioning des Charts

### Monitoring (10% du temps)
- Créer des dashboards
- Configurer des alertes
- Vérifier les métriques pendant les déploiements

## Exemple: Une journée complète avec Jenkins automatisé

```
9h00  Daily standup
      Vous: "Hier j'ai déployé payment-service v2.1.0
            Aujourd'hui je dois debugger le pod redis qui crash en prod"

9h15  Incident Slack
      "@devops Le dashboard n'affiche plus les commandes"
      
      ➜ Vous investigez:
      kubectl get pods -n production | grep dashboard
      kubectl logs dashboard-xyz -n production --tail=100
      kubectl describe pod dashboard-xyz -n production
      
      ➜ Problème: Services n'est pas accessible
      kubectl get svc dashboard -n production
      kubectl get endpoints dashboard -n production
      
      ➜ Solution: Redémarrer le service
      kubectl rollout restart deployment/dashboard -n production
      
      ➜ Vérifier:
      curl -I http://dashboard-service:8080

10h00 Jenkins pipeline auto-déploie la nouvelle version de user-service
      
      ➜ Vous regardez en direct:
      kubectl get pods -n production -l app=user-service -w
      kubectl logs -f -n production -l app=user-service
      
      ➜ Problème détecté: Pod crash après 20 secondes
      kubectl logs -n production user-service-abc
      
      Output: "Error: Config map 'user-service-config' not found"
      
      ➜ C'est vous qui allez corriger:
      kubectl get cm -n production | grep user-service
      # ConfigMap manquante, elle doit être créée par le Helm chart
      
      ➜ Vérifier le Helm chart:
      cat helm/user-service/templates/configmap.yaml
      
      ➜ Problème: ConfigMap n'est pas créée correctement
      helm get manifest user-service -n production
      
      ➜ Fix: Éditer le Helm chart ou créer le ConfigMap manuellement
      kubectl create configmap user-service-config \
        --from-literal=DB_HOST=postgres \
        --namespace production
      
      ➜ Redéployer:
      helm upgrade user-service ./helm/user-service \
        --namespace production \
        --wait

11h30 Déboguer le problème redis qui crash (de ce matin)
      
      ➜ kubectl logs -n production redis-master
      Output: "Connection timeout: Too many connections (1000 max)"
      
      ➜ Problem: Limite de connexions atteinte
      ➜ Solution: Augmenter les limites dans Helm
      
      vim helm/user-service/values-prod.yaml
      # redis.maxclients: 1000 → 2000
      
      helm upgrade user-service ./helm/user-service \
        --namespace production \
        --values values-prod.yaml

12h00 Lunch

13h00 Code review d'un collègue
      ➜ Review le Helm chart qu'il a créé
      ➜ Commentaires: "Resource limits manquent, HPA pas configurée"

14h00 Créer les dashboards Grafana pour les nouvelles métriques
      ➜ PromQL query: rate(http_requests_total[5m])
      ➜ Ajouter une alerte si error_rate > 5%

15h00 Tester un nouveau Helm chart en dev
      
      helm install test-app ./helm/test-app \
        --namespace dev \
        --values values-dev.yaml \
        --dry-run --debug
      
      ➜ Corriger les erreurs
      ➜ Installer pour de vrai
      ➜ Tester en manual

16h00 Documenter le runbook pour le problème redis
      ➜ Créer sur Confluence
      ➜ "Que faire si redis-master crash"

17h00 Vérifier que les déploiements de la journée se portent bien
      ➜ Regarder les logs one more time
      ➜ Vérifier Grafana
```

## Le point clé

**Jenkins automatise le processus de déploiement (la commande elle-même)**

Mais vous gérez:

- ✅ **Avant:** Créer les Charts, écrire les Jenkinsfiles
- ✅ **Pendant:** Watcher les logs, détecter les crashs
- ✅ **Après:** Debugger, corriger, optimiser
- ✅ **Entre:** Maintenance, amélioration, troubleshooting

**C'est pour ça que vous êtes payé** - pas pour lancer "helm install", mais pour gérer tout ce qui peut mal tourner.

## Donc en résumé

| Activité | Automatisée? | Votre rôle |
|----------|--------------|------------|
| Push code → Git | ✅ Auto | - |
| Jenkins déclenché | ✅ Auto | - |
| Build/Test/Docker | ✅ Auto | Créer Jenkinsfile |
| Helm deploy | ✅ Auto | Watcher, corriger si échoue |
| Pods démarre | ✅ Auto | Vérifier les logs |
| Service accessible | ❌ Auto | Vous debuggez |
| Monitoring actif | ❌ Auto | Vous regardez Grafana |
| Alerts config | ❌ Auto | Vous créez les règles |
| Logs analysés | ❌ Auto | Vous enquêtez |
| Performance OK | ❌ Auto | Vous optimisez Helm Chart |