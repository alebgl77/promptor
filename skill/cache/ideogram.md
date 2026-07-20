# Ideogram — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Version : Ideogram 4.0, entraîné exclusivement sur des captions JSON structurées ; Magic Prompt convertit le texte libre vers ce format.
- Formule officielle en 8 éléments : [Image summary]. [Main subject details], [Pose or action], [Secondary elements], [Setting & Background], [Lighting & Atmosphere], [Framing & Composition], [Technical enhancers]. Tous les blocs ne sont pas obligatoires.
- Rester sous ~150-160 mots (~200 tokens) : au-delà, la fin du prompt est diluée ou ignorée — l'essentiel en tête.
- Phrases naturelles > listes de tags (depuis la v2.0).
- Texte (point fort d'Ideogram) : texte exact entre guillemets, placé TÔT dans le prompt, court et percutant, sur fond simple à fort contraste.
- Typographie : décrire par attributs (bold, serif, script) — jamais par nom de police.
- Texte multi-lignes : découper en sections avec placement explicite de chacune.
- Correction de coquilles : peindre sur les caractères fautifs dans l'éditeur puis remixer à force d'image élevée (~90/100).
- Layouts de production (affiches, placement exact, palette précise) : prompter directement en JSON — texte naturel pour l'exploration/photoréalisme, JSON pour la répétabilité.

## Paramètres
- JSON : high_level_description (résumé 1-2 phrases, fortement recommandé) + style_description (optionnel) + compositional_deconstruction (OBLIGATOIRE : fond + éléments « obj »/« text »).
- Ordre imposé des sous-champs style_description — photo : aesthetics, lighting, photo, medium, color_palette ; art : aesthetics, lighting, medium, art_style, color_palette.
- Couleurs : 16 hex max pour l'image entière ET jusqu'à 5 par élément individuel.
- Bounding boxes optionnelles : [y_min, x_min, y_max, x_max] normalisées 0-1000, origine en haut à gauche.
- Magic Prompt : LLM intégré qui réécrit/étend le prompt (texte libre → caption JSON).
- Endpoint API, rendering_speed, style_type, magic_prompt_option exacts : non consultés [À VÉRIFIER].

## Pièges
- Dépasser ~200 tokens : la fin du prompt est ignorée ou diluée.
- Texte long ou scène chargée (fond complexe, multi-sujets) : fautes d'orthographe et distorsions du lettrage.
- Alphabets non latins : rendu imprévisible — l'anglais est optimal.
- Nommer une police précise au lieu de décrire ses attributs : non fiable.
- Envoyer du JSON avec Magic Prompt actif en réécriture : risque de réinterprétation — JSON direct pour la production.
