# Modèles de raisonnement OpenAI (o-series, modes reasoning GPT-5.x) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Ne jamais demander « think step by step » ni forcer un raisonnement à voix haute : le raisonnement est interne, le CoT explicite est inutile et peut dégrader la performance.
- Ne pas induire de raisonnement supplémentaire avant chaque appel de fonction en agentique.
- Zero-shot d'abord ; n'ajouter des exemples few-shot que si le format de sortie l'exige.
- Donner un objectif clair, des contraintes fortes (budgets, critères mesurables) et un contrat de sortie — sans prescrire les étapes intermédiaires.
- RAG : ne fournir que le contexte le plus pertinent ; le surplus dilue la précision.
- Router « planners vs workhorses » : raisonnement élevé pour planification, ambiguïté, domaines experts (exactitude) ; modèle rapide/effort bas pour l'exécution de tâches bien définies (vitesse/coût). Pattern : le raisonneur planifie, un modèle rapide exécute.
- Function calling : chaque description d'outil dit quand l'invoquer et comment construire les arguments (few-shot dans la description pour les formats complexes type regex).
- Placer les règles critiques en tête du prompt (+6 % de précision mesurée sur l'exemple grep).
- Anti-hallucination d'appel : « Do NOT promise to call a function later. If a function call is required, emit it now. »
- Persister les reasoning items entre les tours via la Responses API : améliore les décisions d'appel d'outils.

## Paramètres
- Rôle developer (remplace system depuis o1-2024-12-17 ; conversion automatique system→developer).
- Responses API : store:true ou include=["reasoning.encrypted_content"] pour persister le raisonnement (Chat Completions ne le préserve pas).
- Dimensionnement function calling : ~100 outils max, ~20 arguments par outil, schémas plats de préférence.
- Retraits officiels : o3-2025-04-16 arrêté le 11 décembre 2026 ; o4-mini-2025-04-16 (y compris variante fine-tunée) retiré le 23 octobre 2026 → remplacement recommandé gpt-5.4-mini.

## Pièges
- « Formatting re-enabled » en première ligne : workaround legacy propre aux o-series (o1-2024-12-17+), absent de la doc courante — sur GPT-5.x, utiliser « Use Markdown only where semantically correct » (rafraîchir tous les 3-5 messages).
- Chat Completions en agent multi-tours : contexte de raisonnement perdu à chaque tour, re-raisonnement coûteux.
- Contexte surchargé (RAG massif, historique d'outils non pertinent) : élaguer et résumer les anciens tool calls.
- Cibler un o-series pour un nouveau projet : gamme en retrait fin 2026, utiliser GPT-5.x + reasoning_effort.
