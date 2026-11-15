# Apprentissages

## Définir un SLO et alerter dessus
Alerter sur la vitesse de consommation du burn rate plutôt que sur chaque pic d'erreurs, avec un runbook par alerte.

### Problème rencontré et pourquoi il est survenu
- runbook_url mal formée (https://github.com/ en double, .git et nom de dépôt par défaut conservés): une URL de dépôt collée à la place du seul nom de compte
- kubectl exec en timed out waiting for the condition: le Deployment Redis était à 0 replica, donc plus de pod à joindre
- Résolution et étape 6 non observées à cause de problèmes avec Redis 

### Solution appliquée et pourquoi cette solution fonctionne
- URL corrigée avec sed, vérifiée avec grep runbook_url.
- Vérification de l'état réel (kubectl get deploy -n data) plutôt que de réessayer exec.

### Ce que j'ai appris
- 14,4 = burn rate qui consomme 2 % du budget mensuel en 1h; taux d'erreur déclencheur = burn rate x (1 - SLO).
- On alerte sur le symptôme: une cause (Redis down) n'est pas forcément un impact utilisateur, et un symptôme est détecté même pour une cause inconnue
- Les alertes basées sur les métriques internes ont un angle mort : ce qui est refusé avant d'atteindre l'application (Ingress, pods non prêts) n'est pas compté
- Un resolved sans trafic n'a pas de valeur: le test de charge doit tourner jusqu'à la résolution.
- Un rechargement de la config Alertmanager ne redémarre pas le pod 