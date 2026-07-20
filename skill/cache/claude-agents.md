# Claude agents long-horizon (Fable 5, Opus 4.8) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Anti-fabrication (snippet officiel) : « Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. »
- Vérification à intervalle : « Establish a method for checking your own work at an interval of [X]… verifying your work with subagents against the specification » — les vérificateurs sous-agents à contexte frais surpassent l'auto-critique.
- Encadrer la délégation : « Use subagents when tasks can run in parallel, require isolated context… work directly rather than delegating » — Opus 4.6 sur-délègue ; Opus 4.7/4.8 délèguent moins par défaut (à encourager) ; Fable 5 délègue de façon fiable.
- Multi-fenêtres : premier prompt dédié au setup (écrire les tests, créer init.sh), puis itérer sur une todo-list dans les fenêtres suivantes.
- État sur disque : tests.json (structuré) + progress.txt (notes libres) + git.
- Au reset de contexte : redémarrage à froid avec « Review progress.txt, tests.json, and the git logs » plutôt que compaction — les modèles récents redécouvrent très bien l'état depuis le filesystem.
- Verrouiller les tests : « It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality. »
- Autonomie : « You are operating autonomously… End your turn only when the task is complete or you are blocked on input only the user can provide. »
- Chaining explicite (appels API séparés) uniquement pour inspecter, logger, évaluer ou brancher sur des sorties intermédiaires ; pattern : draft → review → refine.
- Mémoire persistante : un fichier de leçon par sujet, résumé d'une ligne en tête, mettre à jour plutôt que dupliquer.

## Paramètres
- `send_to_user` tool (client-side, input_schema avec un champ message) pour délivrer du contenu verbatim en cours de run sans terminer le tour — exige une instruction d'élicitation dans le system prompt.
- Sous-agents : exiger des résumés condensés (1 000–2 000 tokens), jamais de logs bruts.

## Pièges
- Compteur de tokens restants visible → arrêts prématurés et handoffs sur Fable 5 : masquer, ou ajouter « You have ample context remaining. Do not stop, summarize, or suggest a new session… ».
- Skills/prompts hérités des générations antérieures : trop prescriptifs pour Fable 5, dégradent la qualité — auditer et élaguer à la migration.
- Actions destructives (git push --force, rm -rf, visibles par des tiers) : snippet de confirmation préalable documenté pour Opus 4.6 ; pour Fable 5 : « Pause for the user only when the work genuinely requires them: a destructive or irreversible action… ».
- Un `send_to_user` défini sans instruction d'élicitation dans le system prompt n'est quasiment jamais appelé.
