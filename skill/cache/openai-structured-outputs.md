# Structured Outputs OpenAI (sorties JSON via API) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Toujours préférer Structured Outputs (json_schema + strict:true) au JSON mode : seul Structured Outputs garantit l'adhérence au schéma, le JSON mode ne garantit que du JSON valide.
- Nommer les clés du schéma de façon intuitive ; documenter les champs importants via titles/descriptions.
- Tester response.refusal avant de parser : un refus de sécurité peut remplacer la sortie structurée sans respecter le schéma.
- Extraction : fournir le schéma JSON explicite dans le prompt et distinguer champs requis/optionnels.
- Extraction : imposer null pour toute valeur absente (jamais deviner) et exiger un re-scan du document source avant de finaliser la sortie (pattern <extraction_spec> du guide GPT-5.2).
- Pour les nouveaux projets, partir de gpt-5.6 (recommandation de la doc officielle courante).

## Paramètres
- Responses API : text.format = { type: "json_schema", name, schema, strict: true } ; Chat Completions : response_format équivalent.
- Schéma : additionalProperties: false obligatoire ; toutes les clés listées dans required.
- Function calling strict : "strict": true dans le schéma des outils.
- Supporté depuis gpt-4o-2024-08-06 / gpt-4o-mini.

## Pièges
- JSON mode (type: json_object) = JSON syntaxiquement valide seulement, aucune garantie de schéma.
- Première requête avec un nouveau schéma : latence additionnelle de compilation (les requêtes suivantes retrouvent la performance normale).
- Laisser le modèle deviner les valeurs manquantes au lieu d'imposer null : source d'hallucinations en extraction.
- Schémas profondément imbriqués : préférer des schémas plats (~100 outils, ~20 arguments par outil en pratique).
