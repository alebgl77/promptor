# Agents OpenAI (GPT-5.x) & AGENTS.md — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Bloc de persistance verbatim (GPT-5) : « You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user » ; documenter les hypothèses au lieu de demander confirmation.
- Réduire l'avidité de recherche : early stop criteria (« you can name exact content to change », convergence ~70 % des hits sur une zone) + budget explicite (« absolute maximum of 2 tool calls ») pour les cas à faible latence.
- reasoning_effort : ne l'augmenter qu'après une régression mesurée en éval, jamais préventivement.
- Structurer le prompt en blocs XML : output_verbosity_spec, design_and_scope_constraints, uncertainty_and_ambiguity.
- User updates de 1-2 phrases aux phases majeures, chacune avec au moins un outcome concret.
- Contexte long : au-delà de ~10k tokens, demander un outline interne + re-position des contraintes avant de répondre.
- AGENTS.md : documenter commandes de build/test exactes, style de code, instructions de test, sécurité, conventions de commit/PR ; traiter comme documentation vivante.
- Monorepos : AGENTS.md imbriqués par sous-projet — l'agent lit le plus proche de la zone de travail (le plus spécifique gagne) ; les instructions explicites du chat priment toujours sur le fichier.
- AGENTS.md est un format ouvert (60 000+ projets, Agentic AI Foundation / Linux Foundation), lu par Codex, Jules, Cursor, Factory, entre autres.

## Paramètres
- reasoning_effort par défaut : none pour GPT-5.1 ET GPT-5.2 (GPT-5 avait medium).
- Migration : GPT-4o/4.1 → GPT-5.2 reasoning_effort none ; GPT-5.1 minimal → none.
- Compaction en cours de session : POST /v1/responses/compact (après les phases tool-heavy).

## Pièges
- Instructions contradictoires dans le prompt : gaspillent des tokens de raisonnement à réconcilier.
- Narration des appels d'outils routiniers dans les user updates.
- Sur-verbosité par défaut : fixer une spec de verbosité explicite.
