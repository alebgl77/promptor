# Mistral (Large 3, Medium 3.5, Ministral 3, Le Chat) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Structurer chaque prompt en sections hiérarchiques Markdown ou XML : contexte, instructions, « Response Format », exemples.
- Ouvrir par « You are a <role>, your task is to <task> » ; rédiger comme pour quelqu'un sans aucun contexte préalable.
- Few-shot : injecter les exemples comme tours user/assistant artificiels dans l'historique (meilleure fidélité au format de sortie) ; en mono-message, section « Below are sample inputs and expected outputs ».
- Remplacer tout adjectif quantitatif flou (« too long », « interesting ») par un seuil chiffré ; calculer les comptes de mots/caractères côté client et les passer en entrée — ne jamais faire compter le modèle.
- Notation/classification : échelle verbale (faible/moyen/élevé) plutôt que numérique 1-10 ; catégories prédéfinies dans le prompt ; coupler au JSON mode.
- Instructions conditionnelles : arbre de décision explicite (conditions imbriquées) plutôt que règles qui se chevauchent.
- Modèles reasoning (mistral-medium-3-5) : reasoning_effort="high" pour agentique et code ; rejouer les ThinkChunk INTACTS dans l'historique multi-tour ; "none" pour les tours simples.
- Self-hosting des poids ouverts : tokeniser avec mistral-common (ground truth officiel), jamais un template Jinja communautaire ; le system prompt est préfixé au DERNIER message user (tokenizer V2+).
- Le Chat : assigner un rôle explicite, préciser le format cible (tableau, puces), donner plusieurs exemples, activer le mode Think pour les problèmes complexes, itérer (reformuler, ajouter des détails).

## Paramètres
- Mistral Large 3 : temperature < 0.1 en production ; températures plus hautes réservées au créatif.
- reasoning_effort : "high" | "none" (mistral-small-latest, mistral-medium-3-5).
- vLLM (Large 3) : --tokenizer_mode mistral --config_format mistral --load_format mistral --max-model-len 262144 ; tool use : --enable-auto-tool-choice --tool-call-parser mistral.
- Contexte : 256k (Large 3 et Medium 3.5) ; JSON mode disponible pour sortie structurée.
- Aucune valeur de sampling officielle publiée pour Medium 3.5 [À VÉRIFIER].

## Pièges
- Demander au modèle de compter mots/tokens : non fiable — fournir les comptes en entrée.
- Faire générer ce qui peut être fourni en entrée : l'ingestion est plus rapide que la génération.
- Supprimer les ThinkChunk avant de rejouer l'historique reasoning : dégradation significative de la qualité.
- Template Jinja tiers en self-hosting : espaces et placement du system prompt divergent silencieusement de mistral-common.
- Utiliser Magistral : officiellement DÉPRÉCIÉ — remplacé par Mistral Medium 3.5 (dense 128B, absorbe Magistral et Devstral 2).
- Le Chat : attendre un résultat parfait au premier prompt — l'itération fait partie de la méthode.
