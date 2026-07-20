# Bolt (StackBlitz) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Traiter le premier prompt comme un blueprint : déclarer d'emblée l'architecture et la stack (ex. Astro, Tailwind, ShadCN) pour un scaffolding correct dès le départ.
- Ajouter ensuite les composants et features un par un, avec des petits prompts spécifiques.
- Utiliser le bouton « Enhance prompt » (icône + du chatbox) : questionnaire guidé qui regénère un prompt recommandé éditable.
- En dépannage : « Make one change. Check that the change works. Move on to the next. »
- Scoper les modifications : clic droit sur un fichier en vue Code → « Target file » pour limiter les changements à ce fichier.
- Verrouiller les fichiers finis : « Lock file » / « Lock all » pour les exclure de toute modification ultérieure.
- Cibler une zone précise : sélectionner du code puis « Ask Bolt ».
- Persister le contexte dans « Knowledge » (niveau compte, projet ou équipe) : instructions inter-sessions.
- Batcher les petites instructions en un seul message ; purger régulièrement le contexte (Clear context) pour la gestion du contexte sur les longues sessions.

## Paramètres
- Enhance prompt : icône + dans le chatbox → questionnaire guidé
- Target file / Lock file / Lock all : clic droit en vue Code
- Knowledge : instructions persistantes compte / projet / équipe
- Batching d'instructions simples en un seul message

## Pièges
- Laisser Bolt choisir la stack : si un framework précis est voulu, le nommer dans le premier prompt — le scaffolding initial est difficile à changer après.
- Ne pas verrouiller les fichiers finis : Bolt peut les modifier lors de prompts ultérieurs.
- Contexte jamais purgé : dérive du modèle sur les longues sessions (le lien direct avec l'économie de crédits n'est pas formulé dans la doc officielle [À VÉRIFIER]).
