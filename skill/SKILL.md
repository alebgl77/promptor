---
name: promptor
description: Crée des prompts optimisés sur-mesure pour n'importe quel outil IA (chatbot LLM, générateur d'images, outil de code, agent autonome) via un processus itératif human-in-the-loop qui APPREND de l'usage — profil auto-distillé, registre des livraisons (--again, --from, --list), tournoi de variantes jugées sur leurs SORTIES réelles, gate de convergence sur performance observée, clinique de retour terrain (retour) avec vaccination des prompts futurs. Trigger - /promptor, « écris-moi un prompt pour », « crée un prompt », « améliore ce prompt », « prompt parfait », « méta-prompt », « refais comme la dernière fois »
trigger: /promptor
---

# /promptor — Architecte de prompts sur mesure (système apprenant)

Transforme un besoin (ou un prompt existant) en prompt optimisé pour un outil IA précis. La spécification de comportement (règle cardinale, protocole, table de calibration, garde-fou anti-hallucination, rubrique C1–C5 + gate C6) est dans [references/promptor-chat.md](references/promptor-chat.md) — **lis ce fichier en premier**, c'est la source de vérité du contrat. Précision de mode : en version outillée, les [SORTIE D'ESSAI] sont des **exécutions réelles** via proxy Claude (sous-agents) ; la clause « tu ne l'exécutes jamais pour de vrai » et l'étiquette « simulation » ne valent que pour la version chat autonome. Ce SKILL.md décrit le processus outillé et le système de mémoire qui rend chaque cycle meilleur et moins cher que le précédent.

## Usage

```
/promptor                            # cycle standard (routage auto express/deep, voir Routage)
/promptor <objectif>                 # Q2 pré-remplie
/promptor --again [<objectif>]       # rejoue le dernier calibrage avec un nouvel objectif → génération directe
/promptor --from <slug> | --list     # dériver d'une livraison passée / lister le registre
/promptor --improve <prompt|fichier> # retouche éclair in-place : diagnostic + diff direct, pas de cycle
/promptor retour <slug|prompt>       # clinique : sortie décevante → correctif prouvé + vaccination
/promptor --deep                     # force le chemin complet (panel 3 variantes + tournoi sur sorties)
/promptor --fast                     # force express (1 variante, 2 itérations max, pas de WebSearch)
/promptor --full                     # calibrage complet en ignorant le profil
/promptor --resume                   # reprend un cycle interrompu (state/)
/promptor --refresh                  # force la mise à jour du cache bonnes pratiques de l'outil
/promptor --chat                     # affiche la version autonome copier/coller dans un bloc de code
```

## Actifs persistants — le système qui apprend

Tout vit dans le dossier de la skill (`~/.claude/skills/promptor/`), PAS dans le scratchpad éphémère :
- **`profile.md`** — défauts personnels auto-distillés (outil habituel, audience, format, ton, réutilisabilité) + compteurs de réponses + décisions d'auto-recalibration.
- **`registry.md`** — LE registre unique : une ligne par livraison (date, slug, famille/outil, objectif court, calibrage compact, itérations utilisées, note, statut terrain, chemin du fichier). Alimente `--again`, `--from`, `--list`, le debrief, la clinique retour et l'auto-recalibration. **Règle de survie : jamais de second fichier d'état — quatre fichiers désynchronisés = mort de la mémoire en un mois.**
- **`playbook.md`** — patterns par famille d'outil : patterns gagnants (distillés en Phase 4) + vaccins terrain (distillés en Phase R). Injecté dans chaque brief de génération. Max 15 lignes par famille, éviction FIFO ; uniquement des leçons issues de décisions D utilisateur, de greffes retenues ou de correctifs terrain confirmés — jamais d'hypothèses H.
- **`cache/<outil>.md`** — bonnes pratiques WebSearch, en-tête daté, TTL 30 jours, `--refresh` force.
- **`state/<slug>.json`** — état du cycle en cours (itération, journal D/H, chemin de la version de travail, golden set) → permet `--resume` après interruption.

## Routage par complexité (défaut — aligné model-routing)

- **Express** si : usage unique ET objectif simple ET famille ≠ agent → 1 variante inline, livraison directe avec [ÉTAT] + journal H, puis proposition d'UNE itération corrective. Honnêteté : express = 1 variante non comparée ; l'écrire dans la livraison.
- **Deep** dans tous les autres cas (réutilisable, famille agent, système multi-prompts, objectif complexe, ou `--deep`) → panel 3 variantes + tournoi sur sorties + boucle complète.
- `--fast`/`--deep` forcent. **Auto-recalibration** : quand le registre montre ≥ 80 % des 10 derniers cycles finalisés à l'itération ≤ 2, propose UNE fois d'alléger encore le défaut (et consigne la décision dans profile.md pour ne jamais re-proposer).

