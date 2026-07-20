# Cursor — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Placer les règles projet dans `.cursor/rules/*.mdc` — un fichier .md ordinaire y est ignoré (seuls .mdc et AGENTS.md sont pris en compte).
- Choisir le mode d'activation via le frontmatter : `alwaysApply: true` (injecté à chaque session), `globs` (attachement auto quand un fichier correspondant est actif), `description` (l'agent décide de la pertinence), aucun des trois = invocation manuelle via @rule-name.
- Quatre types actuels : Always Apply / Apply Intelligently / Apply to Specific Files / Apply Manually — « Auto Attached » et « Agent Requested » sont les anciens libellés, mécanismes identiques.
- Garder chaque règle sous 500 lignes ; scinder les grosses règles en règles composables.
- Écrire les règles comme une doc interne claire : exemples concrets, références de fichiers plutôt que code copié.
- AGENTS.md supporté, y compris imbriqué dans les sous-dossiers — le plus spécifique gagne.

## Paramètres
- `alwaysApply: true|false` (frontmatter .mdc)
- `globs:` patterns séparés par virgules (ex. `src/**/*.tsx`)
- `description:` texte lu par l'agent pour décider de charger la règle
- Limite recommandée : 500 lignes par règle
- Précédence : Team Rules → Project Rules → User Rules ; AGENTS.md imbriqué le plus proche gagne

## Pièges
- `alwaysApply: true` fait ignorer `globs` et `description` — une règle « always » avec globs ne filtre rien.
- Ne pas dupliquer dans les règles des style guides ou du code déjà présents dans le repo.
- Fichier .md simple dans `.cursor/rules` = ignoré silencieusement.
- Prompting des complétions inline (Tab) : aucune doc officielle dédiée [À VÉRIFIER].
