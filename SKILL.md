---
name: performance-optimization
description: Démarche d'optimisation de performance fondée sur la mesure - profiler, identifier le vrai goulot d'étranglement (requêtes N+1, rendu inutile, I/O bloquant, algorithme), corriger, puis re-mesurer pour confirmer le gain. À utiliser quand l'utilisateur dit que quelque chose est lent, consomme trop de mémoire ou de CPU, veut "optimiser", accélérer une requête SQL, une page web, un script, un build ou une API, ou demande des avis sur la scalabilité, même sans mentionner de mesure. Use for slow code, slow queries, high memory or CPU, and any performance tuning request.
---

# Optimisation de performance

Règle d'or : **mesurer avant d'optimiser**. L'intuition sur l'endroit où le temps est perdu est souvent fausse, et une optimisation non mesurée ajoute de la complexité pour un gain nul. Cette démarche empêche de réécrire la mauvaise partie du code.

## Démarche

### 1. Définir l'objectif chiffré
"Rapide" ne se vérifie pas. Obtenir : métrique (latence p50/p95, débit, temps de build, mémoire), valeur actuelle, valeur visée, et contexte (volume de données, charge, matériel). Sans cela, proposer de d'abord établir la mesure de référence.

### 2. Mesurer l'état de référence
Reproduire la lenteur de manière répétable (mêmes données, plusieurs exécutions, ignorer la première exécution à froid). Noter les chiffres : ils serviront à prouver le gain.

### 3. Profiler pour trouver le vrai goulot
Choisir l'outil selon le contexte :
- **Python** : `cProfile`, `py-spy`, `line_profiler`, `tracemalloc` ; **JavaScript/Node** : profileur Chrome DevTools, `--prof`, `clinic` ; **Web** : onglet Performance et Network, Lighthouse, Core Web Vitals ; **Go/Rust/Java** : pprof, flamegraph, JFR
- **SQL** : `EXPLAIN (ANALYZE, BUFFERS)` ; repérer scans séquentiels sur grandes tables, estimations de lignes très fausses, tris sur disque
- **Système** : `top`, `htop`, `iostat`, `vmstat`

Si le code peut être exécuté ici, mesurer réellement avec l'outil bash (`time`, benchmark minimal) plutôt que spéculer.

### 4. Identifier la catégorie du problème
| Symptôme | Suspect habituel |
|---|---|
| Temps proportionnel au carré des données | Algorithme O(n²) : boucles imbriquées, recherches linéaires dans des listes (utiliser dict/set/index) |
| Beaucoup de petites requêtes | **N+1** : charger relations en lot (`JOIN`, `IN`, préchargement/eager loading) |
| Requête SQL lente | Index manquant ou non utilisé, fonction sur colonne indexée, `SELECT *`, `OFFSET` profond |
| Temps passé à attendre | I/O bloquant : parallélisme/asynchrone, mise en lot, cache |
| Interface qui rame | Rendus inutiles (React : mémoïsation, clés stables, état trop haut placé), listes longues sans virtualisation, images non optimisées, JS volumineux |
| Mémoire qui grimpe | Fuite (références conservées, écouteurs non retirés), chargement complet au lieu de flux/itérateurs |
| Démarrage / build lent | Dépendances superflues, absence de cache, étapes séquentielles parallélisables |

### 5. Corriger le goulot dominant, un seul à la fois
Traiter d'abord ce qui représente la plus grande part du temps (loi d'Amdahl : optimiser 2 % du temps total ne peut pas apporter plus de 2 %). Préférer les gains structurels (algorithme, requête, mise en lot, cache) aux micro-optimisations.

### 6. Re-mesurer et comparer
Même protocole qu'à l'étape 2. Rapporter avant/après. Si le gain est faible ou invisible, annuler le changement : la complexité ajoutée n'est pas justifiée. Vérifier que les résultats restent corrects (tests).

### 7. Prévenir la régression
Suggérer un benchmark automatisé, une alerte ou un budget de performance si le sujet est critique.

## Précautions
- Un cache apporte un problème d'invalidation et de cohérence : le mentionner et définir la durée de vie/la clé d'invalidation.
- Ne pas sacrifier la lisibilité ou la sécurité pour un gain non démontré.
- Les résultats de benchmark dépendent de la machine : les présenter comme relatifs.

## Format de réponse
```
## Objectif et mesure de référence
## Diagnostic (où va le temps, preuve à l'appui)
## Correctifs proposés (par impact estimé, avec le code)
## Résultat attendu / mesuré (avant → après)
## Suivi (benchmark, supervision)
```
