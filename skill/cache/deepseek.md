# DeepSeek (V4 flash/pro, R1) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- API (hors thinking) : régler la température par type de tâche — 0.0 code/maths, 1.0 analyse de données, 1.3 conversation/traduction, 1.5 écriture créative. (La restriction « hors thinking » est déduite : en thinking mode le sampling est ignoré.)
- Piloter le thinking mode explicitement : {"thinking": {"type": "enabled"|"disabled"}} via extra_body ; activé par défaut (reasoning_effort par défaut : high).
- Tool calls en thinking mode : toujours inclure reasoning_content dans l'historique (sinon erreur 400) ; sans tool calls, le reasoning des tours précédents est inutile.
- R1 local : AUCUN system prompt — toutes les instructions dans le message user ; forcer le début de réponse par « <think>\n » (le modèle saute parfois le raisonnement) ; maths : « Please reason step by step, and put your final answer within \boxed{} ».
- Contexte long : instructions en haut du prompt, question en bas.
- Tours simples : désactiver le reasoning ou demander un thinking concis pour réduire le surcoût.

## Paramètres
- temperature (non-thinking) : 0.0 code/maths, 1.0 data, 1.3 conversation/traduction, 1.5 créatif ; défaut 1.0.
- R1 local : temperature 0.5-0.7, recommandé 0.6.
- thinking: {type: enabled|disabled} via extra_body (format OpenAI) ; reasoning_effort: high|max ; défaut high.
- V4 Pro : temperature=1.0 / top_p=1.0 (relayé par Together AI).
- Alias deepseek-chat → v4-flash non-thinking, deepseek-reasoner → v4-flash thinking : retirés le 24/07/2026 à 15:59 UTC.
- Tailles : v4-pro ≈ 1,6T total/49B actifs, v4-flash ≈ 284B/13B actifs, contexte 1M ; interface Anthropic-compatible disponible [À VÉRIFIER].

## Pièges
- Mettre un system prompt sur R1 open-weight : dégrade (contrainte officielle du README).
- Few-shot sur R1 : dégrade souvent — le modèle mime le pattern des exemples au lieu de raisonner (source : README/papier DeepSeek-R1) [À VÉRIFIER].
- Régler temperature/top_p en thinking mode : ignoré silencieusement, aucune erreur levée.
- Omettre reasoning_content dans un enchaînement de tool calls : erreur 400.
- Température trop basse sur les modèles reasoning : effondre la trace de raisonnement et dégrade les réponses.
