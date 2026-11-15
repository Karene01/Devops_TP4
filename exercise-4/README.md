# Apprentissages

## Construire un dashboard RED
Visualiser Rate, Errors et Duration du guestbook et observer l'effet du HPA sous charge.

### Problème rencontré et pourquoi il est survenu
- Panels vides ou plats : la période "Last 30 minutes" ne contenait plus la charge, et un 'rate' sans trafic ne renvoie rien
- Les courbes '/' (static) et '/lrange' se superposaient, '/lrange' semblait absent
- Le résumé k6 était perdu: le pod du job avait été supprimé avant la lecture des logs.

### Solution appliquée et pourquoi cette solution fonctionne
- Période absolue 16:30-17:10 pour voir la charge
- Filtre 'handler!~"/healthz|/metrics"': les probes et scrapes ne sont pas des utilisateurs
- 'or vector(0)' : un 'sum(rate(...))' sur un ensemble vide est vide, pas 0
- Duration filtré sur '/lrange/{key}', la route du SLO

### Ce que j'ai appris
- RED = ce que vivent les utilisateurs, contrairement à USE (ressources)
- 'histogram_quantile' interpole dans un bucket : 5 ms signifie "sous le bucket 5ms"
- Moyenne et p99 se ressemblent sans queue de latence ; la différence se voit en cas de requêtes lentes
- Le CPU sature avant la mémoire quand la request CPU est basse
- Lire les logs d'un Job avant de le supprimer

## Analyse
La charge démarre vers 16:40 (~200 req/s). Le CPU sature en premier : request de 50m, cible HPA 50 % (seuil 25m par pod). Le HPA réagit immédiatement : 3->6 pods (~16:40), 7 (~16:42), 8 (~16:53, CPU à 53 %), répartis 4/4 sur les nœuds. Un creux de rate vers 16:44 coïncide avec le scale-up. Après la fin du test (17:03), le HPA redescend à 7 puis 3 pods vers 17:07-17:08, après la fenêtre de stabilisation de 5 min. Mémoire : 9-24 Mi sur 128 Mi.