# Lovable — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Structurer les prompts complexes en sections markdown étiquetées : `# Context` / `## Task` / `### Guidelines` / `#### Constraints` (technique « training wheels »).
- Meta prompting : « Rewrite this prompt to be more concise and detailed: '[prompt]' » pour faire améliorer le prompt par l'outil lui-même.
- Reverse meta prompting après un debug : faire résumer les erreurs rencontrées et leur résolution en un prompt détaillé réutilisable la prochaine fois.
- Alimenter la Knowledge Base avec 5 documents : PRD, user flow, tech stack, guidelines frontend, structure backend — contexte persistant interprété à chaque prompt.
- Générer l'UI par « modular parts » : hero, feature grid, testimonial slider, pricing table — jamais une page entière d'un coup.
- Fournir du contenu réel : bannir lorem ipsum et « feature 1 / feature 2 » (Lovable fonctionne mal avec du placeholder).
- Poser tôt des descripteurs esthétiques précis (« minimal », « premium », « cinematic », « playful ») : ils pilotent typo, espacements, ombres, radius et palette.
- Ajouter « Ask me any questions you need in order to fully understand what I want from this feature » — idéalement en Plan mode.
- Déboguer en Chat Mode : faire analyser les erreurs AVANT toute modification de code.
- Après 3 « Try to fix » infructueux : demander un raisonnement chain-of-thought pour identifier la cause racine au lieu de relancer le bouton.

## Paramètres
- Structure : # Context / ## Task / ### Guidelines / #### Constraints
- Knowledge Base : PRD + user flow + tech stack + frontend guidelines + backend structure
- Modes : Default (build), Chat (analyse/debug), Plan (clarification)
- Seuil : 3 échecs « Try to fix » → chain-of-thought

## Pièges
- Prompter une landing page entière d'un coup — comparé par la doc à « verser une recette dans un blender ».
- Contenu placeholder : le modèle répond à la structure et à l'intention, le placeholder les masque.
- Spammer « Try to fix » sans analyse : boucle d'échecs — passer en Chat Mode.
- Framework « C.L.E.A.R. » attribué à Lovable par des tiers : absent des docs officielles — ne pas s'y référer.
