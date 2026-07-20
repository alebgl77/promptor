# GPT-5.x (API OpenAI) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Utiliser la Responses API (jamais Chat Completions pour un agent) avec previous_response_id : le modèle réutilise ses reasoning items entre les tours (Tau-Bench Retail 73,9 % → 78,2 %).
- Fixer reasoning_effort explicitement à chaque migration : GPT-4o/4.1 → none ; GPT-5 « minimal » → none ; GPT-5.1 → même valeur. Ne monter le niveau qu'après évaluation, jamais par précaution.
- Contrôler la longueur avec verbosity (paramètre API) globalement + overrides en langage naturel pour des contextes précis (ex. verbosity haute uniquement pour les outils d'édition de code).
- Structurer le developer message : Identity → Instructions → Examples → Context, délimiteurs Markdown/XML ; contenu statique en tête, données variables en fin (maximise le prompt caching).
- Encapsuler chaque comportement dans un bloc XML nommé : <persistence>, <context_gathering>, <tool_preambles>, <self_reflection> (GPT-5) ; <solution_persistence>, <final_answer_formatting>, <user_updates_spec> (5.1) ; <design_and_scope_constraints>, <uncertainty_and_ambiguity>, <long_context_handling>, <extraction_spec>, <high_risk_self_check> (5.2).
- Réduire l'eagerness agentique : budget explicite (« an absolute maximum of 2 tool calls »), critère d'arrêt, clause d'échappement « even if it might not be fully correct ». L'augmenter : « You are an agent — keep going until the user's query is completely resolved », interdire les questions de clarification, documenter les hypothèses après action.
- Tâches agentiques longues : update de 1-2 phrases au moins toutes les 6 étapes d'exécution ou 8 tool calls, chacune avec un résultat concret ; plan rapide avant le premier tool call ; pas de narration des appels routiniers.
- Prompts contradictoires = perte de performance : metaprompting en 2 temps — (1) faire diagnostiquer au modèle les sections fautives, (2) demander des patchs ciblés, pas de réécriture complète.
- À la migration de modèle : repartir du plus petit prompt qui préserve le contrat produit, puis réajuster reasoning effort, verbosity, descriptions d'outils et format de sortie sur des exemples représentatifs.
- Borner le scope : « Implement EXACTLY and ONLY what the user requests » ; cibles de verbosité chiffrées : 3-6 phrases ou ≤5 bullets en standard, ≤2 phrases pour une question oui/non.
- Induire le Markdown explicitement (« Use Markdown only where semantically correct ») et rafraîchir l'instruction tous les 3-5 messages utilisateur.

## Paramètres
- reasoning_effort : none | minimal | low | medium | high | xhigh | max (disponibilité selon modèle ; GPT-5.6 : none/low/medium/high/xhigh/max). Défauts : GPT-5 = medium, GPT-5.1/5.2 = none, GPT-5.5/5.6 = medium — toujours fixer explicitement.
- verbosity : low | medium | high.
- GPT-5.6 (juillet 2026) : tiers Sol (gpt-5.6-sol, frontier, 5 $/30 $ par M tokens), Terra (2,50 $/15 $), Luna (1 $/6 $) ; contexte 1 050 000 tokens ; knowledge cutoff 16 février 2026.
- Prompt caching : préfixe ≥1024 tokens ; prompt_cache_key ~15 req/min par clé ; GPT-5.6+ : prompt_cache_options.ttl = "30m" (seule valeur supportée), écritures facturées 1,25× l'input non caché ; modèles antérieurs : prompt_cache_retention (in_memory/24h), écritures gratuites.
- POST /v1/responses/compact (GPT-5.2+) pour compacter les workflows agentiques longs.

## Pièges
- Laisser le reasoning_effort par défaut : les défauts varient selon le modèle → profil latence/coût imprévisible.
- Markdown désactivé par défaut en API : sans instruction explicite, pas de mise en forme.
- API v1/prompts (prompt objects) dépréciée : arrêt le 30 novembre 2026 — versionner les prompts dans le code (CI/CD).
- Narration de chaque tool call et politesses répétées (« got it ») : explicitement déconseillées.
- Aucun guide de prompting dédié GPT-5.6/Sol/Terra/Luna ni pour xhigh/max [À VÉRIFIER] : appliquer les guides 5.1/5.2 en attendant.
