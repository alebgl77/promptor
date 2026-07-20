# Stable Diffusion 3.5 — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Variantes : SD 3.5 Large (8B, ~1 mégapixel, référence open-weights), Large Turbo, Medium.
- Prompter en langage naturel descriptif : le modèle adhère mieux aux phrases complexes qu'aux listes de tags (triple encodeur : 2x CLIP 77 tokens + T5-XXL).
- Prompts longs : passer max_sequence_length=512 (valeur de l'exemple diffusers ; le contexte T5 documenté à l'entraînement est 77/256 tokens — la pratique fonctionne mais la capacité annoncée est moindre).
- negative_prompt : affinage léger UNIQUEMENT — exclusions ciblées (élément, couleur, style), jamais les litanies « worst quality, bad anatomy » ; SD3.5 gère mal le CFG sur les negatives.
- Structure en 7 blocs ordonnés [À VÉRIFIER — issue du guide Stability retiré, relais tiers] : Style → Sujet/Action → Composition/Cadrage → Lumière/Couleur → Paramètres techniques → Texte (entre "guillemets doubles") → Negative prompt.
- Texte à rendre dans l'image : entre guillemets doubles.

## Paramètres
- num_inference_steps = 28 ; guidance_scale 3.5-4.5 (Large) ; Turbo : CFG ~1 et peu de steps
- max_sequence_length = 512 pour les prompts longs (T5)
- aspect_ratio : 16:9, 1:1 (défaut), 21:9, 2:3, 3:2, 4:5, 5:4, 9:16, 9:21
- prompt et negative_prompt : max 10 000 caractères chacun
- seed : 0-4294967294 (0 = aléatoire)
- mode : text-to-image | image-to-image ; strength 0-1 en i2i (0 = préserve l'entrée, 1 = l'ignore)
- output_format : jpeg | png | webp
- Résolution ~1 mégapixel, dimensions divisibles par 64 ; zone optimale ~960-1152px pour le carré

## Pièges
- Recycler les negative prompts fleuves SD1.5/SDXL : contre-productif.
- CFG élevé type SDXL (7-12) : les exemples officiels tournent à 3.5-4.5.
- Syntaxe de weighting (mot:1.2) : non documentée pour SD3.5, comportement non garanti [À VÉRIFIER].
- Dépasser ~1 MP ou dimensions non divisibles par 64 : artefacts.
- Le guide de prompting officiel Stability est hors ligne : sources = model card HF + doc AWS Bedrock + relais tiers.
