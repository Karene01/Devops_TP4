# Apprentissages

## Instrumenter le guestbook
Exposer la latence pour mesurer le SLO et faire scraper l'application

### Problème rencontré et pourquoi il est survenu
- La v1.0.0 n'expose pas la latence : impossible de mesurer le SLO de 400 ms.
- 'go build' local refusé : le cache Go (~/go/pkg/mod) n'était pas accessible en écriture.
- Le helm upgrade (révision 5) n'a rien changé : values.yaml était resté en tag v1.0.0,
  donc le template était identique et Helm n'a pas recréé les pods.

### Solution appliquée et pourquoi cette solution fonctionne
- Histogramme observé dans le middleware avec le nom de route : cardinalité bornée, bucket 0.4 posé sur le seuil du SLO pour compter exactement les requêtes sous 400 ms
- Compilation dans Docker (build isolé, reproductible) au lieu du go build local.
- Vérification de l'image réelle avec
  'kubectl get deploy guestbook -o jsonpath='{.spec.template.spec.containers[0].image}''
  plutôt que de se fier au "successfully rolled out".
- ServiceMonitor : Service sélectionné par labels, port par son nom ; pris en compte par Prometheus grâce à serviceMonitorSelectorNilUsesHelmValues: false (exo2).

### Ce que j'ai appris
- Counter : ne fait que monter (utilisé avec rate). Gauge : monte et descend (guestbook_redis_up). Histogramme : buckets cumulés, percentiles via histogram_quantile.
- recover() dans le middleware : sans lui un panic serait compté en 200. On force 500 pour que les métriques reflètent ce que vivent les utilisateurs.
- Un label par URL réelle ferait exploser le nombre de séries : on utilise la route.
- Un rollout réussi ne prouve pas que la bonne version tourne : toujours vérifier l'image.