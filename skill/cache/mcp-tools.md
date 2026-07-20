# Définition de tools (API Anthropic + MCP) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Namespacer par service/ressource : asana_projects_search, asana_users_search.
- Consolider en un tool par workflow plutôt que des wrappers d'API atomiques : schedule_event au lieu de list_users + list_events + create_event ; get_customer_context au lieu de 3 appels.
- Paramètres non ambigus : user_id plutôt que user.
- Décrire chaque tool « comme à un nouveau collègue » : contexte implicite, formats spéciaux, relations entre ressources.
- Offrir un paramètre enum response_format (concise/detailed) : mesuré 72 vs 206 tokens, environ 2/3 de contexte économisé.
- Pagination/troncature avec défauts sensés ; erreurs actionnables qui proposent des améliorations concrètes, pas des stack traces.
- Évaluer les tools sur des tâches réalistes multi-appels (accuracy, nombre d'appels, tokens).
- MCP : inputSchema JSON Schema valide obligatoire (MUST) ; outputSchema pour valider le structuredContent.
- MCP : erreurs d'exécution en isError:true avec message auto-corrigeable (ex. « Invalid departure date: must be in the future »), à distinguer des erreurs protocole JSON-RPC.
- MCP : human-in-the-loop — afficher les inputs et confirmer les opérations sensibles (SHOULD de la spec).

## Paramètres
- response_format : enum concise | detailed.
- Claude Code tronque les réponses de tools à 25 000 tokens par défaut.
- Noms de tools MCP (recommandations SHOULD, pas des MUST) : 1-128 caractères, uniques par serveur, sans espaces ; caractères recommandés A-Z, a-z, 0-9, underscore, tiret, point.
- Spec MCP de référence : 2025-11-25.

## Pièges
- Wrappers 1:1 d'endpoints API existants sans valeur agentique.
- Serveurs MCP fourre-tout (kitchen-sink) sans périmètre clair.
- Retourner des UUID techniques au lieu de champs lisibles par le modèle.
- Faire confiance aux annotations (readOnly/destructive) de serveurs non vérifiés.