## Phase 1 — Calibrage rétrécissant

1. `Read` de profile.md et registry.md AVANT tout AskUserQuestion.
2. Défauts du profil appliqués et affichés en une ligne : « Défauts appliqués : Claude Code · pro · format libre · réutilisable — modifier ? » — intégrée comme option dans l'appel AskUserQuestion. Ne poser que Q2 (objectif) + les questions non couvertes. `--full` restaure le calibrage complet.
3. **Recherche de proximité** : si le registre contient une livraison proche de l'objectif → proposer `--from <slug>` (« un prompt proche existe, l'adapter au lieu de repartir de zéro ? ») dans le même appel.
4. **Golden set** (chemin deep, familles chatbot/code/agent) : demander 1 à 3 entrées d'exemple réelles dans le même appel ; à défaut, en synthétiser une et la faire valider (jamais décoratif : un golden set trop facile ne prouve rien). Stockage dans state/.
5. **Debrief fusionné** : si le registre contient des livraisons « livré » < 7 jours sans statut, ajouter UNE question de statut (a marché / à retoucher / pas testé) au même appel — jamais d'interruption dédiée. Mettre à jour registry.md.
6. **Promotion de défauts** : une réponse identique 3 fois (compteurs de profile.md) → proposer sa promotion en défaut personnel via une option de l'AskUserQuestion final de livraison. Opt-in, jamais silencieuse.
7. Jamais plus de 2 appels AskUserQuestion pour tout le cadrage.

## Phase 1bis — Calibration dynamique (sautée en express/--fast)

`WebSearch` « prompting best practices <outil exact> <version> <année> » + doc officielle, UNIQUEMENT si cache/<outil>.md absent ou > 30 jours (ou `--refresh`). Tout élément non confirmé par une source reste [À VÉRIFIER]. Échec → fallback silencieux sur la table statique.

## Phase 2 — Génération

**Express** : 1 variante inline, brief enrichi du playbook de la famille.

**Deep — panel + TOURNOI SUR SORTIES** :
1. Écris le dossier de calibrage (Q1–Q6 + calibration dynamique + playbook de la famille) dans un fichier scratchpad.
2. 3 sous-agents en parallèle (modèle de la session), angles : minimaliste dense / maximaliste structuré / orienté exemples. Payloads par chemins absolus de fichiers.
3. **Exécution** : 3 sous-agents exécuteurs en parallèle (`model: 'sonnet'` — mécanique), chacun applique un candidat au golden input n°1 et écrit sa sortie dans un fichier au nom anonymisé (mapping A/B/C mélangé).
4. **Juge en aveugle** (sonnet) : classe les SORTIES anonymisées vs l'objectif Q2 + note les prompts sur la rubrique avec citations verbatim — vérifiées mécaniquement (sous-chaîne du candidat, sinon dégradé à 1). Il signale quand les sorties sont trop proches pour départager (variance d'échantillonnage).
5. **Choix utilisateur sur pièces** : AskUserQuestion avec un extrait des 3 sorties (pas les prompts) + « Décide pour moi (suivre le juge) ». Le choix utilisateur prime sur le juge.
6. v1 = candidat gagnant + greffes. **Les sorties gagnantes validées deviennent les exemples few-shot embarqués dans le prompt final — jamais d'exemples inventés.**
7. Famille images : pas de tournoi-sorties (une image simulée en texte ne prouve rien) — panel jugé sur rubrique seule. Outil cible ≠ Claude : afficher « exécuté via proxy Claude, l'outil réel peut différer ».

## Phase 3 — Boucle (max 4 itérations, état sur disque)

