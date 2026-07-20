# Llama 4 (Scout, Maverick) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Utiliser le template Llama 4 exact : <|begin_of_text|>, <|header_start|>role<|header_end|>, fin de tour <|eot|> ; rôles supportés : system, user, assistant.
- Images : <|image_start|>…<|patch|>…<|image|>…<|image_end|>, avec <|tile_x_separator|>/<|tile_y_separator|> pour les grandes images.
- Tool calling zero-shot : déclarer les fonctions dans le message system OU user ; exiger une sortie strictement de la forme [func_name(param1=value1, param2=value2)], sans aucun texte autour.
- S'inspirer du system prompt de référence Meta (model card Scout, « You are an expert conversationalist... »), conçu pour un ton conversationnel et anti-moralisateur.
- Vérifier le chat template du serveur d'inférence (vLLM/llama.cpp) contre la model card avant tout déploiement.

## Paramètres
- Contexte : 10M tokens (Scout) / 1M (Maverick) ; seuil de dégradation pratique non documenté [À VÉRIFIER].
- Maximum testé : 5 images en entrée multimodale.
- Format d'appel de fonction zero-shot : [func_name(param1=value1)].
- Aucune recommandation officielle Meta de température pour Llama 4 n'a pu être confirmée [À VÉRIFIER].

## Pièges
- Réutiliser les tokens Llama 3 (<|start_header_id|>, <|eot_id|>) : les tokens ont changé — un mauvais template dégrade silencieusement les réponses.
- Dépasser 5 images en entrée : hors du périmètre testé par Meta, comportement non garanti.
- System prompts très longs : préférer 2-4 phrases courtes et focalisées [À VÉRIFIER].
