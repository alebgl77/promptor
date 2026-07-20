# Midjourney — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Cibler V8.1 (défaut depuis le 11/06/2026) ; basculer --v 7 uniquement si --oref/--cref sont nécessaires.
- Écrire des phrases complètes décrivant la scène, jamais des listes de mots-clés : V8 exige un nouveau style de prompting et récompense les prompts longs et spécifiques.
- Hiérarchie de prompt : [SUJET] [DÉTAILS], [CONTEXTE], [STYLE], [TECHNIQUE] --paramètres.
- Employer le vocabulaire d'éclairage (« Rembrandt lighting », « golden hour ») et de photographie (85mm f/1.8, medium format).
- Mettre le texte à faire apparaître dans l'image entre "guillemets" ; ajouter --raw pour une interprétation plus littérale.
- Transfert de style : --sref URL (ou --sref random) ; doser avec --sw ; anciens codes V6 → ajouter --sv 4.
- Cohérence personnage/objet : --oref URL + --ow 0-1000, en --v 7 uniquement (non supporté en V8.1).
- Workflow économique : explorer en --draft (24 images, ~50 % du coût GPU), finaliser en --hd (2048px natif) ; --sd = définition standard moins chère.
- Calibrer --stylize : 0-100 photoréalisme fidèle, 100-300 équilibré, 300-1000 fortement esthétisé (pousser à 1000 est encouragé par l'annonce V8).
- Combos relevés : photoréalisme « --raw --s 50-100 » ; artistique « --s 300-500 --chaos 50 » ; expérimental « --weird 500-1000 --chaos 75 --s 600 ». [À VÉRIFIER]

## Paramètres
- --v 8.1 (défaut) | --v 7 (pour --oref/--cref)
- --ar : 1:1, 4:5, 5:4, 16:9, 21:9, 6:11, 9:16, 3:2, 2:3 (4:5/5:4/21:9/6:11 ajoutés en V8.1)
- --sref URL | random ; --sw 0-1000, défaut 100 [À VÉRIFIER] ; --sv 4/6/7 (défaut 7 [À VÉRIFIER])
- --oref URL + --ow 0-1000 (V7 uniquement)
- --s 0-1000 (défaut 100) ; --raw ; --exp 0-100 (zone utile ~10-25 [À VÉRIFIER])
- --chaos 0-100 (défaut 0) ; --weird 0-3000 (défaut 0) ; --iw 0-2 (défaut 1)
- --hd / --sd ; --draft ; --q 4 (V8 ; retiré en V8.1 [À VÉRIFIER])
- --p / --profile [ID] : ~40 images notées minimum, ~200 pour un profil stable [À VÉRIFIER — chiffres tiers]
- --no [éléments] pour les exclusions (pas de champ negative prompt dédié)

## Pièges
- Prompts V6-style (tags + « 8k, masterpiece, ultra detailed ») en V8 : résultats dégradés.
- --oref/--cref en V8.1 : non supportés — repasser en --v 7.
- --q : disponible en V8 sous forme --q 4, retiré en V8.1 seulement — ne pas affirmer « jamais porté en V8 ».
- Anciens codes --sref V6 sans --sv 4 : le style rendu ne correspond pas.
- docs.midjourney.com bloque les fetchs automatisés (403) : sourcer via updates.midjourney.com.