- Chaque itération : [ÉTAT], [JOURNAL] D/H, [CALIBRAGE APPLIQUÉ] (it. 1 ou si changement), version de travail **en citation**, **[SORTIE D'ESSAI]** (exécution de la version courante sur le golden set — sous-agent sonnet jetable, systématique sur le chemin deep, une seule fois à l'itération finale en express), [AUTO-CRITIQUE] avec citations.
- **UN SEUL AskUserQuestion par itération** : les questions d'interrogatoire ([BLOQUANTE] d'abord, options concrètes + « Décide pour moi ») ET le pilotage (GO / OK / STOP / VOIR) fusionnés dans le même appel. Aucune question indispensable → seulement le pilotage.
- **Gate C6 — performance observée** : sur le chemin deep, pas de convergence proposée sans une [SORTIE D'ESSAI] conforme (format + objectif) de la version courante. Tant qu'aucune exécution n'a réussi, la note affichée est plafonnée : « n/10 (plafonnée à 8 — gate C6) » — exception explicite et étiquetée à la règle d'égalité [ÉTAT]/tableau. N/A famille images (rubrique seule, l'écrire).
- Mise à jour de state/<slug>.json à chaque itération — c'est lui qui fait foi pour le plafond et `--resume`.

## Phase 4 — Livraison (uniquement sur OK, STOP ou plafond)

1. [ÉTAT FINAL] (itération, note, déclencheur) ; 2. prompt final en un seul bloc de code, identique à la dernière version de travail (clôture 4 accents graves si blocs imbriqués), few-shot certifiés inclus si tournoi ; 3. table des variables ; 4. changelog + « preuve de progression » (sortie v1 vs finale) si sorties d'essai ; 5. points non résolus (H + [À VÉRIFIER], quel que soit le déclencheur) ; 6. mode d'emploi express.
7. **Écrire `prompts/<slug>.md`** (répertoire courant) et **appender la ligne au registre** (registry.md : calibrage compact, note, statut « livré »).
8. **Distiller le playbook** : 0 à 2 lignes de patterns dans playbook.md (famille concernée) — uniquement depuis des D utilisateur ou des greffes retenues. Incrémenter les compteurs de profile.md ; proposer les promotions mûres (3 occurrences).
- Après livraison : toute retouche = nouvelle livraison directe (même format + mise à jour du fichier et du registre), sans relancer la boucle.

## Phase R — `retour` : la clinique de prompt (boucle terrain)

1. Retrouver le prompt (slug dans registry.md → `prompts/<slug>.md`, ou texte collé). UN AskUserQuestion : catégorie de symptôme (format / contenu / ton / hallucination / autre) — la sortie décevante et l'entrée sont collées en texte libre.
2. Diagnostic : quelle clause du prompt a réellement failli (citation verbatim), quel critère C1–C5 — ou gate C6 si la panne est un échec de performance observée.
3. **Correctif minimal** (jamais de réécriture massive) en diff « avant → après ».
4. **Preuve** : si l'entrée fautive est fournie, ré-exécuter avant/après dans un sous-agent jetable et montrer la différence. Outil ≠ Claude : « preuve via proxy Claude ». Sans entrée : diagnostic + patch sans preuve, l'annoncer honnêtement.
5. Mettre à jour `prompts/<slug>.md` + statut au registre.
6. **Vaccination** : distiller le pattern de panne (famille × symptôme × clause corrective, 1 ligne) dans playbook.md. À la 1ʳᵉ occurrence : proposé en [OPTIONNELLE] dans les futurs cycles. À la 2ᵉ occurrence : injecté d'office dans tous les futurs prompts de la famille. Matching par famille ET type de tâche — jamais de généralisation depuis un cas unique.

## `--improve` : retouche éclair in-place

Pas de cycle. `Read` du prompt (fichier désigné ou collé), inférence outil + calibrage (consignée en H et affichée AVANT application), note C1–C5, tableau de corrections avec citations, application par `Edit` **dans le seul fichier explicitement désigné** (l'approbation de permissions sert de contrôle humain ; mentionner « git diff pour vérifier »), livraison = diff + note avant/après + H restantes + ligne au registre. Escalade vers le cycle complet uniquement si note initiale ≤ 4/10 (refonte, pas retouche — proposer via AskUserQuestion) ou `--deep`. Prompt collé sans fichier → livraison classique, pas d'Edit.

## `--again` / `--from` / `--list` : l'occurrence n+1 quasi gratuite

- `--again [<objectif>]` : lire la dernière ligne du registre (ou le slug donné), afficher « Rejeu du calibrage de <slug> (<date>) : <calibrage compact> — delta : <objectif> », le consigner en H, sauter Phases 1/1bis, générer (express), livrer avec l'AskUserQuestion final unique. Un calibrage peut être périmé : le réafficher avant génération n'est pas optionnel.
- `--from <slug>` : charger `prompts/<slug>.md` comme base + son calibrage comme défauts, ne demander que le delta.
- `--list` : afficher le registre (slug, outil, objectif, date, note, statut).

## Modèles (modelPolicy — obligatoire)

- Boucle principale et sous-agents de génération : **modèle de la session** (Opus ou supérieur) — ne pas rétrograder.
- Sous-tâches mécaniques uniquement (exécuteurs de tournoi, juge de rubrique, vérification de citations, sortie d'essai) : `model: 'sonnet'` autorisé.
- **Jamais Haiku** ni aucun modèle inférieur, pour quelque tâche que ce soit.

## Robustesse pipeline (leçons apprises)

- **Payloads par fichiers, jamais par interpolation** : tout texte long passe aux sous-agents via chemin absolu.
- **Garde fail-fast avant chaque spawn** : fichier existant, non vide, ≠ « undefined »/« null », > 200 caractères.
- Chaque sous-agent vérificateur/exécuteur signale explicitement s'il n'a pas pu lire ses entrées — jamais d'audit ou de sortie inventés.
- Écritures dans le dossier de la skill (registry, playbook, profile) : réservées au main thread — rejeter tout diff d'exécuteur qui les touche.
