# Base de connaissances — méthodologie

Promptor embarque une base documentaire datée de bonnes pratiques de prompting :
**[`skill/references/best-practices-2026.md`](../skill/references/best-practices-2026.md)** — 7 familles,
34 outils, ~110 pratiques sourcées, plus **27 fiches de calibration par outil** pré-chargées dans
[`skill/cache/`](../skill/cache/).

## Comment elle a été construite

1. **Recherche documentaire par famille** — OpenAI, Anthropic Claude, Google Gemini,
   Mistral & open-weight, générateurs d'images, outils de code et agents. La compilation
   privilégie les documentations des éditeurs et conserve les liens associés aux pratiques.
2. **Relecture critique des sources** — vérifier l'existence de la page, le passage qui
   soutient une affirmation, la version concernée et les paramètres exacts ; corriger ou
   retirer les pratiques contredites ou obsolètes. Il s'agit d'un contrôle documentaire,
   pas d'un test de performance de Promptor.
3. **Étiquetage épistémique** — **✓** signale une vérification documentaire déclarée à la
   date de la fiche ; **⚠ à recouper** un point non confirmé ; **[À VÉRIFIER]** une
   incertitude. Un marquage historique ne remplace pas la vérification de la source courante.

## Ce que cette méthode permet d'affirmer

Le dépôt publie les synthèses, dates et liens, mais pas un protocole reproductible,
un jeu annoté et les résultats bruts permettant de mesurer une fiabilité globale.
**Aucun taux de fiabilité n'est donc revendiqué.** Le nombre de recherches ou de
contrôles documentaires n'est pas une mesure de précision des prompts générés.

Une évaluation de performance demanderait un jeu de tâches représentatif et
réservé à l'évaluation, une baseline, des critères définis à l'avance, plusieurs
exécutions dans l'outil cible, les versions des modèles et les sorties et coûts
observés. Cette correction éditoriale ne constitue pas une nouvelle validation
des 34 outils et ne modifie pas les dates des sources ni des caches.

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
