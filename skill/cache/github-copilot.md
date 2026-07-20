# GitHub Copilot — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Structurer `.github/copilot-instructions.md` en 5 blocs : (1) pitch projet en quelques phrases, (2) tech stack avec notes d'usage, (3) coding guidelines (points-virgules, type hints, exigences de tests), (4) structure des dossiers, (5) scripts/outils/serveurs MCP disponibles.
- Ajouter les commandes exactes de build/test/lint, le setup d'environnement et les workflows CI. Tenir le tout en ~2 pages.
- Un fichier imparfait vaut mieux que rien : publier tôt, itérer ensuite.
- Instructions par chemin : `.github/instructions/NAME.instructions.md` avec frontmatter `applyTo` en globs ; `excludeAgent` pour restreindre à un usage.
- AGENTS.md supporté n'importe où dans le repo (le plus proche gagne) ; CLAUDE.md/GEMINI.md à la racine.
- Coding agent : rédiger les issues comme des prompts — problème clair + critères d'acceptation complets (tests requis ?) + fichiers à toucher. Test : « l'issue fonctionnerait-elle comme prompt IA ? ».
- Bonnes cibles coding agent : bug fixes, UI, couverture de tests, docs, dette technique. À éviter : refactoring transverse, code sensible sécurité/prod.
- Pré-installer les dépendances du coding agent via `copilot-setup-steps.yml`.
- Adapter le prompt au mode : Ask = comprendre sans modifier ; Edit = directive chirurgicale scoped avec diff approuvé ; Agent = objectif haut niveau avec intention, périmètre et condition d'arrêt, boucle verify–fix–verify (tests/lint après chaque changement).
- Agent mode : les éditions de fichiers s'appliquent sans approbation préalable, mais les commandes potentiellement risquées restent présentées pour revue avant exécution — d'où l'importance de custom instructions détaillées.

## Paramètres
- Fichier repo-wide : `.github/copilot-instructions.md`
- Frontmatter .instructions.md : `applyTo: "app/models/**/*.rb"` (patterns multiples séparés par virgules) ; `excludeAgent: "code-review"|"cloud-agent"`
- `copilot-setup-steps.yml` : pré-installation des dépendances du coding agent
- Longueur cible des instructions : ~2 pages (la limite « ~1000 lignes » est une source tierce [À VÉRIFIER])

## Pièges
- Instructions trop longues ou narratives — préférer des directives impératives courtes sous des titres distincts.
- Assigner au coding agent des tâches floues, du refactoring transverse ou du code sensible.
- Commentaires de PR envoyés un par un : chaque envoi déclenche une itération — grouper via « Start a review ».
