# Context engineering & multi-agents (Anthropic) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Viser le plus petit ensemble de tokens à haut signal : chaque élément du contexte doit mériter sa place.
- System prompt à la bonne altitude : ni hardcoding fragile de cas particuliers, ni directives vagues — assez spécifique pour guider, assez flexible pour rester heuristique.
- Organiser en sections balisées (XML ou titres Markdown) : background_information, instructions, tool guidance, output description.
- Few-shot : exemples divers et canoniques, pas une liste de cas limites.
- Just-in-time retrieval : garder des identifiants légers (chemins, requêtes) et charger dynamiquement plutôt que précharger.
- Long-horizon : combiner compaction (garder décisions architecturales et bugs non résolus, jeter les sorties de tools redondantes ; variante légère : clearing des tool results), notes structurées (NOTES.md réinjecté au besoin) et sous-agents à résumés compacts.
- Délégation à un sous-agent : toujours 4 éléments — objectif précis, format de sortie attendu, outils/sources recommandés, limites explicites de la tâche (frontières négatives : « ne cherche pas X, c'est le travail d'un autre sous-agent »).
- Écrire des règles d'échelle explicites dans le prompt orchestrateur et forcer le parallélisme à 2 niveaux (sous-agents simultanés + tool calls parallèles) : jusqu'à -90 % de temps de recherche.
- Extended thinking pour la planification du lead ; thinking intercalé après les résultats d'outils chez les workers.
- Évaluer le système multi-agent sur l'état final, pas étape par étape (des chemins divergents peuvent être valides).
- Référence chiffrée confirmée : multi-agent (lead Opus 4 + subagents Sonnet 4) surpasse le single-agent Opus 4 de 90,2 % sur l'éval interne Anthropic.

## Paramètres
- Résumés de sous-agents : 1000-2000 tokens.
- Compaction : maximiser d'abord le rappel, puis itérer sur la précision.
- Barème d'échelle : simple = 1 agent / 3-10 tool calls ; comparaison = 2-4 sous-agents / 10-15 calls chacun ; complexe = 10+ sous-agents à responsabilités découpées.
- Parallélisme : 3-5 sous-agents simultanés ; 3+ tool calls parallèles par sous-agent.

## Pièges
- Précharger toutes les données au lieu du just-in-time retrieval.
- Compacter en perdant décisions et bugs ouverts : ne jeter que le redondant.
- Délégation vague (« research X ») : duplication du travail et trous de couverture.
- Sur-investissement : des dizaines de sous-agents pour une requête simple.
- Évaluer étape par étape des chemins pourtant valides : préférer end-state + checkpoints discrets.
