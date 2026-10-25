# performance-optimization

Impose à Claude la règle d'or de l'optimisation : **mesurer avant d'optimiser**. L'intuition sur l'endroit où le temps est perdu est souvent fausse, et une optimisation non mesurée ajoute de la complexité pour un gain nul.

Skill Claude : un dossier d'instructions que Claude charge automatiquement quand ta demande correspond. Tu n'as rien à invoquer : décris ton problème normalement.

## Quand il se déclenche

- quelque chose est lent, consomme trop de mémoire ou de CPU ;
- tu veux « optimiser », accélérer une requête SQL, une page web, un script, un build ou une API ;
- tu demandes un avis sur la scalabilité, même sans parler de mesure.

Exemples de messages :

- « Cette requête SQL met 8 secondes, comment l'accélérer ? »
- « Ma page React est lente quand la liste dépasse 500 éléments. »
- « Mon script Python consomme trop de mémoire sur de gros fichiers. »

## Ce que le skill fait faire à Claude

1. **Fixer un objectif chiffré** : métrique (latence p50/p95, débit, mémoire, temps de build), valeur actuelle, valeur visée, contexte. Sans cela, Claude propose d'abord d'établir la mesure de référence.
2. **Mesurer l'état de référence**, de façon répétable (mêmes données, plusieurs exécutions, première exécution à froid ignorée).
3. **Profiler pour trouver le vrai goulot** avec l'outil du contexte : `cProfile`, `py-spy`, profileur Chrome ou Node, Lighthouse, `EXPLAIN (ANALYZE, BUFFERS)` pour SQL, `top` ou `iostat` pour le système. Si le code peut être exécuté, Claude mesure réellement au lieu de spéculer.
4. **Identifier la catégorie du problème** : algorithme en O(n²), requêtes N+1, index manquant, I/O bloquant, rendus inutiles, fuite mémoire, build lent.
5. **Corriger un seul goulot à la fois**, en commençant par celui qui pèse le plus (loi d'Amdahl), en préférant les gains structurels aux micro-optimisations.
6. **Re-mesurer et comparer** avec le même protocole. Si le gain est faible ou invisible, le changement est annulé : la complexité ajoutée n'est pas justifiée.
7. **Prévenir la régression** : benchmark automatisé, alerte ou budget de performance si le sujet est critique.

## Format de la réponse

```
## Objectif et mesure de référence
## Diagnostic (où va le temps, preuve à l'appui)
## Correctifs proposés (par impact estimé, avec le code)
## Résultat attendu / mesuré (avant → après)
## Suivi (benchmark, supervision)
```

## Installation

**Claude Code**

```bash
git clone https://github.com/AGBERESahir/performance-optimization.git ~/.claude/skills/performance-optimization
```

Le dossier doit s'appeler `performance-optimization` et contenir `SKILL.md` à sa racine. Relance Claude Code si une session était ouverte.

**Claude.ai**

Zippe le dossier `performance-optimization` (avec `SKILL.md` dedans), puis ajoute l'archive dans **Paramètres > Skills**.

## Vérifier qu'il fonctionne

Envoie l'un des messages d'exemple ci-dessus. La réponse doit suivre la structure décrite plus haut. Si Claude répond de façon libre, vérifie que le skill est activé dans tes paramètres et que `SKILL.md` n'est pas dans un sous-dossier.

## Limites

- Sans données de mesure de ta part (ou sans pouvoir exécuter ton code), les gains annoncés restent **estimés**, et la réponse le dit.
- Les résultats de benchmark dépendent de la machine : ils sont relatifs, pas absolus.
- Un cache apporte un problème d'invalidation : le skill demande de définir sa durée de vie et sa clé d'invalidation.
- Le skill refuse de sacrifier la lisibilité ou la sécurité pour un gain non démontré.

## Voir aussi

- [architecture-data-review](https://github.com/AGBERESahir/architecture-data-review) : index, requêtes critiques et conception de schéma.
- [safe-refactoring-migration](https://github.com/AGBERESahir/safe-refactoring-migration) : pour restructurer le code sans casser le comportement.
- [infra-deploy-diagnosis](https://github.com/AGBERESahir/infra-deploy-diagnosis) : quand la lenteur vient de l'infrastructure.

## Fichiers

| Fichier | Rôle |
|---|---|
| `SKILL.md` | Les instructions : nom, description de déclenchement, démarche et format de sortie. |
| `README.md` | Cette page, pour les humains (Claude ne la lit pas). |

## Licence

Aucune licence n'est encore indiquée dans ce dépôt ; ajoute un fichier `LICENSE` avant de le réutiliser ou de le partager.
