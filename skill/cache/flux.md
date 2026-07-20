# Flux (Black Forest Labs) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Famille : FLUX.1 (dev/schnell/pro), FLUX.1 Kontext (édition in-context), FLUX.2.
- JAMAIS de negative prompt : Flux ne les supporte pas — décrire ce qu'on veut, pas ce qu'on ne veut pas.
- Écrire en prose naturelle et précise ; l'éclairage explicite est le levier n°1 de qualité.
- Prompter en anglais pour la précision maximale (autres langues supportées mais moins fiables).
- Gabarit officiel (point de départ flexible) : [SUBJECT], [LOCATION], [STYLE], [CAMERA SETTINGS], [LIGHTING], [COLORS], [EFFECT], [ADDITIONAL ELEMENTS].
- Commencer simple, puis itérer un détail à la fois plutôt que réécrire tout le prompt.
- Couleurs de marque : code hex #RRGGBB + nom de la couleur, en génération comme en édition.
- Texte dans l'image en 3 temps : texte exact entre guillemets + placement décrit + style typographique par attributs (« gold serif lettering ») ; placer le texte tôt dans le prompt et le garder court.
- Édition (Kontext/FLUX.2) : verbes d'action précis — Change / Replace / Add / Remove / Turn into.
- Toujours ajouter une clause de préservation : « Change nothing else » ou « while keeping the rest of the image unchanged ».
- Exemple canonique d'édition : « Replace the cherries in the right-most jar with multi-colored sprinkles. Change nothing else. »

## Paramètres
- Kontext et FLUX.2 : jusqu'à 10 images d'entrée (multi-référence).
- Hex : #RRGGBB + nom (« text 'ACME' in color #FF5733 » ; « Change the cow's white fur to the color #8bc4bb »).
- Typographie par attributs : serif / sans-serif / script / display.

## Pièges
- Coller un negative prompt SD-style : non supporté — reformuler positivement.
- Formulations d'édition vagues (« make it better », « improve the lighting », « fix the image ») : résultats imprévisibles.
- Éditer sans clause de préservation : le reste de l'image change aussi.
- Textes longs dans l'image : la fidélité des caractères s'effondre — garder court.
- Prompter hors anglais : supporté mais moins précis.
