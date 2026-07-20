# Claude Code — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- CLAUDE.md court et élagué : n'inclure que ce que Claude ne peut pas deviner en lisant le code. Test par ligne : « sa suppression causerait-elle des erreurs ? »
- Inclure : commandes bash non devinables, style divergeant des défauts, test runners, gotchas du repo. Exclure : conventions standard, doc API, descriptions fichier par fichier.
- Renforcer les règles critiques avec IMPORTANT / YOU MUST ; importer d'autres fichiers via @path/to/import ; générer avec /init, auditer avec /context.
- Le savoir occasionnel (procédures rares) va dans un skill, pas dans CLAUDE.md.
- Toujours donner un critère de fin exécutable (tests, build, screenshot à comparer). Gradation de dureté : prompt → /goal → Stop hook → subagent relecteur.
- Le Stop hook bloque la fin de tour jusqu'au vert (Claude Code force la fin après 8 blocages consécutifs).
- Exiger des preuves (sortie de tests, commande exécutée), pas une affirmation de succès.
- Review adverse : subagent en contexte frais qui ne voit que le diff + les critères ; lui demander de ne signaler QUE les écarts de correction (sinon sur-rapportage).
- Workflow Explore → Plan → Code → Commit en plan mode ; skipper le plan si le diff se décrit en une phrase.
- Préserver le contexte : /clear entre tâches non liées, /compact <instructions> ciblé, subagents d'investigation qui rapportent des résumés.
- Après 2 corrections ratées : /clear et réécrire un meilleur prompt initial plutôt qu'itérer.

## Paramètres
- Headless : claude -p "prompt" --output-format stream-json --verbose
- Scoper un fan-out batch : --allowedTools "Edit,Bash(git commit *)"
- Exécution non supervisée avec classifieur : claude --permission-mode auto
- Compaction ciblée : /compact <instructions>

## Pièges
- Session fourre-tout (tâches mélangées) : faire /clear entre tâches.
- CLAUDE.md sur-spécifié : les règles importantes se perdent dans le bruit.
- Investigation non scopée qui remplit le contexte de centaines de fichiers.
- Accepter du code plausible sans vérification exécutable.
- Chasser tous les findings d'un reviewer : sur-ingénierie.
