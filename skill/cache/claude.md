# Claude (Anthropic) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Encapsuler chaque type de contenu dans des balises XML descriptives : `<instructions>`, `<context>`, `<examples>`, `<input>` ; noms de tags cohérents d'un prompt à l'autre. Surtout utile sur les prompts complexes.
- Documents longs (20k+ tokens) : `<documents><document index="n"><source>…</source><document_content>…</document_content></document></documents>` placés EN HAUT ; requête et instructions à la FIN (jusqu'à +30 % de qualité).
- Grounding long contexte : demander d'abord l'extraction des citations pertinentes dans `<quotes>` avant d'exécuter la tâche.
- Multishot : 3–5 exemples réalistes et diversifiés dans `<examples>`, couvrant les cas limites ; avec thinking activé, inclure des blocs `<thinking>` dans les exemples (le style de raisonnement est généralisé).
- Donner le pourquoi de chaque contrainte : « …read aloud by a text-to-speech engine, so never use ellipses » > « NEVER use ellipses ». Gabarit Fable 5 : « I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request]. »
- Formater par affirmation : dire quoi faire, pas quoi éviter (« Your response should be composed of smoothly flowing prose paragraphs » plutôt que « Do not use markdown ») ; snippets officiels `<avoid_excessive_markdown_and_bullet_points>` et anti-LaTeX. [À VÉRIFIER]
- JAMAIS de prefill du dernier tour assistant (erreur 400 dès Claude 4.6). JSON/classification → Structured Outputs ou tool à champ enum ; anti-préambule → « Respond directly without preamble. Do not start with phrases like "Here is…" » ; continuation → « Your previous response was interrupted and ended with [previous_response]. Continue from where you left off. »
- Diriger l'action : « Change this function to improve its performance » plutôt que « Can you suggest some changes » ; snippets `<default_to_action>`, `<do_not_act_before_instructions>`, `<use_parallel_tool_calls>`.
- Dé-agressiver les prompts hérités : remplacer « CRITICAL: You MUST use this tool when… » par « Use this tool when… » ; supprimer « If in doubt, use [tool] » (sur-déclenchement sur Opus 4.5/4.6).
- System prompt d'agent : ensemble minimal d'informations décrivant entièrement le comportement ; sections distinctes (`<background_information>`, `<instructions>`, ## Tool guidance) ; exemples canoniques plutôt qu'énumération exhaustive.
- Nommer explicitement la capacité voulue (« use web search », « deep research ») pour forcer son déclenchement ; structurer en `<context>`/`<task>`/`<output_format>` ; les instructions explicites priment sur les défauts de formatage du modèle.

## Paramètres
- Rôle en une phrase dans le champ `system` de l'API (pas dans le message user).
- 3–5 exemples dans `<examples>` ; requête placée après les documents longs (jusqu'à +30 %).

## Pièges
- Guides 2025 recommandant encore le prefill : obsolète, 400 sur Claude 4.6+ — vérifier la date de tout guide.
- Le style du prompt fuit dans la sortie : markdown dans le prompt = markdown en sortie.
- Instructions négatives moins fiables que l'instruction positive équivalente.
- Sur-structurer un prompt simple : XML/role prompting n'apportent de valeur nette que sur les prompts complexes. [À VÉRIFIER]
- Modèles récents concis par défaut : demander explicitement les résumés après tool calls si besoin de visibilité.
