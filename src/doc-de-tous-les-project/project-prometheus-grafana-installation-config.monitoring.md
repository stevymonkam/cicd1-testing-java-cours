# Installation de Helm, Prometheus et Grafana pour le monitoring d'ArgoCD

featur/helm

## Installation de Helm

Avant d'installer Prometheus, il faut d'abord installer Helm :

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## Installation de Prometheus

Ajouter le repository Helm et installer le stack Prometheus :

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack
```

## Forwarding des ports

### Port Grafana

Pour rendre Grafana accessible depuis l'extérieur :

```bash
kubectl port-forward --address 0.0.0.0 svc/prometheus-grafana 8070:80
```

### Port Prometheus

Pour rendre Prometheus accessible depuis l'extérieur :

```bash
kubectl port-forward --address 0.0.0.0 svc/prometheus-kube-prometheus-prometheus 8090:9090
```

## Configuration de Prometheus pour ArgoCD

### Création des ServiceMonitors

Les ServiceMonitors permettent à Prometheus de récupérer automatiquement les métriques exposées par ArgoCD.

#### 1. ServiceMonitor pour argocd-server-metrics

```bash
vi argocd-server-metrics.yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server-metrics
  namespace: argocd
  labels:
    release: prometheus
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server-metrics
  endpoints:
    - port: metrics
```

#### 2. ServiceMonitor pour argocd-metrics

```bash
vi argocd-metrics.yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
  labels:
    release: prometheus
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-metrics
  endpoints:
    - port: metrics
```

#### 3. ServiceMonitor pour argocd-repo-server-metrics

```bash
vi argocd-repo-server-metrics.yaml
```

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-repo-server-metrics
  namespace: argocd
  labels:
    release: prometheus
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-repo-server
  endpoints:
    - port: metrics
```

### Appliquer les ServiceMonitors

```bash
kubectl apply -f argocd-server-metrics.yaml
kubectl apply -f argocd-metrics.yaml
kubectl apply -f argocd-repo-server-metrics.yaml
```

## Configuration de Grafana

### Connexion de Grafana à Prometheus

1. Aller dans **Home → Connections → Data sources → Prometheus**
2. Au niveau de la connexion, configurer :
   - **Prometheus server URL** : `http://prometheus-kube-prometheus-prometheus.default.svc:9090`

### Création du dashboard ArgoCD dans Grafana

1. Aller dans **New Dashboard → Import**
2. Coller le code JSON du dashboard ArgoCD (voir le fichier JSON complet dans dashbord.json dans le dossier doc-de-tous-les-project)

Le dashboard inclut les sections suivantes :
- **Overview** : Vue d'ensemble avec uptime, clusters, applications, repositories
- **Application Status** : Statut de santé et de synchronisation
- **Sync Stats** : Activité de synchronisation et échecs
- **Controller Stats** : Statistiques du contrôleur ArgoCD
- **Controller Telemetry** : Métriques de performance (mémoire, CPU, goroutines)
- **AppSet Controller Telemetry** : Métriques du contrôleur ApplicationSet
- **Cluster Stats** : Statistiques des clusters gérés
- **Repo Server Stats** : Statistiques du serveur de repositories
- **Server Stats** : Statistiques des services gRPC
- **Redis Stats** : Statistiques Redis

## Accès aux interfaces

- **Grafana** : `http://<votre-ip>:8070`
- **Prometheus** : `http://<votre-ip>:8090`

## Notes importantes

- ArgoCD expose nativement les métriques à travers son API
- Les ServiceMonitors détectent automatiquement les services qui exposent des métriques
- Le label `release: prometheus` est nécessaire pour que Prometheus détecte les ServiceMonitors


INSTALLATION DE PROMETHEUS SANS HELM : TU VAS JUSTE DANS LE DOSSIER PROMETHEUS-TRAINING ET TU FAIT LES KUBECTL APPLY TOUT EST DEJA CONFIGURE SI TU VEUX MONITORE ARGOCD TU FAIT JUSTE LES KUBECTL SUR LES YAML DES METRIC QUI SONT DANS GIT TU NE CHANGE RIEN : POUR INSTALLER GRAFANA C EST AVEC HELM VOIR CE QUI S'EST PASSER PRECEDENMENT
