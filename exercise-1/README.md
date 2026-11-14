# Apprentissages

## Répartir les pods (Topology Spread Constraints)
Répartir les pods du guestbook sur les 2 nœuds pour survivre à la perte d'un nœud.

### Problème rencontré et pourquoi il est survenu
- Après le drain, les 3 pods se sont retrouvés sur le même nœud : le scheduler place chaque pod là où il y a de la place, sans garantie de répartition
- Après l'uncordon, les pods ne sont pas revenus sur le nœud remis en service
- Le cluster a aussi planté et j'ai dû le recréer

### Solution appliquée et pourquoi cette solution fonctionne
- topologySpreadConstraints: le scheduler garde au plus 1 pod d'écart entre nœuds au moment de placer un pod
- Rééquilibrage manuel avec rollout restart, car la contrainte ne s'applique qu'à la planification, pas aux pods déjà en place.

### Ce que j'ai appris
- DoNotSchedule laisse le pod Pending, ScheduleAnyway est une préférence
- Pendant un drain avec DoNotSchedule, le nœud cordonné compte pour 0 pod 
- Redis n'a qu'un replica
