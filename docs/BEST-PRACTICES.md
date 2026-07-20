# Base de connaissances — méthodologie

Promptor embarque une base de connaissances des bonnes pratiques de prompting actuelles :
**[`skill/references/best-practices-2026.md`](../skill/references/best-practices-2026.md)** — 7 familles,
34 outils, ~110 pratiques sourcées, plus **27 fiches de calibration par outil** pré-chargées dans
[`skill/cache/`](../skill/cache/).

## Comment elle a été construite

1. **Recherche multi-sources** — un chercheur par famille de modèles (OpenAI, Anthropic Claude,
   Google Gemini, Mistral & open-weight, générateurs d'images, outils de code, agents autonomes),
   273 requêtes web au total, priorité aux documentations officielles des éditeurs. Chaque pratique
   exige une URL réellement consultée ; ce qui n'a pas de source fiable est déclaré « non établi »
   plutôt qu'inventé.
2. **Vérification adversariale indépendante** — un vérificateur par famille, chargé de *réfuter* :
   116 requêtes de contre-vérification, chaque source rouverte (existe-t-elle ? dit-elle vraiment
   cela ?), versions et paramètres exacts recoupés — c'est là que se logent les hallucinations.
   Résultat : fiabilité mesurée de ~95 à 98 % selon la famille ; les affirmations réfutées ou
   obsolètes ont été corrigées ou supprimées, et ~40 corrections de précision appliquées.
3. **Étiquetage épistémique** — dans la base : **✓** confirmé en source primaire, **⚠ à recouper**
   plausible non contre-vérifié, **[À VÉRIFIER]** incertain. Le même contrat d'honnêteté que
   Promptor impose aux prompts qu'il produit.

## Comment Promptor la consomme

Chaîne de repli de la **Phase 1bis** (calibration dynamique) :

```
cache/<outil>.md (< 30 jours)          ← fiches pré-chargées, mises à jour à l'usage
  └─ references/best-practices-2026.md ← base embarquée, datée
       └─ WebSearch (docs officielles) ← si cache absent/périmé ; réécrit le cache
            └─ table de calibration    ← repli statique du contrat
```

- Le cache par outil a un **TTL de 30 jours** (`--refresh` force la mise à jour).
- Si la base embarquée dépasse **90 jours**, elle reste utilisable mais déclenche un
  rafraîchissement WebSearch en arrière-plan de calibrage.
- La version chat (sans outils) bénéficie indirectement de la base : la table de calibration
  du méta-prompt en est la distillation intemporelle, et le bloc `PATTERNS APPRIS` permet
  d'y injecter les pratiques spécifiques d'un outil.

## Politique de fraîcheur

Les modèles bougent vite ; cette base est datée (2026-07) et conçue pour périmer proprement :
chaque fiche porte sa date, les éléments volatils (versions, paramètres, tarifs) sont marqués,
et le mécanisme de cache régénère fiche par fiche sans toucher au reste. Pour une mise à jour
complète, rejouer le processus de recherche + vérification et remplacer la base.
