# Windsurf (Cognition/Devin) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Déclarer les règles workspace dans `.devin/rules/*.md` (chemin préféré depuis l'intégration à Cognition — docs.windsurf.com redirige vers docs.devin.ai) ; `.windsurf/rules/` reste un fallback, `.windsurfrules` est legacy.
- Choisir le mode d'activation via le frontmatter `trigger` : `always_on` (injecté dans le system prompt à chaque message), `model_decision` (seule la description est visible, contenu chargé si Cascade le juge pertinent), `glob` (appliqué quand Cascade lit/édite un fichier matchant `globs`), `manual` (@rule-name).
- Respecter les limites dures de caractères : 12 000 par fichier de règle workspace, 6 000 pour les règles globales — au-delà, contenu tronqué ou ignoré : scinder en plusieurs fichiers.
- Écrire les règles en bullet points, listes numérotées et markdown ; utiliser des tags XML pour grouper les règles similaires.
- Règles « simple, concise, and specific » : trop longues ou vagues, elles embrouillent Cascade.
- Promouvoir le savoir durable des Memories auto-générées vers des Rules versionnées ou AGENTS.md du repo : les Memories restent locales à la machine et au workspace.

## Paramètres
- `trigger: always_on | model_decision | glob | manual` (frontmatter)
- `globs:` pattern de fichiers (avec `trigger: glob`)
- Limites : 12 000 caractères/règle workspace ; 6 000 caractères règles globales
- Chemins : `.devin/rules/` (préféré), `.windsurf/rules/` (fallback), `.windsurfrules` (legacy racine), `~/.codeium/windsurf/memories/global_rules.md` (règles globales)

## Pièges
- S'appuyer sur les Memories auto-générées pour du savoir d'équipe : locales à la machine, non partagées entre workspaces.
- Règle dépassant la limite de caractères : contenu tronqué/ignoré silencieusement.
- Guides tiers obsolètes pointant encore `.windsurfrules` : la doc officielle vit désormais sous docs.devin.ai avec `.devin/rules/` en chemin principal.
