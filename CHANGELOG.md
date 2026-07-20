# Changelog

Toutes les évolutions notables de Promptor sont documentées ici.

## [3.1.0] — 2026-07

**La base de connaissances.** Promptor démarre informé : les bonnes pratiques actuelles des modèles récents sont embarquées, sourcées et contre-vérifiées.

### Ajouté
- `skill/references/best-practices-2026.md` — base de connaissances : 7 familles, 34 outils, ~110 pratiques issues des documentations officielles (273 requêtes de recherche), passées au crible d'une vérification adversariale indépendante (116 requêtes de contre-vérification, fiabilité mesurée ~95-98 %, affirmations réfutées supprimées, ~40 corrections de précision appliquées).
- `skill/cache/` pré-chargé — 27 fiches de calibration par outil (GPT-5.x, Claude, Gemini, Mistral, Midjourney V8.1, Flux, Stable Diffusion, Ideogram, Cursor, Copilot, v0, Lovable, Bolt, Windsurf, Claude Code, MCP, agents…), en-tête daté, TTL 30 jours.
- `docs/BEST-PRACTICES.md` — méthodologie de constitution et politique de fraîcheur.

### Modifié
- Phase 1bis : chaîne de repli explicite `cache/<outil>.md` → base embarquée → WebSearch → table statique ; la base au-delà de 90 jours reste utilisable mais déclenche un rafraîchissement.

## [3.0.0] — 2026-07

**Le système apprenant.** L'évaluation bascule du texte vers les sorties, et la mémoire transforme chaque cycle en capital.

### Ajouté
- **Tournoi sur sorties** (skill, chemin `--deep`) : les 3 variantes du panel sont exécutées en parallèle sur l'entrée d'essai ; juge en aveugle sur les sorties anonymisées, choix utilisateur sur extraits A/B/C. Les sorties gagnantes validées deviennent les exemples few-shot du prompt final.
- **Gate C6 « performance observée »** : aucune convergence sans sortie d'essai conforme. La version chat produit des `[SORTIE D'ESSAI — simulation]` étiquetées à chaque itération.
- **Mémoire persistante à trois étages** : `profile.md` (défauts personnels, promotion opt-in à la 3ᵉ occurrence), `registry.md` (registre unique des livraisons), `playbook.md` (patterns gagnants + vaccins terrain par famille d'outil).
- **Commandes de capital** : `--again` (rejoue le dernier calibrage), `--from <slug>`, `--list`, `--resume`.
- **Clinique `retour`** : diagnostic de la clause fautive avec citation, correctif minimal, preuve avant/après par ré-exécution, vaccination des prompts futurs (injection d'office à la 2ᵉ occurrence du pattern de panne).
- **`--improve` retouche éclair** : diagnostic + diff direct dans le fichier source, un tour, sans cycle.
- **Routage express/deep** : la cérémonie est proportionnelle à l'enjeu (réutilisable/agent → deep ; besoin simple → express).
- **Raccourcis version chat** : `EXPRESS`, `MON PROFIL`, `RAPPEL`, `RETOUR`, `PATTERNS APPRIS` + `[CARNET DE BORD]` de fin de cycle (mémoire portable inter-sessions).
- Un seul point d'interaction par itération (interrogatoire et pilotage fusionnés).

### Modifié
- La règle d'exception aux raccourcis est portée au niveau du préambule de la Règle Cardinale (règles 2, 4 et 5).
- Le plafond de note de l'itération 1 admet une exception documentée (prompt existant déjà excellent, citations à l'appui).
- L'en-tête `[ÉTAT]` est réservé aux Phases 2–3 ; la Phase 4 conserve son `[ÉTAT FINAL]` propre.

## [2.0.0] — 2026-07

**Le protocole structuré.** Refonte complète de la mécanique d'interaction et de notation.

### Ajouté
- **Règle Cardinale** avec préséance explicite en cas de conflit ; plafond dur de 4 itérations ; l'arrêt appartient exclusivement à l'humain (`OK`/`STOP`) ou au plafond.
- **Rubrique ancrée C1–C5** (descripteurs 0/1/2 par critère) remplaçant la note « étoiles » : citation verbatim obligatoire pour tout 2/2, note d'itération 1 bornée, note d'en-tête strictement égale au total du tableau.
- **Protocole de commandes** `GO / OK / STOP / VOIR / ?` avec règles de reconnaissance (un « ok » en milieu de phrase n'est pas la commande OK ; répondre aux questions vaut GO).
- **Journal D/H** : décisions verrouillées vs hypothèses renversables — une itération ne peut plus défaire la précédente.
- **Table de calibration** intégrée pour 5 familles d'outils, avec ligne de repli pour les outils non listés.
- **Garde-fou d'honnêteté épistémique** remplaçant l'interdiction d'halluciner : étiquetage `[À VÉRIFIER]`, hypothèses tracées.
- **Calibrage en 6 questions** à réponse une-ligne (options, défauts entre crochets, `tout par défaut`).
- Cas prévus : prompt existant collé (reverse engineering), demandes multiples, réponses partielles, autre langue.
- Section `[CALIBRAGE APPLIQUÉ]` : la logique de construction reste visible et corrigeable.
- Livraison protégée : bloc de code réservé à la livraison finale, versions intermédiaires en citation « ne pas copier » ; table des variables `{{…}}` ; points non résolus obligatoires.

## [1.0.0]

**Le méta-prompt original.** Processus en 3 étapes (identification, création en 4 parties — Calibrage, Prompt, Auto-Critique, Interrogatoire — puis itération), attente explicite des réponses utilisateur, livraison finale en bloc de code.
