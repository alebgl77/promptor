# GPT-image (OpenAI / images ChatGPT) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Modèle API le plus récent : gpt-image-2 (gpt-image-1.5, gpt-image-1, gpt-image-1-mini maintenus).
- Structurer dans un ordre constant : scène/contexte → sujet → détails clés → contraintes.
- Annoncer l'usage visé (pub, mockup UI, infographie) pour fixer le « mode » et le niveau de polish.
- Demandes complexes : segments courts étiquetés ou sauts de ligne plutôt qu'un paragraphe unique ; JSON, tags ou prose fonctionnent tous si l'intention est claire.
- Texte dans l'image : littéral entre guillemets ou en MAJUSCULES ; spécifier typographie/taille/couleur/placement ; exiger « verbatim, no extra characters » ; épeler lettre par lettre les mots difficiles.
- Cas critiques (pubs, traductions d'images) : demander la préservation verbatim — changer uniquement le texte, sans reflow du layout.
- Édition itérative : « change only X, keep everything else the same » ; petits changements par tour ; lister les invariants (identité, géométrie, marque) et les RE-spécifier à chaque itération.
- Compositing multi-images : référencer chaque entrée par index (« Image 1: photo produit… Image 2: référence style… ») et décrire leurs interactions.
- Image API pour le one-shot ; Responses API pour l'édition conversationnelle multi-tours (File IDs, outil image_generation intégré).
- Édition avec préservation d'identité : input_fidelity=high (gpt-image-1.5) ; gpt-image-2 traite toujours en haute fidélité ; masques via canal alpha.

## Paramètres
- quality : low | medium | high | auto (défaut auto ; high pour texte critique et infographies denses, low pour la latence)
- size : libre jusqu'à 3840px par côté, multiples de 16, ratio max 3:1 (ex. 1024x1024, 1536x1024, 2048x2048)
- output_format : png (défaut) | jpeg | webp ; output_compression 0-100
- moderation : auto | low ; streaming : partial_images 0-3 ; n=X pour X variantes
- input_fidelity : high (édition gpt-image-1.5)

## Pièges
- Surcharger un seul prompt d'édition au lieu d'itérer : dérive et régressions sur les zones non ciblées.
- Oublier de re-spécifier les invariants à chaque tour : dérive sur l'identité et le layout.
- Texte voulu sans guillemets : fautes d'orthographe et caractères parasites.
- Image API pour un flux conversationnel multi-tours : utiliser la Responses API.
- Le guide consulté couvre gpt-image-1.5 ; principes applicables à gpt-image-2 selon OpenAI [À VÉRIFIER — cookbook dédié gpt-image-2 non consulté].
