# Claude API — effort, adaptive thinking et prompt caching
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Profondeur de raisonnement : `thinking: {type:"adaptive"}` + `output_config: {effort:"low"|"medium"|"high"|"xhigh"|"max"}` — ne plus JAMAIS utiliser `budget_tokens`.
- `effort: high` = défaut (identique à l'omission) ; effort affecte TOUS les tokens (texte, tool calls, thinking).
- Par modèle : Opus 4.7/4.8 → démarrer à xhigh pour coding/agentic (max_tokens ~64k) ; Fable 5 → high par défaut, xhigh pour le plus sensible, medium/low pour le routinier ; Sonnet 4.6 → medium par défaut ; low → sous-agents et gros volumes.
- Guider le raisonnement par instructions générales (« Think thoroughly ») plutôt qu'un plan pas-à-pas imposé ; ajouter « Before you finish, verify your answer against [test criteria] » ; interleaved : « After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. » [À VÉRIFIER]
- Caching : structurer chaque requête « stable d'abord » — tools → system → messages, contenu variable en toute fin (le cache est un préfixe à correspondance exacte haché dans cet ordre).
- Poser `cache_control: {type:"ephemeral"}` sur le DERNIER bloc identique entre requêtes ; isoler tout contenu par requête (timestamps) APRÈS le breakpoint.
- Le caching automatique suffit pour les conversations multi-tours simples ; pre-warm possible avec `max_tokens: 0`.

## Paramètres
- xhigh : Fable 5, Mythos 5, Opus 4.8/4.7, Sonnet 5. max : idem + Opus 4.6, Sonnet 4.6 — indisponible sur Haiku 4.5, Sonnet 4.5, Opus 4.5.
- Thinking non désactivable sur Fable 5/Mythos 5 ; adaptive par défaut sur Sonnet 5.
- max_tokens ≥ 64k à xhigh/max (plafond dur couvrant thinking + réponse).
- cache_control : max 4 breakpoints ; lookback 20 blocs ; `ttl:"1h"` (écriture 2x) vs 5 min défaut (écriture 1,25x) ; lecture 0,1x du prix input.
- Minimums cacheables : 512 (Fable 5/Mythos 5), 1 024 (Opus 4.8, Sonnet 5, Sonnet 4.6, Sonnet 4.5), 2 048 (Opus 4.7), 4 096 (Opus 4.6, Opus 4.5, Haiku 4.5).
- Suivi : `usage.cache_read_input_tokens` / `usage.cache_creation_input_tokens`.

## Pièges
- `budget_tokens` : erreur 400 sur Opus 4.7+ et Fable 5/Mythos 5 ; déprécié sur Opus 4.6/Sonnet 4.6.
- Overthinking à effort élevé : baisser effort plutôt qu'empiler des instructions ; sinon snippet « choose an approach and commit to it… avoid revisiting decisions ».
- Opus 4.5 avec thinking désactivé : hypersensible au mot « think » — préférer « consider », « evaluate », « reason through ».
- Fable 5 : demander la retranscription du raisonnement interne → refus `reasoning_extraction` ; lire les blocs thinking structurés de l'API.
- Sous le minimum cacheable, le caching est ignoré silencieusement ; `tool_choice`/images invalident le cache des messages ; modifier une définition de tool invalide tout.
