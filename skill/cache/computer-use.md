# Computer use (Claude) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Forcer la vérification visuelle après chaque étape avec le prompt exact recommandé : « After each step, take a screenshot and carefully evaluate if you have achieved the right outcome. Explicitly show your thinking: "I have evaluated step X..." If not correct, try again. Only when you confirm a step was executed correctly should you move on to the next one. »
- Préférer des tâches simples et bien définies aux workflows longs et ambigus.
- Utiliser les raccourcis clavier pour les dropdowns et scrollbars difficiles à manipuler à la souris.
- Pour les tâches répétables : inclure des screenshots + tool calls d'exécutions réussies dans le prompt.
- Mettre les consignes récurrentes dans le system prompt plutôt que dans chaque message.
- Isoler les credentials éventuels dans des balises XML dédiées (risque d'injection de prompt accru).
- Prompts positionnels quand des éléments se ressemblent : « le bouton Submit bleu en bas à droite ».
- Sessions longues : exiger une vérification end-to-end dès le début de session, pas seulement après implémentation.

## Paramètres
- Beta header : computer-use-2025-11-24 (Claude Sonnet 5, Opus 4.8/4.7/4.6, Sonnet 4.6, Opus 4.5) ; computer-use-2025-01-24 pour les modèles antérieurs (Sonnet 4.5, Haiku 4.5, Opus 4.1...).
- enable_zoom: true pour inspecter les petits textes.
- Budget : screenshots ≈ 1000-1800 tokens d'input chacun ; overhead system prompt du beta : 466-499 tokens.
- Caching : 1 breakpoint cache_control après system prompt + tools, jusqu'à 3 sur les derniers blocs tool_result, avancés à chaque tour.

## Pièges
- Claude suppose le résultat d'une action sans vérifier : le prompt de vérification est obligatoire.
- Prompt injection : les instructions contenues dans pages/images peuvent surpasser les vôtres — isoler données et actions sensibles.
- Manipulation de dropdowns/scrollbars à la souris peu fiable : passer aux raccourcis clavier.
