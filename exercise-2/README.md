# Apprentissages

## Déployer la stack Prometheus
Installer kube-prometheus-stack et explorer les dashboards et les targets

### Problème rencontré et pourquoi il est survenu
Grafana en CrashLoopBackOff : limite mémoire de 256Mi trop basse

### Solution appliquée et pourquoi cette solution fonctionne
Limite relevée à 768Mi dans exercise-2/monitoring.yaml, puis helm upgrade -f
Grafana a de la marge, le pod est passé en 3/3 Running

### Ce que j'ai appris
- Lire lastState.terminated.reason avant de relancer à l'aveugle
- serviceMonitorSelectorNilUsesHelmValues et ruleSelectorNilUsesHelmValues à false :
  Prometheus sélectionne tous les ServiceMonitors et PrometheusRules, pas seulement ceux de la release Helm 
- Les 4 targets du control plane sont DOWN sur minikube, d'où leur désactivation.
