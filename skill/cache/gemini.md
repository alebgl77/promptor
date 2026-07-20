# Gemini (2.5 / 3.x / 3.5) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Instructions directes et concises ; supprimer l'over-engineering hérité de 2.5 (répétitions, chaînes de persuasion) ; re-tester tout prompt en migrant vers 3.x/3.5.
- Ne PAS toucher temperature/top_p/top_k sur 3.x/3.5 : les retirer des requêtes (température basse = boucles et dégradation, surtout maths/raisonnement).
- System instruction en tout début, en sections balisées : <role>, <constraints>, <output_format>. Ordre du prompt : system instruction → contexte/données → exemples few-shot → tâche.
- Gemini 3 est laconique par défaut : spécifier explicitement verbosité et persona si ton conversationnel/détaillé voulu.
- Toujours inclure des exemples few-shot au format strictement identique (mêmes délimiteurs, même casse) ; commencer petit (trop d'exemples = overfitting) ; amorcer la sortie par un préfixe de complétion.
- Long contexte : question/instruction À LA FIN, après tout le contexte volumineux ; phrase-pont type « Based on the information above... » (pratique annexe Google/communautaire, pas dans la doc long-context).
- Context caching pour tout contexte volumineux réutilisé entre requêtes (optimisation n°1 du coût d'entrée).
- JSON structuré via response_format + JSON Schema (types forts, enums, bornes, $ref) plutôt que « réponds en JSON » ; remplir les description ; toujours valider les VALEURS côté application. JSON mode + outils intégrés : statut Preview seulement.
- Function calling : noms sans espaces, descriptions précises, enums pour valeurs fermées, 10–20 outils actifs maximum.
- Sur 3.5 : chaque FunctionResponse doit porter l'id et le name du FunctionCall ; contenu multimodal DANS les parts de la function response.
- Anti sur-appel d'outils : réduire thinking_level d'abord, puis system instruction « You have a limited action budget of <n> tool calls ».
- Multimodal : texte AVANT l'image (image unique) ; prompt APRÈS la vidéo (vidéo unique) ; questions temporelles en MM:SS ; segmentation → thinking minimal.
- Adapter thinking_level à la complexité : minimal/low (extraction, factuel), medium (défaut), high (maths, code avancé, planification multi-étapes). Défaut 3.5 Flash passé high → medium : re-tester.
- Latence : thinking_level low + instruction « think silently » [À VÉRIFIER — retiré de la doc actuelle].
- Agents : décomposer en sous-objectifs ; avant chaque tool call, énoncer le pourquoi et le résultat attendu ; persister jusqu'à résolution complète ; ne jamais inventer des données manquantes (les demander).

## Paramètres
- temperature=1.0 (défaut, ne pas modifier) ; retirer top_p/top_k des requêtes 3.x/3.5
- thinking_level: minimal | low | medium | high (remplace thinking_budget) ; défauts : medium (3.5 Flash), high (3.1 Pro), minimal (3.1 Flash-Lite)
- thinking_summaries: "auto" | "none" ; stateful : store: true + previous_interaction_id ; stateless : renvoyer les blocs thought tels quels
- response_format: { type: "text", mime_type: "application/json", schema: {...} } (API Interactions 2026 ; generateContent legacy : response_mime_type + response_schema)
- tool_choice: auto | any | none | validated (preview)
- Contexte : 1M tokens entrée / 65k sortie (3.5 Flash) ; images : 3 600 max/requête, 258 tokens ≤384px sinon tuiles 768×768 ; vidéo : 1 FPS défaut, 10 max/requête, 1h standard / 3h basse résolution ; media_resolution_high pour PDF/texte fin
- box_2d = [ymin, xmin, ymax, xmax] normalisé 0–1000 (à re-normaliser vers les dimensions réelles)

## Pièges
- Réutiliser des prompts sur-ingéniérés conçus pour 2.5 : dégrade 3.x et fait doublon avec thinking_level.
- Baisser la température pour « fiabiliser » : boucles et dégradation documentées.
- Multi-needle (~100 extractions en une requête) : précision dégradée — fractionner en plusieurs requêtes.
- Facturation = output_tokens + thought_tokens : réduire thinking_level avant toute autre optimisation.
- Stateless : omettre/altérer les blocs thought casse la continuité du raisonnement entre tours.
- Schémas JSON très volumineux ou profondément imbriqués rejetés (surtout tool_choice=any).
- Migration 2.5 → 3.x : tokens PDF en hausse, tokens vidéo en baisse — re-chiffrer les coûts.
- Gemini 3.5 Pro non sorti (au 16 juillet 2026) : toute guidance spécifique serait spéculative.
