# Base de connaissances — Bonnes pratiques de prompting (juillet 2026)

Dernière mise à jour : 2026-07-20

Cette base alimente la **Phase 1bis** de Promptor (calibration dynamique). Elle est issue d'une
recherche multi-sources sur les documentations officielles (273 requêtes web), suivie d'une
**vérification adversariale indépendante** (116 requêtes de contre-vérification : sources consultées
une à une, versions et paramètres recoupés — fiabilité mesurée : ~95-98 % selon la famille).

Marquage : **✓** confirmé en source primaire par le vérificateur · **⚠ à recouper** plausible mais
non contre-vérifié · **[À VÉRIFIER]** incertain. Ce qu'aucune source fiable n'établit est listé en
fin de section (« Non établi ») — jamais inventé.

---

## OpenAI (GPT-5.x, o-series, API)
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — 15 des 16 affirmations confirmées à la source primaire, aucune réfutée, une seule pratique obsolète corrigée.*

### GPT-5.x via API (Responses API) — prompting général et agentique
**État courant** — GPT-5.6 (sorti ~9 juillet 2026, knowledge cutoff 16 février 2026) décliné en trois tiers : Sol (`gpt-5.6-sol`, frontier, 5 $/30 $ par M tokens), Terra (équilibre, 2,50 $/15 $) et Luna (coût/latence, 1 $/6 $) ; contexte 1,05 M tokens ; `reasoning_effort` jusqu'à xhigh/max. Guides de prompting officiels publiés jusqu'à GPT-5.2 (11 déc. 2025) ; la guidance GPT-5.5 est publiée dans la doc API officielle (« Model guidance »).

**Pratiques clés**
- **Responses API + `previous_response_id`** — OpenAI « recommande fortement » la Responses API : réutiliser les reasoning items entre les tours a fait monter Tau-Bench Retail de 73,9 % à 78,2 % dans le guide officiel. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) ✓
- **Fixer explicitement `reasoning_effort` à la migration** — Mapping officiel : GPT-4o/4.1 → none ; GPT-5 « minimal » → none ; GPT-5.1 → même valeur. Attention : le défaut dépend du modèle (none pour GPT-5.1/5.2, medium pour GPT-5, GPT-5.5 et GPT-5.6) — ne monter le niveau qu'après évaluation, jamais par précaution. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ✓
- **Contrôler la longueur avec `verbosity` + overrides en langage naturel** — Le paramètre règle la longueur globale de la réponse finale ; le modèle respecte des overrides ciblés dans le prompt (exemple Cursor : verbosity basse globalement, haute uniquement pour les outils d'édition de code). [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) ✓
- **Developer message ordonné Identity → Instructions → Examples → Context** — Ordre officiel avec délimiteurs Markdown/XML ; le rôle developer prime sur user (chaîne de commandement du Model Spec). Le placement statique-en-tête/variable-en-fin et le seuil de 1024 tokens relèvent du guide prompt-caching (attribution corrigée). [source](https://developers.openai.com/api/docs/guides/prompt-engineering) ✓
- **Encapsuler chaque comportement dans des blocs XML nommés** — `<persistence>`, `<context_gathering>`, `<tool_preambles>`, `<self_reflection>` (GPT-5) ; `<solution_persistence>`, `<final_answer_formatting>`, `<user_updates_spec>` (5.1) ; `<design_and_scope_constraints>`, `<uncertainty_and_ambiguity>`, `<long_context_handling>`, `<extraction_spec>`, `<high_risk_self_check>` (5.2). [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ✓
- **Calibrer l'« eagerness » agentique dans les deux sens** — Réduire : budget explicite (« an absolute maximum of 2 tool calls »), critère d'arrêt, clause d'échappement « even if it might not be fully correct ». Augmenter : « keep going until the user's query is completely resolved », interdire les questions de clarification, documenter les hypothèses après action. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) ✓
- **User updates brèves et cadencées en tâche agentique longue** — 1-2 phrases au moins toutes les 6 étapes d'exécution ou 8 tool calls, chacune avec un résultat concret ; plan rapide avant le premier tool call ; jamais de narration des appels routiniers. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-1_prompting_guide) ✓
- **Metaprompting chirurgical contre les contradictions** — GPT-5.x est plus pénalisé que ses prédécesseurs par les prompts contradictoires ou vagues. Méthode officielle : (1) faire diagnostiquer au modèle les sections fautives, (2) demander des patchs ciblés plutôt qu'une réécriture complète. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-1_prompting_guide) ✓
- **À la migration, repartir d'une baseline minimale** — « Start with the smallest prompt that preserves the product contract », puis réajuster reasoning effort, verbosity, descriptions d'outils et format de sortie sur des exemples représentatifs. Guidance officielle OpenAI (doc API « Model guidance »), relayée par Willison. [source officielle](https://developers.openai.com/api/docs/guides/prompt-guidance) · [analyse](https://simonwillison.net/2026/apr/25/gpt-5-5-prompting-guide/) ✓
- **Borner le scope : « Implement EXACTLY and ONLY what the user requests »** — Formulation exacte de `<design_and_scope_constraints>` (5.2) ; `<output_verbosity_spec>` chiffre les cibles : 3-6 phrases ou ≤5 bullets en standard, ≤2 phrases pour une question oui/non. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ✓

**Paramètres** : `reasoning_effort` — échelle complète none, minimal, low, medium, high, xhigh, max (valeurs et défauts dépendants du modèle ; GPT-5.6 : none/low/medium/high/xhigh/max) · `verbosity` (low/medium/high) · `previous_response_id` (Responses API) · prompt caching : préfixe ≥1024 tokens, `prompt_cache_key` ~15 req/min par clé ; GPT-5.6+ : `prompt_cache_options.ttl` « 30m » (seule valeur), écritures facturées 1,25× l'input non caché ; modèles antérieurs : `prompt_cache_retention` (in_memory/24h), écritures gratuites · endpoint `POST /v1/responses/compact` (GPT-5.2+) pour compacter les workflows agentiques longs.

**Pièges** :
- Laisser le `reasoning_effort` par défaut à la migration — profil latence/coût/verbosité imprévisible (défauts variables : none en 5.1/5.2, medium en 5.5/5.6).
- Prompts contradictoires : plus dommageables sur GPT-5.x que sur les générations précédentes.
- Markdown désactivé par défaut en API — l'induire explicitement (« Use Markdown only where semantically correct », rafraîchir tous les 3-5 messages utilisateur).
- API v1/prompts (prompt objects réutilisables) dépréciée le 3 juin 2026, arrêt le 30 novembre 2026 — versionner les prompts dans le code avec CI/CD.
- Narration de chaque tool call et formules de politesse répétées (« got it ») : explicitement déconseillées par les guides 5.1/5.2.

### Modèles de raisonnement (o-series et modes reasoning des GPT-5.x)
**État courant** — Le raisonnement est intégré aux GPT-5.x via `reasoning_effort` ; les o-series sont en retrait officiel : o3-2025-04-16 déprécié (annonce du 11 juin 2026, arrêt le 11 décembre 2026), o4-mini-2025-04-16 retiré le 23 octobre 2026 (remplacement recommandé : gpt-5.4-mini). Le guide « Reasoning best practices » reste la base documentaire pour tout mode raisonnement.

**Pratiques clés**
- **Ne PAS demander de chain-of-thought explicite** — Le raisonnement est interne : « think step by step » est inutile et peut dégrader la performance ; ne pas non plus induire de raisonnement supplémentaire avant chaque appel de fonction. [source](https://developers.openai.com/api/docs/guides/reasoning-best-practices) ✓
- **Zero-shot d'abord, contraintes et critères de succès explicites** — Commencer sans exemple few-shot, n'en ajouter que si le format l'exige ; objectif clair, contraintes fortes, contrat de sortie — sans prescrire les étapes intermédiaires. En RAG, ne fournir que le contexte le plus pertinent : le surplus dilue la précision. [source](https://developers.openai.com/api/docs/guides/reasoning-best-practices) ✓
- **Router selon « planners vs workhorses »** — Raisonnement élevé pour planification, ambiguïté, domaines experts (priorité exactitude) ; GPT/effort bas pour l'exécution rapide de tâches bien définies (priorité vitesse/coût). Pattern recommandé : le modèle de raisonnement planifie, un modèle rapide exécute. [source](https://developers.openai.com/api/docs/guides/reasoning-best-practices) ✓
- **Function calling : critères d'usage dans la description, règles critiques en premier** — Chaque description d'outil dit quand l'invoquer et comment construire les arguments (few-shot dans la description pour formats complexes type regex) ; règles critiques en tête (+6 % de précision mesurée sur l'exemple grep) ; anti-hallucination : « Do NOT promise to call a function later. If a function call is required, emit it now. » [source](https://developers.openai.com/cookbook/examples/o-series/o3o4-mini_prompting_guide) ✓

**Paramètres** : rôle developer (remplace system depuis o1-2024-12-17, conversion automatique system→developer) · Responses API avec `store:true` ou `include=["reasoning.encrypted_content"]` pour persister les reasoning items entre les tours (Chat Completions ne les préserve pas) · dimensionnement function calling : ~100 outils max, ~20 arguments par outil, schémas plats de préférence.

**Pièges** :
- « Formatting re-enabled » en première ligne du developer message : workaround temporaire propre aux o-series legacy (o1-2024-12-17+), absent de la doc reasoning courante — sur GPT-5.x, utiliser l'instruction Markdown du guide GPT-5.
- Prompter « think step by step » ou forcer un raisonnement à voix haute : contre-productif.
- Chat Completions pour un agent multi-tours : contexte de raisonnement perdu à chaque tour (re-raisonnement coûteux).
- Surcharger le contexte (RAG massif, historique d'outils non pertinent) : élaguer et résumer les anciens tool calls.

### Structured Outputs / sorties JSON (API)
**État courant** — Doc courante (juillet 2026) : « pour les nouveaux projets, partir de gpt-5.6 » ; supporté depuis gpt-4o-2024-08-06 / gpt-4o-mini. Présenté comme l'évolution du JSON mode.

**Pratiques clés**
- **Préférer Structured Outputs (json_schema + strict:true) au JSON mode** — Seul Structured Outputs garantit l'adhérence au schéma (le JSON mode ne garantit que la validité JSON). Config exacte : `type: json_schema`, `strict: true`, `additionalProperties: false`, toutes les clés dans `required` ; documenter les champs via titles/descriptions du schéma. [source](https://developers.openai.com/api/docs/guides/structured-outputs) ✓
- **Gérer le champ `refusal` dans le code applicatif** — Un refus de sécurité peut remplacer la sortie structurée sans respecter le schéma : tester `response.refusal` avant de parser. [source](https://developers.openai.com/api/docs/guides/structured-outputs) ✓
- **Extraction : schéma dans le prompt, null plutôt que deviner** — Pattern `<extraction_spec>` du guide 5.2 : schéma JSON explicite, distinguer champs requis/optionnels, imposer null pour toute valeur absente, exiger un re-scan du document source avant de finaliser la sortie. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ✓

**Paramètres** : Responses API `text.format = { type: "json_schema", name, schema, strict: true }` (Chat Completions : `response_format` équivalent) · `additionalProperties: false` obligatoire + toutes les clés dans `required` · function calling strict : `"strict": true` dans le schéma des outils.

**Pièges** :
- JSON mode (`type: json_object`) ne garantit que du JSON syntaxiquement valide, pas le respect du schéma.
- Première requête avec un nouveau schéma : latence additionnelle de compilation (les suivantes retrouvent la performance normale).
- Laisser le modèle deviner les valeurs manquantes au lieu d'imposer null : source d'hallucinations en extraction.
- Schémas profondément imbriqués : préférer des schémas plats (~100 outils, ~20 arguments en pratique).

**Non établi faute de source fiable** :
- Guide de prompting officiel dédié à GPT-5.6 et à ses tiers Sol/Terra/Luna (modèle trop récent — seuls la page modèles, l'annonce et les docs structured outputs/caching le mentionnent).
- Guidance officielle d'usage pour les niveaux xhigh et max de `reasoning_effort` (listés sur la page modèles GPT-5.6, sans guide associé).
- Bonnes pratiques officielles pour ChatGPT grand public (custom instructions, projets, mémoire) — les sources consultées ciblent l'API et les développeurs.
- Différentiel exact GPT-5.4 (mars 2026) : aucun guide de prompting dédié trouvé pour cette version intermédiaire.

---

## Anthropic Claude
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — 12 des 13 affirmations vérifiées confirmées contre les sources primaires (souvent mot pour mot, snippets et chiffres inclus), une seule erreur chiffrée corrigée (minimum cacheable Haiku 4.5), aucune pratique réfutée.*

### Claude — prompting général (API Messages et Claude.ai, tous modèles courants)
**État courant** — Modèles courants dans la doc officielle (19/07/2026) : Claude Fable 5, Claude Mythos 5, Opus 4.8/4.7/4.6, Sonnet 5/4.6, Haiku 4.5. La page « Prompting best practices » est consolidée pour toute cette génération.

**Pratiques clés**
- **Balises XML descriptives et cohérentes** — Encapsuler chaque type de contenu dans son tag (`<instructions>`, `<context>`, `<examples>`, `<input>`), imbriquer si hiérarchie (`<documents><document index="n"><source>…</source><document_content>…</document_content></document></documents>`), garder les mêmes noms d'un prompt à l'autre ; la doc cadre les XML tags comme utiles surtout pour les prompts complexes. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Long contexte (20k+ tokens) : documents en haut, requête à la fin** — Placer la requête après les documents longs améliore la qualité jusqu'à +30 % dans les tests, surtout en multi-documents ; demander d'abord l'extraction des citations pertinentes dans `<quotes>` (grounding par citation). [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Multishot : 3–5 exemples diversifiés dans `<examples>`** — Exemples réalistes couvrant les cas limites, assez variés pour ne pas induire de patterns involontaires ; avec thinking, inclure des blocs `<thinking>` dans les exemples (le style est généralisé) ; Claude peut évaluer la diversité des exemples ou en générer. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Donner le pourquoi de chaque contrainte** — « Your response will be read aloud by a text-to-speech engine, so never use ellipses » fonctionne mieux que « NEVER use ellipses ». Gabarit officiel pour Fable 5 : « I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request]. » [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) · [gabarit Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) ✓
- **Formater par affirmation** — Dire quoi faire, pas quoi éviter (« smoothly flowing prose paragraphs » plutôt que « Do not use markdown ») ; le style du prompt fuit dans la sortie ; snippets officiels `<avoid_excessive_markdown_and_bullet_points>` et anti-LaTeX à coller dans le system prompt. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ⚠ à recouper
- **Plus jamais de prefill du dernier tour assistant (400 dès Claude 4.6)** — Migrer vers Structured Outputs (ou tool à champ enum) pour JSON/classification ; « Respond directly without preamble. Do not start with phrases like "Here is…" » contre les préambules ; continuation déplacée dans le message user (« Your previous response was interrupted and ended with [previous_response]. Continue from where you left off. »). [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Diriger explicitement l'action et dé-agressiver les prompts hérités** — « Change this function to improve its performance » plutôt que « Can you suggest some changes » ; snippets `<default_to_action>` / `<do_not_act_before_instructions>` / `<use_parallel_tool_calls>` ; sur Opus 4.5/4.6, remplacer « CRITICAL: You MUST use this tool when… » par « Use this tool when… » et supprimer « If in doubt, use [tool] » (sur-déclenchement). [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **System prompt d'agent à la bonne « altitude »** — Viser l'ensemble minimal d'informations décrivant entièrement le comportement attendu ; sections distinctes (`<background_information>`, `<instructions>`, ## Tool guidance) ; exemples canoniques plutôt qu'énumération exhaustive des cas limites ; long terme : compaction, NOTES.md, sous-agents renvoyant des résumés condensés (1 000–2 000 tokens), récupération just-in-time. [source](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) ✓
- **Déclarer le format de sortie et nommer la capacité à activer** — Les instructions explicites priment sur les défauts du modèle (bullets vs prose) ; nommer un outil (« Perform deep research and use web search to verify latest developments ») force son déclenchement ; structurer en `<context>`/`<task>`/`<output_format>`. [source](https://www.pantaleone.net/blog/claude-opus-4.6-system-prompt-analysis-tuning-insights-template) ✓

**Paramètres** :
- Rôle en une phrase dans le champ `system` de l'API (pas dans le message user).
- 3–5 exemples dans `<examples>` ; documents dans `<documents><document index="n">` avec `<source>` et `<document_content>`.
- Requête placée APRÈS les documents longs : jusqu'à +30 % de qualité constatée en interne.

**Pièges** :
- Guides 2025 (y compris officiels) recommandant encore le prefill, rejeté (400) sur Claude 4.6+ — toujours vérifier la date du guide.
- Le style du prompt fuit dans la sortie : un prompt bourré de markdown produit une sortie bourrée de markdown.
- Instructions négatives (« ne fais pas X ») moins fiables que l'instruction positive équivalente.
- Sur-structurer un prompt simple : XML et role prompting n'apporteraient de valeur nette que sur les prompts complexes (source blog nov. 2025 non retrouvée ; inférence cohérente avec la doc) ⚠ à recouper.
- Modèles récents plus concis par défaut : ils peuvent sauter les résumés après tool calls — les demander explicitement.

### Adaptive thinking + paramètre effort (API)
**État courant** — `thinking: {type:"adaptive"}` sur Opus 4.6→4.8 et Sonnet 4.6 ; toujours actif (non désactivable) sur Fable 5/Mythos 5 ; actif par défaut sur Sonnet 5. `budget_tokens` déprécié sur Opus 4.6/Sonnet 4.6 et rejeté (400) sur Opus 4.7+ et la famille 5.

**Pratiques clés**
- **Piloter la profondeur via effort + adaptive thinking, plus jamais via budget_tokens** — Syntaxe : `thinking: {type:"adaptive"}` + `output_config: {effort:"low"|"medium"|"high"|"xhigh"|"max"}` ; high = défaut (identique à l'omission) et affecte TOUS les tokens (texte, tool calls, thinking). Recommandations : Opus 4.7/4.8 → démarrer à xhigh pour coding/agentic avec max_tokens ~64k ; Fable 5 → high par défaut, xhigh pour le plus sensible, medium/low pour le routinier ; Sonnet 4.6 → medium par défaut ; low pour sous-agents et gros volumes. [source](https://platform.claude.com/docs/en/build-with-claude/effort) ✓
- **Guider le raisonnement par instructions générales + auto-vérification** — « Think thoroughly » bat souvent un plan pas-à-pas imposé ; « Before you finish, verify your answer against [test criteria] » attrape les erreurs de façon fiable ; interleaved : « After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding. » [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ⚠ à recouper

**Paramètres** :
- `thinking: {type:"adaptive"}` ; `output_config: {effort:"high"}` = défaut.
- xhigh : Fable 5, Mythos 5, Opus 4.8/4.7, Sonnet 5 ; max : idem + Opus 4.6 et Sonnet 4.6 — indisponible sur Haiku 4.5, Sonnet 4.5 et Opus 4.5.
- max_tokens ≥ 64k recommandé à xhigh/max (plafond dur couvrant thinking + réponse).

**Pièges** :
- `budget_tokens` : 400 sur Opus 4.7+ et Fable 5/Mythos 5 ; déprécié sur Opus 4.6/Sonnet 4.6 — utiliser effort ou max_tokens.
- Overthinking à effort élevé : baisser effort plutôt qu'empiler des instructions ; sinon snippet officiel « choose an approach and commit to it… avoid revisiting decisions ».
- Opus 4.5, thinking désactivé, hypersensible au mot « think » — préférer « consider », « evaluate », « reason through ».
- Fable 5 : demander la retranscription du raisonnement interne déclenche le refus `reasoning_extraction` — lire les blocs thinking structurés de l'API.

### Prompt caching (API) — implications sur la structure du prompt
**État courant** — `cache_control {type:"ephemeral"}`, TTL 5 min (défaut) ou 1h. Minimums cacheables (19–20/07/2026) : 512 tokens (Fable 5/Mythos 5), 1 024 (Opus 4.8, Sonnet 5, Sonnet 4.6, Sonnet 4.5), 2 048 (Opus 4.7), 4 096 (Opus 4.6, Opus 4.5, Haiku 4.5).

**Pratiques clés**
- **Structurer chaque requête « stable d'abord » : tools → system → messages** — Le cache est un préfixe à correspondance exacte haché dans cet ordre : tools, system, gros contexte et exemples en tête, contenu variable en toute fin ; `cache_control: {type:"ephemeral"}` sur le DERNIER bloc identique entre requêtes — jamais sur un bloc à timestamp/contenu par requête (l'isoler après le breakpoint). Max 4 breakpoints ; TTL 1h via `ttl:"1h"`. Coûts : écriture 1,25x (5 min) ou 2x (1h), lecture 0,1x ; pre-warm avec `max_tokens: 0` ; le caching automatique suffit pour le multi-tours simple. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) ✓

**Paramètres** :
- `cache_control: {type:"ephemeral"}` sur le dernier bloc statique ; max 4 breakpoints ; fenêtre de lookback de 20 blocs.
- `ttl:"1h"` (écriture 2x) vs défaut 5 min (écriture 1,25x) ; lecture 0,1x du prix input.
- Suivi via `usage.cache_read_input_tokens` / `cache_creation_input_tokens` ; pre-warm avec `max_tokens: 0`.

**Pièges** :
- Un timestamp ou tout contenu par-requête dans le segment caché invalide le cache à chaque appel.
- Sous le minimum cacheable (512–4 096 tokens selon modèle), le caching est ignoré silencieusement, sans erreur.
- Changer `tool_choice` ou ajouter/retirer des images invalide le cache des messages ; modifier une définition de tool invalide tout (tools + system + messages).

### Agents long-horizon et chaining (Claude Fable 5, Opus 4.8)
**État courant** — Claude Fable 5 / Mythos 5 (frontier, runs autonomes de plusieurs heures à plusieurs jours) avec page de prompting dédiée dans la doc officielle.

**Pratiques clés**
- **Ancrer les rapports dans les tool results + vérifier par sous-agents à contexte frais** — Snippet anti-fabrication : « Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly. » (élimine quasi totalement les statuts fabriqués en test interne). Vérificateurs sous-agents à contexte frais > auto-critique. Encadrer la délégation via le snippet « Use subagents when tasks can run in parallel, require isolated context… work directly rather than delegating » : Opus 4.6 sur-délègue ; Opus 4.7/4.8 délèguent moins par défaut (à encourager) ; Fable 5 délègue de façon fiable. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) · [snippet délégation](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Workflows multi-fenêtres : prompt de setup, état sur disque, redémarrage à froid** — Première fenêtre dédiée (écrire les tests, créer init.sh), puis itérer sur une todo-list ; état en tests.json (structuré) + progress.txt (libre) + git ; au reset, contexte neuf avec « Review progress.txt, tests.json, and the git logs » plutôt que compaction ; verrouiller les tests (« It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality. »). [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓
- **Chaining explicite réservé à l'inspection d'intermédiaires** — Avec adaptive thinking et l'orchestration native de sous-agents, le multistep interne suffit le plus souvent ; chaîner en appels API séparés seulement pour logger, évaluer ou brancher sur des sorties intermédiaires ; pattern documenté : draft → review → refine. [source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) ✓

**Paramètres** :
- `send_to_user` tool (client-side, input_schema avec un champ message) pour délivrer du contenu verbatim en cours de run sans terminer le tour.
- Mémoire persistante : un fichier de leçon par sujet, résumé d'une ligne en tête, mise à jour plutôt que duplication.
- Snippet autonomie : « You are operating autonomously… End your turn only when the task is complete or you are blocked on input only the user can provide. »

**Pièges** :
- Compteur de tokens restants visible → arrêts prématurés et handoffs sur Fable 5 — masquer, ou ajouter « You have ample context remaining. Do not stop, summarize, or suggest a new session… ».
- Skills et prompts écrits pour les générations antérieures : souvent trop prescriptifs pour Fable 5, dégradent la qualité — auditer et élaguer à la migration.
- Actions destructives (git push --force, rm -rf, actions visibles par des tiers) : snippet de confirmation préalable documenté pour Opus 4.6 (« Balancing autonomy and safety ») ; pour Fable 5, la doc recommande plutôt « Pause for the user only when the work genuinely requires them: a destructive or irreversible action… ».
- Un `send_to_user` défini sans instruction d'élicitation dans le system prompt n'est quasiment jamais appelé.

**Non établi faute de source fiable** :
- Recommandations officielles détaillées propres à Claude Sonnet 5 (page dédiée « Prompting Claude Sonnet 5 » non consultée en intégralité).
- Bonnes pratiques officielles à jour pour l'application Claude.ai grand public (Projects, styles, instructions personnalisées).
- Benchmarks chiffrés publics de l'impact des balises XML seules (seul chiffre sourcé : « jusqu'à +30 % » lié au placement de la requête après les documents).
- Page autonome à jour « use-xml-tags » (l'URL historique redirige vers la page consolidée).
- Détails officiels du memory tool et du header `context-management-2025-06-27` (mentionnés uniquement par la source tierce PromptHub, 02/10/2025).
- Tarification input par modèle au-delà de l'exemple Opus 4.8 ($5/MTok base, $6,25 écriture 5 min, $0,50 lecture).

---

## Google Gemini
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — 12 affirmations sur 13 confirmées mot pour mot sur les sources citées, aucune hallucination de version ou de paramètre détectée.*

### Gemini API — prompt design général & system instructions (Gemini 3.x/3.5)
**État courant** — gemini-3.5-flash GA (annoncé à I/O 2026 le 19 mai ; contexte 1M tokens, 65k tokens de sortie), gemini-3.1-pro-preview, gemini-3.1-flash-lite ; gemini-2.5-flash et 2.5-pro encore servis. Gemini 3.5 Pro retardé : non sorti au 16 juillet 2026 (trois échéances manquées selon Bloomberg, modèle de base reconstruit). Nouvelle API « Interactions » (client.interactions.create) GA depuis juin 2026, generateContent maintenu en legacy.
**Pratiques clés**
- **Instructions directes et concises** — supprimer l'over-engineering hérité de Gemini 2.5 (chaînes de persuasion, répétitions) au profit d'un thinking_level adapté ; re-tester les prompts existants en migrant plutôt que les copier. [source](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) ✓
- **Laisser temperature/top_p/top_k aux défauts (temperature=1.0)** — les retirer carrément des requêtes sur 3.x/3.5 : une température basse cause boucles et dégradation documentées, surtout en maths/raisonnement. [source](https://ai.google.dev/gemini-api/docs/gemini-3) (page dépréciée ; référence à jour : [guide de migration 3.5](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5)) ✓
- **System instruction structurée en sections balisées** — `<role>`, `<constraints>`, `<output_format>` en tout début ; ordre recommandé : instructions système → contexte/données → exemples → tâche. La system instruction prime sur les requêtes utilisateur. [source](https://ai.google.dev/gemini-api/docs/prompting-strategies) ✓
- **Spécifier explicitement verbosité et persona** — « By default, Gemini 3 is less verbose » : pour un ton conversationnel ou détaillé, le demander dans la system instruction. [source](https://ai.google.dev/gemini-api/docs/gemini-3) (page dépréciée ; guidance à jour dans le guide 3.5) ✓
- **Toujours inclure des exemples few-shot au format strictement identique** — commencer petit (trop d'exemples → overfitting), mêmes délimiteurs et casse partout ; technique complémentaire : préfixe de complétion (amorcer le début de la sortie attendue). [source](https://ai.google.dev/gemini-api/docs/prompting-strategies) ✓
- **Agents : annoncer chaque tool call et persister** — décomposer en sous-objectifs, énoncer avant chaque appel le pourquoi et le résultat attendu, poursuivre jusqu'à résolution complète, interdire d'inventer en cas de données manquantes (demander l'info). Contenu vérifié en ligne quasi verbatim. [source](https://www.philschmid.de/gemini-3-prompt-practices) ✓
**Paramètres** : temperature=1.0 (défaut, ne pas modifier sur 3.x/3.5) ; retirer top_p/top_k · system_instruction (paramètre dédié de client.interactions.create) · ordre : system instruction → contexte → exemples → tâche.
**Pièges** : réutiliser tels quels des prompts sur-ingéniérés conçus pour 2.5 (doublon avec thinking_level, dégrade 3.x) · baisser la température pour « fiabiliser » (boucles documentées) · compter sur la verbosité par défaut (modèle volontairement laconique).

### Long context (fenêtre 1M tokens)
**État courant** — 1M tokens d'entrée sur gemini-3.5-flash, gemini-3.1-pro-preview et gemini-3.1-flash-lite (constaté juillet 2026).
**Pratiques clés**
- **Question/instruction À LA FIN du prompt** — donner d'abord documents/code, puis la consigne en dernier (« put your query at the end of the prompt », verbatim). La phrase-pont d'ancrage type « Based on the information above... » ne figure pas sur cette page : pratique issue de matériels annexes Google (ex. notebooks Vertex) / communautaire. [source](https://ai.google.dev/gemini-api/docs/long-context) ✓
- **Context caching pour tout contexte volumineux réutilisé** — optimisation n°1 recommandée : stocker les fichiers une fois et les référencer plutôt que renvoyer les tokens à chaque appel. [source](https://ai.google.dev/gemini-api/docs/long-context) ✓
**Paramètres** : fenêtre 1M tokens d'entrée (3.x/3.5), 65k en sortie (3.5 Flash) · context caching activable via l'API pour les préfixes répétés.
**Pièges** : récupération multi-needle (~100 extractions) dégrade la précision — préférer plusieurs requêtes séparées · latence du premier token croît avec la taille du contexte — ne pas envoyer 1M tokens quand 50k suffisent.

### Structured output (JSON mode) & function calling
**État courant** — Sur l'API Interactions (2026), le JSON structuré passe par response_format { type: "text", mime_type: "application/json", schema: {...} } ; l'ancienne forme generateContent (response_mime_type + response_schema) reste accessible via bascule sur la doc. Mode tool_choice « validated » en preview.
**Pratiques clés**
- **Contraindre le JSON par schéma via le paramètre dédié** — plutôt que « réponds en JSON » dans le prompt : JSON Schema avec types forts (integer, string, enum), bornes (minimum/maximum, minItems/maxItems), $ref récursifs ; remplir les champs description. Le JSON est syntaxiquement garanti mais « always validate values in your application ». La combinaison JSON mode + outils intégrés (Search, code execution, function calling) est documentée pour la série Gemini 3 avec label **Preview** (pas GA). [source](https://ai.google.dev/gemini-api/docs/structured-output) ✓
- **Function calling : descriptions précises, noms sans espaces, typage fort, 10–20 outils actifs max** — enums plutôt que strings libres pour les valeurs fermées ; tool_choice auto (défaut) / any / none / validated (preview) ; appels parallèles et compositionnels supportés nativement. [source](https://ai.google.dev/gemini-api/docs/function-calling) ✓
- **Sur 3.5 : chaque FunctionResponse porte l'id et le name du FunctionCall correspondant** — et le contenu multimodal retourné par un outil va DANS les parts de la function response ; instructions inline ajoutées au texte de la réponse (deux sauts de ligne), pas en parts séparées. Thought signatures gérées automatiquement par les SDK officiels. [source](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) ✓
- **Limiter les appels d'outils excessifs** — d'abord réduire thinking_level (medium → low → minimal), ensuite ajouter en system instruction : « You have a limited action budget of \<n\> tool calls ». [source](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) ✓
**Paramètres** : response_format: { type: "text", mime_type: "application/json", schema: {...} } (API Interactions 2026) · tool_choice: auto | any | none | validated (preview) · 10–20 outils actifs maximum.
**Pièges** : sous-ensemble d'OpenAPI/JSON Schema seulement — schémas très volumineux ou profondément imbriqués rejetés, surtout en tool_choice=any · confondre structured output (formater la réponse FINALE) et function calling (déclencher des ACTIONS) · migration 3.5 : oublier id/name sur les FunctionResponse ou mettre le multimodal hors des parts.

### Multimodalité (images, vidéo)
**État courant** — Docs image/video-understanding consultées en juillet 2026 (API Interactions) ; limites : 3 600 images/requête, vidéo jusqu'à 1h (résolution standard) ou 3h (basse), 10 vidéos max/requête depuis Gemini 2.5+.
**Pratiques clés**
- **Ordre des modalités : texte AVANT l'image (image unique) ; prompt APRÈS la vidéo (vidéo unique)** — asymétrie documentée telle quelle, suivre chaque doc pour sa modalité. [source](https://ai.google.dev/gemini-api/docs/image-understanding) ✓
- **Vidéo : timestamps MM:SS et échantillonnage 1 FPS** — poser les questions temporelles en MM:SS (ex. 01:15) ; monter media_resolution pour lire du texte fin à l'écran (coût tokens/latence accrus) ; tester media_resolution_high pour PDF/documents denses en migrant depuis 2.5. [source](https://ai.google.dev/gemini-api/docs/video-understanding) ✓
- **Détection/segmentation : box_2d normalisé 0–1000 et thinking minimal** — box_2d = [ymin, xmin, ymax, xmax] à redimensionner vers les dimensions réelles ; pour la segmentation, « disable thinking by setting the thinking level to 'minimal' ». Tokens image : 258 si ≤384px, sinon tuiles 768×768 de 258 tokens. [source](https://ai.google.dev/gemini-api/docs/image-understanding) ✓
**Paramètres** : media_resolution (media_resolution_high pour PDF/texte fin) · vidéo : 1 FPS défaut, MM:SS, max 10 vidéos/requête, 1h standard / 3h basse résolution · images : max 3 600/requête ; 258 tokens (≤384px) ou tuiles 768×768.
**Pièges** : scènes d'action rapide perdues à 1 FPS · oublier de re-normaliser box_2d (0–1000) vers les dimensions réelles · migration 2.5 → 3.x : tokens en HAUSSE pour les PDF, en BAISSE pour la vidéo — re-chiffrer les coûts.

### Modèles thinking (raisonnement)
**État courant** — thinking_level a remplacé thinking_budget sur 3.x/3.5. Défauts : medium sur gemini-3.5-flash (changé depuis high sur gemini-3-flash-preview), high sur gemini-3.1-pro, minimal sur gemini-3.1-flash-lite ; gemini-2.5-flash reste sur l'ancien système.
**Pratiques clés**
- **Adapter thinking_level (minimal/low/medium/high) à la complexité** — minimal/low : extraction, factuel, latence minimale ; medium (défaut) : tâches modérées ; high : maths type AIME, code avancé, planification multi-étapes. Remplacer thinking_budget par thinking_level en migrant et re-tester les prompts (défaut passé de high à medium sur 3.5 Flash). En stateless, renvoyer tous les blocs thought exactement tels que reçus ; en stateful, store: true + previous_interaction_id. [source](https://ai.google.dev/gemini-api/docs/thinking) ✓
- **Astuce latence : thinking_level LOW + instruction « think silently »** — la partie thinking_level low est confirmée, mais « think silently » n'apparaît plus sur la page thinking actuelle (figurait dans le guide Gemini 3 de nov. 2025 et chez [Schmid](https://www.philschmid.de/gemini-3-prompt-practices)). ⚠ à recouper
**Paramètres** : thinking_level: minimal | low | medium | high (remplace thinking_budget sur 3.x/3.5) · défauts : medium (3.5 Flash), high (3.1 Pro), minimal (3.1 Flash-Lite) · thinking_summaries: "auto" | "none" · stateful : store: true + previous_interaction_id.
**Pièges** : facturation = output_tokens + thought_tokens (réduire le niveau avant d'optimiser autre chose) · en stateless, omettre/altérer les blocs thought casse la continuité du raisonnement · compenser un raisonnement faible par du prompt engineering lourd au lieu de monter thinking_level (anti-pattern à abandonner).

**Non établi faute de source fiable** :
- Contenu détaillé du « Gemini 3 prompting guide » sur docs.cloud.google.com (Gemini Enterprise Agent Platform) — la page n'a renvoyé que sa navigation.
- Bonnes pratiques propres à l'interface AI Studio (organisation des prompts, comparaison de modèles dans l'UI).
- Spécifications et guidance de prompting pour Gemini 3.5 Pro — modèle non sorti au 16 juillet 2026, toute caractéristique serait spéculative.
- Statut actuel de propertyOrdering pour le structured output — plus mentionné dans la doc 2026 (API Interactions).
- Nombre optimal chiffré d'exemples few-shot — aucun chiffre officiel, seulement « toujours en inclure » + garde contre l'overfitting.
- Affiliation professionnelle exacte de Philipp Schmid à la date de son guide (contenu du guide vérifié, auteur non affilié de source fiable).
- Guidance officielle détaillée de prompting audio (speech) — seuls des conseils généraux trouvés.

---

## Mistral & open-weight (Llama, DeepSeek)
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — 13 des 14 pratiques confirmées à la citation près sur les sources primaires ; corrections appliquées (Magistral déprécié, attribution du piège few-shot R1 corrigée).*

### Mistral (API et modèles open-weight : Large 3, Medium 3.5, Ministral 3)
**État courant** — Mistral Large 3 (`mistral-large-2512`, déc. 2025, MoE 675B/41B actifs, Apache 2.0, contexte 256k) ; Mistral Medium 3.5 (29 avril 2026, modèle dense 128B, contexte 256k, `reasoning_effort` ajustable) qui absorbe Magistral et Devstral 2, désormais officiellement **dépréciés** ; famille Ministral 3 (3B/8B/14B, variantes base/instruct/reasoning).
**Pratiques clés**
- **Sections hiérarchiques Markdown/XML + rôle explicite** — ouvrir par « You are a \<role\>, your task is to \<task\> » ; diviser en sections claires (contexte, instructions, « Response Format », exemples), car Markdown/XML sont « Readable, Parsable, Familiar » ; rédiger comme pour quelqu'un sans aucun contexte préalable. [source](https://docs.mistral.ai/studio-api/conversations/chat-completion/prompting) ✓
- **Few-shot via tours user/assistant artificiels** — injecter les exemples comme échanges artificiels dans l'historique de conversation plutôt qu'en bloc unique ; améliore surtout la fidélité au format de sortie. En mono-message : section « Below are sample inputs and expected outputs ». [source](https://docs.mistral.ai/studio-api/conversations/chat-completion/prompting) ✓
- **Mesures objectives, jamais d'adjectifs flous** — la doc interdit « too long », « too short », « interesting » : donner des seuils chiffrés, et calculer les comptes (mots, caractères) côté client pour les passer en entrée au lieu de faire compter le modèle. [source](https://docs.mistral.ai/studio-api/conversations/chat-completion/prompting) ✓
- **Échelle verbale pour la notation** — « worded scale for better performance » (faible/moyen/élevé) plutôt que 1-10 ; prédéfinir les catégories dans le prompt et coupler au JSON mode pour le parsing. [source](https://docs.mistral.ai/studio-api/conversations/chat-completion/prompting) ✓
- **Reasoning (Medium 3.5) : effort high + ThinkChunk intacts** — `reasoning_effort="high"` recommandé pour l'agentique et le code ; ne jamais retirer les ThinkChunk des messages assistant avant de rejouer l'historique (« significantly degrades output quality ») ; `"none"` pour les tours simples. Magistral est déprécié sur cette page — utiliser Medium 3.5. [source](https://docs.mistral.ai/capabilities/reasoning) ✓
- **Self-hosting : mistral-common obligatoire** — depuis le tokenizer V2 (étendu V3/Tekken), le system prompt est préfixé au DERNIER message user ; `mistral_common` est « the ground truth for the tokenization process », les templates Jinja communautaires divergent (espaces autour des tokens spéciaux). [source](https://github.com/mistralai/cookbook/blob/main/concept-deep-dive/tokenization/chat_templates.md) ✓

**Paramètres** : Large 3 : temperature < 0.1 en production (model card HF), plus haut réservé au créatif · `reasoning_effort: "high" | "none"` (mistral-small-latest, mistral-medium-3-5) · vLLM Large 3 : `--tokenizer_mode mistral --config_format mistral --load_format mistral --max-model-len 262144`, tool use : `--enable-auto-tool-choice --tool-call-parser mistral` · JSON mode pour sortie structurée.
**Pièges** : faire compter mots/tokens au modèle (fournir les comptes en entrée) · instructions qui se chevauchent (préférer un arbre de décision explicite) · faire générer ce qu'on peut fournir en entrée (« Models are faster at ingesting tokens than generating them ») · template Jinja tiers en self-hosting (dégradation silencieuse) · supprimer les ThinkChunk de l'historique reasoning · utiliser Magistral (déprécié).

### Le Chat (assistant grand public Mistral)
**État courant** — Le Chat, avec mode « Think » (raisonnement pas à pas). À noter : « Vibe » est un produit officiel Mistral distinct (« Remote agents in Vibe. Powered by Mistral Medium 3.5 ») qui coexiste avec Le Chat — ce n'est pas un renommage de Le Chat.
**Pratiques clés**
- **Rôle + contexte + format + few-shot + Think + itération** — dire à Le Chat qui il doit « être » (« You are an expert travel guide »), préciser le format attendu (tableau, puces), donner plusieurs exemples de sortie voulue, activer Think pour les problèmes complexes, et itérer (reformuler, ajouter des détails) plutôt qu'attendre un résultat parfait du premier coup. [source](https://help.mistral.ai/en/articles/424366-get-better-answers-with-simple-prompting-techniques) ✓
**Paramètres** : mode « Think » activable pour forcer une décomposition pas à pas.
**Pièges** : attendre un résultat parfait au premier prompt — l'itération fait partie de la méthode officielle.

### Llama 4.x (Scout, Maverick — open-weight Meta)
**État courant** — Llama 4 Scout (17B actifs/16 experts, contexte annoncé 10M) et Maverick (17B actifs/128 experts, contexte 1M), sortis avril 2025, toujours les open-weight courants de Meta en juillet 2026 ; Behemoth non publié ; Meta a pivoté vers Muse Spark (closed-weight, avril 2026).
**Pratiques clés**
- **Template Llama 4 exact** — `<|begin_of_text|>`, `<|header_start|>role<|header_end|>`, fin de tour `<|eot|>` (différent de Llama 3) ; rôles system/user/assistant ; images via `<|image_start|>…<|patch|>…<|image|>…<|image_end|>` avec `<|tile_x_separator|>`/`<|tile_y_separator|>` pour les grandes images. [source](https://github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md) ✓
- **Tool calling zero-shot** — déclarer les fonctions dans le message system OU user ; pour un appel pur, exiger une sortie strictement de la forme `[func_name(param1=value1, param2=value2)]`, sans texte autour. [source](https://github.com/meta-llama/llama-models/blob/main/models/llama4/prompt_format.md) ✓
- **System prompt de référence Meta + limite de 5 images** — la model card fournit un system prompt anti-moralisateur (« You are an expert conversationalist... ») ; côté vision, « tested for image understanding up to 5 input images » — au-delà, non garanti. [source](https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct) ✓

**Paramètres** : contexte 10M (Scout) / 1M (Maverick) · max testé 5 images · format d'appel `[func_name(param1=value1)]` en zero-shot.
**Pièges** : réutiliser les tokens Llama 3 (`<|start_header_id|>`, `<|eot_id|>`) — dégradation silencieuse, vérifier le template du serveur d'inférence (vLLM/llama.cpp) · dépasser 5 images · system prompts très longs — la règle « 2-4 phrases focalisées » du guide Best Practices Llama API n'a été vue qu'en extrait de moteur de recherche [À VÉRIFIER].

### DeepSeek (V4 flash/pro via API ; R1 et V3.x en open-weight local)
**État courant** — API : `deepseek-v4-flash` (≈ 284B/13B actifs) et `deepseek-v4-pro` (≈ 1,6T/49B actifs), contexte 1M, thinking activé par défaut, interface Anthropic-compatible disponible ; alias hérités `deepseek-chat`/`deepseek-reasoner` retirés le 24 juillet 2026 (15:59 UTC). Open-weight : DeepSeek-R1 et V3.x restent les références locales.
**Pratiques clés**
- **Température par type de tâche** — barème officiel : 0.0 code/maths, 1.0 analyse de données, 1.3 conversation/traduction, 1.5 écriture créative (défaut 1.0). La restriction « mode non-thinking » est déduite de la doc thinking mode (qui y ignore temperature/top_p), pas explicite sur cette page. [source](https://api-docs.deepseek.com/quick_start/parameter_settings) ✓
- **Piloter le thinking mode explicitement** — `{"thinking": {"type": "enabled"|"disabled"}}` via extra_body ; `reasoning_effort` "high"|"max" (défaut high) ; en thinking mode, temperature/top_p/presence_penalty/frequency_penalty sont ignorés SANS erreur ; avec tool calls, inclure `reasoning_content` dans l'historique sous peine d'erreur 400. [source](https://api-docs.deepseek.com/guides/thinking_mode/) ✓
- **R1 open-weight : aucun system prompt** — tout dans le message user ; température 0.5-0.7 (0.6 recommandé) contre les répétitions sans fin ; forcer le début de réponse par « `<think>\n` » (le modèle saute parfois le raisonnement) ; maths : « put your final answer within `\boxed{}` ». [source](https://github.com/deepseek-ai/DeepSeek-R1) ✓
- **Contexte long : instructions en haut, question en bas** — et désactiver le reasoning (ou demander un thinking concis) pour les tours simples ; pour V4 Pro, Together relaie temperature=1.0 / top_p=1.0 car « Lower temperatures can collapse the reasoning trace ». [source](https://docs.together.ai/docs/prompting-deepseek-r1) ✓

**Paramètres** : temperature (non-thinking) : 0.0 code/maths, 1.0 data, 1.3 conversation/traduction, 1.5 créatif, défaut 1.0 · R1 local : 0.5-0.7 (0.6) · `thinking: {type: enabled|disabled}` via extra_body, `reasoning_effort: high|max` (défaut high) · alias deepseek-chat → v4-flash non-thinking, deepseek-reasoner → v4-flash thinking, valables jusqu'au 24/07/2026.
**Pièges** : system prompt sur R1 (dégrade — contrainte officielle du README) · few-shot sur R1 : dégrade souvent, le modèle mime le pattern des exemples au lieu de raisonner — à sourcer sur le README/papier DeepSeek-R1 (l'avertissement n'apparaît plus chez Together) ⚠ à recouper · régler temperature/top_p en thinking mode (ignoré silencieusement) · omettre `reasoning_content` dans un enchaînement de tool calls (erreur 400) · température trop basse sur les modèles reasoning (effondre la trace de raisonnement).

**Non établi faute de source fiable** :
- Valeurs de sampling officielles (temperature/top_p) pour Mistral Medium 3.5 : la page best-practices/sampling ne donne aucune valeur ; seule la model card de Large 3 chiffre (temp < 0.1 en production).
- Recommandations officielles Meta de température pour Llama 4 (pages llama.com/developer.meta.com illisibles au fetch) ; règle « system prompt 2-4 phrases » non confirmée sur la page source.
- Guide de prompting francophone officiel : version française de l'article Le Chat non récupérable ; aucun guide éditeur en français confirmé au-delà du centre d'aide.
- DeepSeek : répartition exacte des capacités entre v4-flash et v4-pro (la doc thinking n'exemplifie que v4-pro) et existence d'un éventuel « R2 » open-weight.
- Comportement réel du contexte 10M de Llama 4 Scout (seuil de dégradation pratique non documenté officiellement).

---

## Générateurs d'images
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — les 10 sources vérifiées existent toutes et confirment le rapport souvent mot pour mot, avec seulement deux corrections factuelles mineures côté Midjourney (appliquées ci-dessous).*

### Midjourney
**État courant** — V8.1, modèle par défaut sur midjourney.com depuis le 11 juin 2026 (V8 alpha : 17 mars 2026, V8.1 : 30 avril 2026). V7 reste sélectionnable et conserve --oref/--cref ; --q existe en V8 (mode --q 4 officiel) mais serait retiré en V8.1 [À VÉRIFIER — périmètre du retrait appuyé sur un guide tiers uniquement].

**Pratiques clés**
- **Écrire des phrases complètes décrivant la scène, pas des listes de mots-clés** — L'annonce officielle V8 demande des « totally new prompting styles » et récompense les prompts longs et spécifiques : abandonner le keyword spam (« 8k, masterpiece ») pour du langage naturel avec vocabulaire d'éclairage (« Rembrandt lighting », « golden hour ») et de photographie (85mm f/1.8). Hiérarchie : [SUJET] [DÉTAILS], [CONTEXTE], [STYLE], [TECHNIQUE] --paramètres. [source](https://updates.midjourney.com/v8-alpha/) ✓
- **Mettre le texte à faire apparaître entre guillemets** — Le rendu de texte V8 est nettement meilleur quand le texte voulu est entre guillemets ; ajouter --raw pour une interprétation plus littérale quand le contrôle prime. [source](https://updates.midjourney.com/v8-alpha/) ✓
- **Transférer un style avec --sref URL, dosé par --sw** — Le sref V7+ fuit beaucoup moins le sujet de la référence ; --sref random tire un style aléatoire ; les anciens codes V6 exigent --sv 4. La plage --sw 0-1000 (défaut 100) et --sv 7 par défaut ne sont confirmées que par un guide tiers [À VÉRIFIER]. [source](https://updates.midjourney.com/style-references-for-v7/) ✓
- **Cohérence personnage/objet : --oref URL + --ow 0-1000, en V7 uniquement** — Basculer --v 7 pour l'utiliser ; non supporté en V8.1, comme --cref. Workflow type : explorer en --draft (24 images, ~50 % du coût GPU), finaliser en --hd (2048px natif). Nota : l'absence de --oref/--cref en V8 repose sur ce guide tiers, pas sur l'annonce officielle. [source](https://blakecrosley.com/guides/midjourney) ✓
- **Calibrer --stylize selon l'objectif** — 0-100 photoréalisme fidèle au prompt, 100-300 équilibré, 300-1000 fortement esthétisé (l'annonce V8 encourage à pousser jusqu'à 1000). Combos relevés : photoréalisme « --raw --s 50-100 » ; artistique « --s 300-500 --chaos 50 » ; --exp zone utile ~10-25. [source](https://blakecrosley.com/guides/midjourney) ✓

**Paramètres** : --v 8.1 (défaut depuis le 11/06/2026) | --v 7 pour --oref/--cref ; --ar 1:1, 4:5, 5:4, 16:9, 21:9, 6:11, 9:16, 3:2, 2:3 (4:5/5:4/21:9/6:11 ajoutés en V8.1) ; --sref URL | random ; --sw 0-1000 (défaut 100) ; --sv 4/6/7 (défaut 7 [À VÉRIFIER]) ; --oref + --ow 0-1000 (V7) ; --s 0-1000 (défaut 100) ; --raw ; --exp 0-100 ; --chaos 0-100 ; --weird 0-3000 ; --iw 0-2 ; --hd (2K natif) / --sd / --draft ; --q 4 (V8, retiré en V8.1 [À VÉRIFIER]) ; --p profil (~40 images notées min., ~200 pour un profil stable — chiffres tiers [À VÉRIFIER]) ; --no pour les exclusions.

**Pièges** : prompts V6-style (tags + « 8k, masterpiece ») en V8 → résultats dégradés ; --oref/--cref en V8.1 → non supportés, repasser --v 7 ; --q 4 disponible en V8 mais retiré en V8.1 (ne pas dire « jamais porté en V8 ») ; anciens codes --sref V6 sans --sv 4 → style incorrect ; docs.midjourney.com bloque les fetchs automatisés (HTTP 403) — s'appuyer sur updates.midjourney.com.

### GPT-image (OpenAI / images ChatGPT)
**État courant** — gpt-image-2 est le dernier modèle API ; gpt-image-1.5, gpt-image-1 et gpt-image-1-mini restent maintenus en rétrocompatibilité (constaté le 2026-07-19 sur developers.openai.com).

**Pratiques clés**
- **Structurer dans un ordre constant : scène/contexte → sujet → détails clés → contraintes** — et annoncer l'usage visé (pub, mockup UI, infographie) pour fixer le niveau de polish. Pour les demandes complexes, préférer des segments courts étiquetés ou des sauts de ligne à un paragraphe unique ; prompts minimaux, JSON, instructions ou tags fonctionnent tous si l'intention est claire. [source](https://developers.openai.com/cookbook/examples/multimodal/image-gen-1.5-prompting_guide) ✓
- **Texte dans l'image : littéral entre guillemets (ou MAJUSCULES)** — spécifier typographie/taille/couleur/placement, exiger « verbatim, no extra characters », épeler lettre par lettre les mots difficiles ; pour les cas critiques, demander la préservation verbatim sans reflow du layout. [source](https://developers.openai.com/cookbook/examples/multimodal/image-gen-1.5-prompting_guide) ✓
- **Édition itérative : « change only X, keep everything else the same »** — petits changements par tour, lister les invariants (identité, géométrie, marque) et les re-spécifier à chaque itération. Compositing multi-images : référencer chaque entrée par index (« Image 1: … Image 2: … ») et décrire leurs interactions. [source](https://developers.openai.com/cookbook/examples/multimodal/image-gen-1.5-prompting_guide) ✓
- **Choisir l'API selon le workflow** — Image API pour le one-shot, Responses API pour l'édition conversationnelle multi-tours (File IDs, outil image_generation, révision automatique des prompts). En édition : input_fidelity=high pour préserver l'identité (gpt-image-1.5) ; gpt-image-2 traite toujours en haute fidélité ; masques via canal alpha. [source](https://developers.openai.com/api/docs/guides/image-generation) ✓

**Paramètres** : quality low/medium/high/auto (défaut auto ; high pour texte critique et infographies denses) ; size libre jusqu'à 3840px/côté en multiples de 16, ratio max 3:1 (ex. 1024x1024, 1536x1024, 2048x2048) ; input_fidelity high (gpt-image-1.5) ; output_format png (défaut)/jpeg/webp + output_compression 0-100 ; moderation auto/low ; stream via partial_images 0-3 ; n=X variantes.

**Pièges** : surcharger un seul prompt d'édition au lieu d'itérer → dérive et régressions ; oublier de re-spécifier les invariants à chaque tour → dérive identité/layout ; texte voulu sans guillemets → fautes et caractères parasites ; Image API pour un flux conversationnel multi-tours → utiliser la Responses API.

### Flux (Black Forest Labs — FLUX.1, FLUX.1 Kontext, FLUX.2)
**État courant** — Famille FLUX.1 (dev/schnell/pro), FLUX.1 Kontext (édition in-context) et FLUX.2, couvertes par le guide officiel docs.bfl.ml ; Kontext et FLUX.2 acceptent jusqu'à 10 images d'entrée (constaté le 2026-07-19).

**Pratiques clés**
- **Ne PAS utiliser de negative prompts** — Règle explicite du dépôt officiel BFL : « FLUX doesn't support them; describe what you want ». Prose naturelle et précise ; l'éclairage explicite a l'impact le plus fort sur la qualité ; les prompts en anglais sont les plus précis. [source](https://github.com/black-forest-labs/skills) ✓
- **Suivre le gabarit officiel** — [SUBJECT], [LOCATION], [STYLE], [CAMERA SETTINGS], [LIGHTING], [COLORS], [EFFECT], [ADDITIONAL ELEMENTS] — présenté comme point de départ flexible, la clarté prime. Commencer simple puis itérer un détail à la fois. [source](https://docs.bfl.ml/guides/prompting_unified_basics.md) ✓
- **Texte dans l'image en 3 temps** — texte exact entre guillemets + placement + typographie par attributs (« gold serif lettering »), placé tôt dans le prompt et gardé court. Les hex de marque fonctionnent pour le texte (« text 'ACME' in color #FF5733 ») — les codes hex sont documentés sur les pages style et édition (pas sur la page basics). [source](https://docs.bfl.ml/guides/prompting_unified_style.md) ✓
- **Édition (Kontext/FLUX.2) : verbes d'action + clause de préservation** — Change / Replace / Add / Remove / Turn into + « Change nothing else » / « while keeping the rest of the image unchanged ». Bannir le vague (« make it better »). Hex utilisables en édition (« Change the cow's white fur to the color #8bc4bb »). [source](https://docs.bfl.ml/guides/prompting_editing_single_reference.md) ✓

**Paramètres** : structure [SUBJECT]…[ADDITIONAL ELEMENTS] ; couleurs hex #RRGGBB + nom de couleur (génération et édition, pages style/édition) ; texte : guillemets + placement + attributs typographiques ; Kontext/FLUX.2 : jusqu'à 10 images d'entrée ; édition : verbes d'action + « Change nothing else ».

**Pièges** : negative prompt SD-style → non supporté, décrire positivement ; éditions vagues (« make it better ») → imprévisible ; éditer sans clause de préservation → le reste change aussi ; textes longs → fidélité des caractères qui s'effondre ; autre langue que l'anglais → supporté mais moins précis.

### Stable Diffusion 3.5 (Large / Large Turbo / Medium)
**État courant** — SD 3.5 Large (8B, ~1 mégapixel) reste la référence open-weights de Stability ; en 2026 Stability a pivoté son site vers Brand Studio et le guide de prompting SD3.5 officiel n'est plus en ligne [À VÉRIFIER pour toute pratique issue de ce guide via relais tiers].

**Pratiques clés**
- **Langage naturel descriptif + triple encodeur** — 2x CLIP (77 tokens) + T5-XXL : le modèle adhère mieux aux phrases qu'aux listes de tags. Exemples officiels : num_inference_steps=28, guidance_scale 3.5-4.5. Nota : max_sequence_length=512 est la valeur de l'exemple diffusers, le contexte T5 documenté à l'entraînement est 77/256 tokens — la pratique fonctionne mais distinguer les deux. [source](https://huggingface.co/stabilityai/stable-diffusion-3.5-large) ✓
- **negative_prompt en affinage léger uniquement** — le champ API (max 10 000 caractères) sert aux exclusions ciblées (élément, couleur, style), pas aux litanies « worst quality, bad anatomy » : « SD 3.5 has issues with CFG in negative prompts, so it must be watered down ». [source](https://sandner.art/stable-diffusion-35-large-what-you-need-to-know/) ✓
- **Structurer en 7 blocs ordonnés** — Style → Sujet/Action → Composition/Cadrage → Lumière/Couleur → Paramètres techniques → Texte (entre guillemets doubles) → Negative prompt. Structure attribuée au guide Stability retiré (relais MimicPC), non vérifiée indépendamment ; les paramètres API Bedrock cités en revanche sont confirmés à la lettre. [source](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-diffusion-3-5-large.html) ⚠ à recouper
- **Paramètres Bedrock exacts** — prompt et negative_prompt max 10 000 caractères ; aspect_ratio {16:9, 1:1, 21:9, 2:3, 3:2, 4:5, 5:4, 9:16, 9:21} défaut 1:1 ; seed 0-4294967294 (0 = aléatoire) ; mode t2i/i2i ; strength 0-1 ; output_format jpeg/png/webp. [source](https://docs.aws.amazon.com/bedrock/latest/userguide/model-parameters-diffusion-3-5-large.html) ✓

**Paramètres** : num_inference_steps=28 ; guidance_scale 3.5-4.5 (Large), Turbo : CFG ~1 et peu de steps ; max_sequence_length=512 (valeur exemple diffusers ; T5 entraîné 77/256 tokens) ; ~1 MP, dimensions divisibles par 64, zone optimale ~960-1152px pour le carré ; strength 0-1 (0 = préserve l'entrée, 1 = l'ignore).

**Pièges** : recycler les negative prompts fleuves SD1.5/SDXL → contre-productif ; CFG élevé type SDXL (7-12) → les exemples officiels tournent à 3.5-4.5 ; syntaxe de weighting (mot:1.2) non documentée pour SD3.5 → comportement non garanti ; dépasser ~1 MP ou dimensions non divisibles par 64 → artefacts.

### Ideogram
**État courant** — Ideogram 4.0, entraîné exclusivement sur des captions JSON structurées ; Magic Prompt convertit le texte libre vers ce format (constaté le 2026-07-19 sur docs.ideogram.ai).

**Pratiques clés**
- **Suivre l'ordre officiel en 8 éléments et rester sous ~150-160 mots (~200 tokens)** — « [Image summary]. [Main subject details], [Pose or action], [Secondary elements], [Setting & Background], [Lighting & Atmosphere], [Framing & Composition], [Technical enhancers] » ; au-delà de la limite, la fin est diluée ou ignorée — l'essentiel en tête ; tous les blocs ne sont pas obligatoires. [source](https://docs.ideogram.ai/using-ideogram/getting-started/prompting-guide/3-prompt-structure.md) ✓
- **Texte (point fort) : exact, entre guillemets, placé TÔT, court** — sur fond simple à fort contraste ; typographie décrite par attributs (bold, serif, script) plutôt que par nom de police ; multi-lignes : découper en sections avec placement explicite ; coquilles : peindre sur les caractères dans l'éditeur puis remixer à force d'image élevée (~90/100). [source](https://docs.ideogram.ai/using-ideogram/getting-started/prompting-guide/2-prompting-fundamentals/text-and-typography.md) ✓
- **Layouts de production : prompter directement en JSON** — high_level_description (fortement recommandé) + style_description (ordre imposé : photo → aesthetics, lighting, photo, medium, color_palette ; art → aesthetics, lighting, medium, art_style, color_palette ; 16 couleurs hex max pour l'image ET jusqu'à 5 par élément) + compositional_deconstruction (obligatoire : fond + éléments « obj »/« text », bounding boxes [y_min, x_min, y_max, x_max] normalisées 0-1000, origine en haut à gauche). Texte naturel pour l'exploration, JSON pour la répétabilité. [source](https://docs.ideogram.ai/using-ideogram/getting-started/prompting-guide/4.-json-prompting-ideogram-4.0.md) ✓

**Paramètres** : limite ~150-160 mots / ~200 tokens, essentiel en tête ; JSON : high_level_description + style_description (16 hex max image entière, 5 max par élément, ordre des champs imposé) + compositional_deconstruction obligatoire ; bounding boxes 0-1000 ; Magic Prompt : réécriture LLM texte libre → caption JSON.

**Pièges** : dépasser ~200 tokens → fin du prompt ignorée ; texte long ou scène chargée → fautes et distorsions du lettrage ; nommer une police précise au lieu d'attributs → non fiable ; envoyer du JSON avec Magic Prompt actif en réécriture → risque de réinterprétation, JSON direct pour la production.

**Non établi faute de source fiable** :
- Guide de prompting SD3.5 officiel de Stability : retiré du site (404, archive inaccessible) — recommandations appuyées sur la model card HF, la doc AWS Bedrock et des relais tiers.
- NON-support du keyword weighting (mot:1.2) en SD3.5 : évoqué dans des extraits de recherche, jamais confirmé sur une page consultée — traité en « non documenté ».
- docs.midjourney.com (Parameter List, Style Reference) : HTTP 403 — plages/défauts exacts de --sw, --ow, --exp, --iw issus d'updates.midjourney.com (partiel) et du guide tiers blakecrosley.com.
- Doc officielle Midjourney sur la plage et la sémantique fine de --exp en V8.1 (seule la disponibilité est confirmée officiellement).
- Guide de prompting officiel dédié à gpt-image-2 : le Cookbook consulté couvre gpt-image-1.5 (principes applicables à la famille selon OpenAI) ; le cookbook gpt-image-2 n'a pas pu être consulté (403).
- Paramètres API Ideogram (endpoint, rendering_speed, style_type, magic_prompt_option exacts) : pages de settings non consultées en détail.
- Chiffres officiels Midjourney sur la personnalisation V8 (nombre d'images à noter) : uniquement des chiffres tiers (40/200/2000).

---

## Outils de code IA
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~95 %) — les 11 sources citées existent et confirment le rapport, aucune affirmation réfutée ni obsolète ; seules quelques nuances de formulation ont été corrigées ci-dessous.*

### Cursor
**État courant** — Docs cursor.com/docs/rules consultées le 2026-07-19 : hiérarchie Team Rules → Project Rules → User Rules, support AGENTS.md y compris imbriqué dans les sous-dossiers. Pas de numéro de version produit publié dans les pages consultées.
**Pratiques clés**
- **Règles projet dans `.cursor/rules/*.mdc` avec frontmatter explicite** — Trois champs pilotent tout : `alwaysApply` (true = injecté à chaque session, ignore globs et description), `globs` (attachement auto quand un fichier correspondant est actif), `description` (l'agent décide de la pertinence). Quatre types actuels : Always Apply / Apply Intelligently / Apply to Specific Files / Apply Manually (@rule-name) — « Auto Attached » et « Agent Requested » sont les anciens libellés, mécanismes identiques. Les .md simples sont ignorés sauf AGENTS.md. [source](https://cursor.com/docs/rules) ✓
- **Garder chaque règle sous 500 lignes et scinder en règles composables** — « Keep rules under 500 lines » et « Split large rules into multiple, composable rules » figurent mot pour mot dans la doc. Écrire les règles comme une doc interne claire, avec exemples concrets et références de fichiers plutôt que du code copié. [source](https://cursor.com/docs/rules) ✓
**Paramètres** : `alwaysApply: true|false` ; `globs:` patterns séparés par virgules (ex. `src/**/*.tsx`) ; `description:` texte pour la sélection intelligente ; limite recommandée 500 lignes/règle ; précédence Team → Project → User, AGENTS.md imbriqué le plus spécifique gagne.
**Pièges** : `alwaysApply: true` fait ignorer globs et description ; ne pas dupliquer dans les règles des style guides ou du code déjà présents dans le repo ; un .md ordinaire dans `.cursor/rules` est ignoré (seuls .mdc et AGENTS.md comptent).

### GitHub Copilot
**État courant** — Docs consultées le 2026-07-19 : trois modes chat (Ask/Edit/Agent) plus coding agent cloud assignable sur issues ; custom instructions supportées par Chat, cloud agent et Code Review.
**Pratiques clés**
- **Structurer `.github/copilot-instructions.md` en 5 blocs** — (1) pitch projet, (2) tech stack avec notes d'usage, (3) coding guidelines (points-virgules, type hints, exigences de tests), (4) structure des dossiers, (5) scripts/outils/serveurs MCP disponibles. Ajouter commandes exactes de build/test/lint et workflows CI. Tenir en ~2 pages ; « un fichier imparfait a bien plus d'impact que rien du tout ». [source](https://github.blog/ai-and-ml/github-copilot/5-tips-for-writing-better-custom-instructions-for-copilot/) ✓
- **Instructions par chemin via `.github/instructions/NAME.instructions.md`** — Frontmatter `applyTo: "app/models/**/*.rb"` (plusieurs patterns séparés par virgules) et clé optionnelle `excludeAgent: "code-review"|"cloud-agent"`. AGENTS.md supporté n'importe où dans le repo (le plus proche gagne), CLAUDE.md/GEMINI.md à la racine. [source](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) ✓
- **Déléguer au coding agent des issues rédigées comme des prompts** — Problème clair + critères d'acceptation complets (tests unitaires requis ?) + fichiers à toucher. Test : « l'issue fonctionnerait-elle comme prompt IA ? ». Bonnes cibles : bug fixes, UI, tests, docs, dette technique ; à éviter : refactoring transverse, code sensible sécurité/prod. Pré-installer les dépendances via `copilot-setup-steps.yml` ; itérer via @copilot en commentaires groupés (« Start a review »). [source](https://docs.github.com/copilot/how-tos/agents/copilot-coding-agent/best-practices-for-using-copilot-to-work-on-tasks) ✓
- **Adapter le prompt au mode** — Ask pour comprendre sans modifier ; Edit pour des changements ciblés avec diff approuvé ; Agent pour un but multi-fichiers avec intention, périmètre et condition de fin, en boucle verify–fix–verify (tests/lint). Nuance vérifiée : l'agent mode applique les éditions de fichiers sans approbation préalable, mais les commandes potentiellement risquées restent présentées pour revue avant exécution. [source](https://github.blog/ai-and-ml/github-copilot/copilot-ask-edit-and-agent-modes-what-they-do-and-when-to-use-them/) ✓
**Paramètres** : `.github/copilot-instructions.md` (repo-wide) ; `applyTo` + `excludeAgent` (frontmatter des .instructions.md) ; `copilot-setup-steps.yml` ; longueur cible ~2 pages.
**Pièges** : instructions trop longues ou narratives (préférer des directives impératives courtes sous titres distincts) ; tâches floues, refactoring transverse ou code sensible assignés au coding agent ; commentaires de PR envoyés un par un (chaque envoi déclenche une itération — grouper via « Start a review »).

### v0 (Vercel)
**État courant** — Plateforme v0.app ; guide officiel « How to prompt v0 » du 2025-12-15, docs v0.app/docs consultées le 2026-07-19.
**Pratiques clés**
- **Template officiel en 3 entrées : product surface + contexte d'usage + contraintes** — « Build [product surface: components, data, actions]. Used by [who], in [what moment], to [what decision or outcome]. Constraints: platform/device, visual tone, layout assumptions ». Mesures Vercel : la spécificité du product surface a produit une génération 19 s plus rapide avec 152 lignes en moins ; les contraintes détaillées ont retiré 110 lignes superflues mais ont pris 10 s de PLUS — le gain de vitesse ne vaut que pour la spécificité. [source](https://vercel.com/blog/how-to-prompt-v0) ✓
- **Construire les pages complexes incrémentalement avec le prompt queuing** — « Large, complex applications are better built incrementally » : commencer par le cœur (ex. grille catalogue), puis panier, auth, etc., en progression section par section (header → hero → features). Le queuing empile jusqu'à 10 prompts exécutés séquentiellement. Spécifier aussi les états d'erreur. [source](https://v0.app/docs/text-prompting) ✓
- **Prompts pour la logique/structure, Design Mode pour le visuel** — Répartition officielle de l'itération : les retouches visuelles (couleurs, espacements) passent par Design Mode, plus rapide et sans consommer de générations. [source](https://vercel.com/blog/how-to-prompt-v0) ✓
**Paramètres** : template Build […] / Used by […] in […] to […] / Constraints ; queue de 10 prompts max ; Design Mode pour l'itération visuelle.
**Pièges** : demander « a dashboard » sans lister données, composants et actions (v0 invente ou oublie des features) ; prompter une application complète d'un coup au lieu d'itérer par bloc.

### Lovable
**État courant** — docs.lovable.dev/prompting/prompting-one et handbook officiel consultés le 2026-07-19 ; modes Default/Chat/Plan.
**Pratiques clés**
- **Structurer en niveaux : « training wheels », meta et reverse meta prompting** — Niveau 1 : sections markdown `# Context` / `## Task` / `### Guidelines` / `#### Constraints`. Meta prompting : « Rewrite this prompt to be more concise and detailed: '[prompt]' ». Reverse meta prompting après un debug : faire résumer les erreurs rencontrées et leur résolution en un prompt réutilisable. [source](https://lovable.dev/blog/2025-01-16-lovable-prompting-handbook) ✓
- **Knowledge Base avec 5 documents** — PRD, user flow, tech stack, guidelines frontend, structure backend : contexte persistant du projet, pour que chaque prompt soit interprété avec les mêmes fondations. [source](https://lovable.dev/blog/2025-01-16-lovable-prompting-handbook) ✓
- **UI composant par composant, contenu réel, descripteurs esthétiques précis** — Prompter par « modular parts » (hero, feature grid, testimonial slider, pricing table) ; bannir lorem ipsum (« Lovable does not work well with placeholder content ») ; poser tôt des buzzwords visuels (« minimal », « premium », « cinematic ») qui pilotent typo, espacements, ombres, radius, palette. Ajouter « Ask me any questions… » idéalement en Plan mode. [source](https://docs.lovable.dev/prompting/prompting-one) ✓
- **Déboguer en Chat Mode, chain-of-thought après 3 « Try to fix »** — Chat Mode pour analyser les erreurs AVANT toute modification ; si l'erreur persiste après trois « Try to fix », demander un raisonnement chain-of-thought pour la cause racine. [source](https://lovable.dev/blog/2025-01-16-lovable-prompting-handbook) ✓
**Paramètres** : structure # Context / ## Task / ### Guidelines / #### Constraints ; Knowledge Base 5 documents ; modes Default (build) / Chat (analyse) / Plan (clarification) ; seuil 3 échecs « Try to fix » → chain-of-thought.
**Pièges** : landing page entière en un prompt (« verser une recette dans un blender ») ; contenu placeholder qui masque structure et intention ; spammer « Try to fix » sans analyse.

### Bolt (StackBlitz)
**État courant** — support.bolt.new consulté le 2026-07-19 ; fonctionnalités Enhance prompt, Target/Lock file, Knowledge.
**Pratiques clés**
- **Premier prompt = blueprint : architecture et stack déclarées d'emblée** — Commencer par « the application architecture, including your choice of tools, frameworks » (ex. Astro, Tailwind, ShadCN) pour un scaffolding correct dès le départ, puis features une par une en petits prompts. Le bouton « Enhance prompt » (icône + du chatbox) pose des questions guidées et regénère un prompt éditable. En dépannage : « Make one change. Check that the change works. Move on to the next. » [source](https://support.bolt.new/best-practices/prompting-effectively) ✓
- **Scoper avec Target file / Lock file, persister le contexte dans Knowledge** — Clic droit en vue Code → « Target file » (limiter les changements à ce fichier), « Lock file »/« Lock all » (exclure de toute modification) ; sélection de code + « Ask Bolt » pour cibler une zone. « Knowledge » (compte/projet/équipe) pour des instructions persistantes inter-sessions. Batcher les petites instructions en un message et purger régulièrement le contexte (Clear context) pour la gestion du contexte — l'argument « économie de crédits » n'est pas formulé tel quel sur la page officielle. [source](https://support.bolt.new/best-practices/prompting-effectively) ✓
**Paramètres** : Enhance prompt (icône +) ; Target file / Lock file / Lock all (clic droit) ; Knowledge compte/projet/équipe ; batching d'instructions simples en un message.
**Pièges** : laisser Bolt choisir la stack (le scaffolding initial est difficile à changer après) ; ne pas verrouiller les fichiers finis (Bolt peut les modifier ensuite) ; contexte jamais purgé (dérive du modèle sur les longues sessions).

### Windsurf (Cognition/Devin)
**État courant** — docs.windsurf.com redirige (307, constaté le 2026-07-19) vers docs.devin.ai — produit intégré à Cognition ; `.devin/rules/` est le chemin préféré, `.windsurf/rules/` et `.windsurfrules` sont legacy.
**Pratiques clés**
- **Règles workspace dans `.devin/rules/*.md` avec frontmatter `trigger` et limites de caractères** — Quatre modes exacts : `always_on` (injecté dans le system prompt à chaque message), `model_decision` (seule la description est visible, contenu chargé si Cascade le juge pertinent), `glob` (appliqué sur fichiers matchant `globs`), `manual` (@rule-name). Limites dures : 12 000 caractères par règle workspace, 6 000 pour les règles globales (`~/.codeium/windsurf/memories/global_rules.md`). [source](https://docs.devin.ai/desktop/cascade/memories) ✓
- **Règles en bullet points/markdown avec tags XML, promouvoir les Memories durables en Rules ou AGENTS.md** — La doc recommande « bullet points, numbered lists, and markdown », tags XML pour grouper, règles « simple, concise, and specific ». Les Memories auto-générées restent locales à la machine et au workspace : pour un savoir partagé en équipe, l'écrire dans une Rule versionnée ou AGENTS.md. [source](https://docs.devin.ai/desktop/cascade/memories) ✓
**Paramètres** : `trigger: always_on | model_decision | glob | manual` ; `globs:` (avec trigger glob) ; limites 12 000 car./règle workspace et 6 000 car. règles globales ; chemins `.devin/rules/` (préféré), `.windsurf/rules/` (fallback), `.windsurfrules` (legacy racine).
**Pièges** : s'appuyer sur les Memories auto-générées pour du savoir d'équipe (locales, non partagées) ; règle dépassant la limite de caractères (contenu tronqué/ignoré — scinder) ; guides tiers pointant encore `.windsurfrules` alors que la doc officielle vit sous docs.devin.ai.

**Non établi faute de source fiable** :
- Prompting des complétions inline (Tab) chez Cursor : aucune page officielle dédiée trouvée.
- Billet officiel Bolt « Prompting like a pro: 8 tips » : renvoie une 404, seules les pages support.bolt.new ont pu être vérifiées.
- Numéros de version produits précis (Cursor x.y, Bolt, Lovable, v0) : non publiés dans les pages consultées.
- Framework « C.L.E.A.R. » attribué à Lovable par des tiers : absent des docs officielles actuelles.
- Limite de ~1000 lignes pour copilot-instructions.md : source tierce uniquement, docs GitHub parlent de « ~2 pages ».
- Bonnes pratiques officielles v0 sur images/maquettes et Design Systems : mentionnées mais pages dédiées non consultées en détail.

---

## Agents autonomes et prompting agentique
*Recherche : 2026-07-19 · Vérification adversariale : fiabilité très élevée (~98 %) — 13 groupes de pratiques vérifiés en source primaire, aucune affirmation réfutée ni obsolète, seules corrections de formulation (SHOULD vs obligation) et d'attribution de source.*

### Claude Code (CLAUDE.md, boucle agentique)
**État courant** — Docs officielles au 2026-07-19 : plan mode, auto mode, /goal, Stop hooks, /rewind, agent teams, headless `claude -p`. Modèles cités : Claude Sonnet 5, Opus 4.8, Sonnet 4.6.
**Pratiques clés**
- **CLAUDE.md court et élagué** — N'inclure que ce que Claude ne peut pas deviner en lisant le code ; tester chaque ligne par « sa suppression causerait-elle des erreurs ? ». Inclure commandes bash non devinables, style divergeant des défauts, gotchas ; exclure conventions standard et descriptions fichier par fichier (un fichier trop long fait ignorer les vraies règles). Renforcer avec IMPORTANT / YOU MUST, imports via `@path/to/import`, générer avec /init, vérifier avec /context ; le savoir occasionnel va dans un skill. [source](https://code.claude.com/docs/en/best-practices) ✓
- **Critère de fin exécutable** — Fermer la boucle avec un check exécutable (tests, build, screenshot à comparer), gradation de dureté : prompt → /goal → Stop hook (bloque la fin de tour jusqu'au vert, override après 8 blocages consécutifs) → subagent relecteur en contexte frais, limité au diff et aux critères, consigné à ne signaler que les écarts de correction. Exiger des preuves plutôt qu'une affirmation de succès. [source](https://code.claude.com/docs/en/best-practices) ✓
- **Explore → Plan → Code → Commit** — Plan mode pour lire/planifier sans modifier ; skipper le plan si le diff se décrit en une phrase. Contexte = ressource n°1 : /clear entre tâches non liées, `/compact <instructions>` ciblé, subagents d'investigation qui rapportent des résumés ; après 2 corrections ratées, /clear et réécrire un meilleur prompt initial. [source](https://code.claude.com/docs/en/best-practices) ✓
**Paramètres** :
- `claude -p "prompt" --output-format stream-json --verbose` pour le headless
- `--allowedTools "Edit,Bash(git commit *)"` pour scoper un fan-out batch
- `claude --permission-mode auto` pour exécution non supervisée avec classifieur
- `/compact <instructions>` pour compaction ciblée
**Pièges** : session fourre-tout (→ /clear entre tâches) ; CLAUDE.md sur-spécifié (les vraies règles se perdent dans le bruit) ; investigation non scopée qui remplit le contexte de centaines de fichiers ; accepter du code plausible sans vérification exécutable ; chasser tous les findings d'un reviewer → sur-ingénierie.

### Agent Skills (SKILL.md, Anthropic)
**État courant** — Doc plateforme Claude, consultée le 2026-07-19.
**Pratiques clés**
- **Description à la 3e personne, spécifique** — Couvrir « ce que ça fait » ET « quand l'utiliser » avec les termes déclencheurs ; jamais « I can help you... » (injectée dans le system prompt, la 1re personne casse la découverte). C'est le seul signal de sélection parmi 100+ skills. `name` : max 64 caractères, minuscules/chiffres/tirets, gérondif recommandé (processing-pdfs) ; `description` : max 1024 caractères. [source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) ✓
- **Progressive disclosure** — SKILL.md sous 500 lignes ; fichiers de référence liés à UN seul niveau de profondeur (Claude prévisualise avec `head -100` et lit incomplet si imbriqué) ; table des matières pour tout fichier >100 lignes ; préciser l'intention : « Run analyze_form.py » (exécuter) vs « See analyze_form.py » (lire) ; construire 3 évaluations AVANT de rédiger la doc. [source](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) ✓
**Paramètres** :
- SKILL.md < 500 lignes ; `description` ≤ 1024 caractères, `name` ≤ 64
- Degrés de liberté : instructions texte (haute), pseudocode (moyenne), script exact sans flags additionnels (basse)
- Outils MCP référencés en nom qualifié `ServerName:tool_name`
**Pièges** : chemins Windows avec backslash (casse sur Unix) ; trop d'options de librairies au lieu d'un défaut + échappatoire ; infos périmables (dates, versions) hors section « old patterns » ; constantes magiques non justifiées dans les scripts ; terminologie incohérente (field/box/element).

### Définition de tools (API Anthropic + spec MCP)
**État courant** — Spec MCP 2025-11-25 (courante au 2026-07-19) ; article Anthropic « Writing effective tools for agents ».
**Pratiques clés**
- **Namespacer et consolider** — Préfixes par service/ressource (asana_projects_search, asana_users_search) ; un tool par workflow plutôt que des wrappers d'API atomiques (schedule_event au lieu de list_users + list_events + create_event ; get_customer_context au lieu de 3 appels). Paramètres non ambigus (user_id plutôt que user) ; décrire chaque tool « comme à un nouveau collègue » : contexte implicite, formats spéciaux, relations entre ressources. [source](https://www.anthropic.com/engineering/writing-tools-for-agents) ✓
- **Optimiser les retours pour le contexte** — Paramètre enum `response_format` concise/detailed (mesuré : 206 vs 72 tokens, ~2/3 de contexte économisé) ; pagination/troncature avec défauts sensés (Claude Code tronque les réponses de tools à 25 000 tokens par défaut) ; erreurs actionnables proposant des améliorations concrètes, pas des stack traces ; évaluer sur des tâches réalistes multi-appels. [source](https://www.anthropic.com/engineering/writing-tools-for-agents) ✓
- **Côté MCP** — `inputSchema` JSON Schema valide obligatoire (MUST) ; `outputSchema` pour valider le structuredContent ; erreurs d'exécution en `isError:true` avec feedback auto-corrigeable (ex. « Invalid departure date: must be in the future »), distinctes des erreurs protocole JSON-RPC. Noms de tools : recommandations SHOULD (pas des MUST) — 1-128 caractères, uniques par serveur, sans espaces ; caractères recommandés A-Z, a-z, 0-9, underscore, tiret, point. Human-in-the-loop : SHOULD afficher les inputs et confirmer les opérations sensibles ; annotations (readOnly/destructive) non fiables sauf serveur de confiance. [source](https://modelcontextprotocol.io/specification/2025-11-25/server/tools) ✓
**Paramètres** :
- `response_format`: enum concise | detailed
- Limite Claude Code : 25 000 tokens par réponse de tool
- MCP : `isError:true` + message actionnable ; `outputSchema` aligné sur structuredContent
**Pièges** : wrappers 1:1 d'endpoints API sans valeur agentique ; serveurs MCP fourre-tout (kitchen-sink) ; UUID techniques au lieu de champs lisibles par le modèle ; faire confiance aux annotations de serveurs non vérifiés.

### Context engineering pour boucles agentiques (Anthropic)
**État courant** — Article Anthropic « Effective context engineering for AI agents », référence courante au 2026-07-19.
**Pratiques clés**
- **Le plus petit ensemble de tokens à haut signal** — System prompt à la bonne altitude (ni hardcoding fragile de cas particuliers, ni directives vagues), organisé en sections balisées XML ou Markdown (background_information, instructions, tool guidance, output description) ; few-shot divers et canonique, pas une liste de cas limites ; just-in-time retrieval : identifiants légers (chemins, requêtes) chargés dynamiquement plutôt que préchargés. [source](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) ✓
- **Long-horizon : compaction + notes + sous-agents** — Compaction : conserver décisions architecturales, bugs non résolus, détails d'implémentation ; jeter les sorties de tools redondantes (variante légère : clearing des tool results). Notes structurées type NOTES.md / to-do réinjectées au besoin. Sous-agents d'exploration profonde isolée retournant des résumés condensés de 1000-2000 tokens au coordinateur. [source](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) ✓
**Paramètres** :
- Résumés de sous-agents : 1000-2000 tokens
- Compaction : maximiser d'abord le rappel, puis itérer sur la précision
**Pièges** : précharger toutes les données au lieu du just-in-time retrieval ; compacter en perdant décisions et bugs ouverts (ne jeter que le redondant).

### Multi-agents (orchestrateur-workers, Anthropic)
**État courant** — Article Anthropic « How we built our multi-agent research system » (système Research en production), consulté le 2026-07-19. Résultat confirmé en source primaire par le vérificateur : le multi-agent (lead Opus 4 + subagents Sonnet 4) surpasse le single-agent Opus 4 de 90,2 % sur l'éval interne.
**Pratiques clés**
- **Délégation à 4 éléments** — Chaque brief de sous-agent contient : objectif précis, format de sortie attendu, outils/sources recommandés, limites explicites de la tâche (frontières négatives : « ne cherche pas X, c'est le travail d'un autre sous-agent »). Les instructions courtes type « research the semiconductor shortage » provoquent duplication et lacunes. Évaluer le système sur l'état final plutôt que sur chaque étape. [source](https://www.anthropic.com/engineering/multi-agent-research-system) ✓
- **Règles d'échelle explicites + parallélisme forcé** — Barème dans le prompt orchestrateur : simple = 1 agent / 3-10 tool calls ; comparaison = 2-4 sous-agents / 10-15 calls chacun ; complexe = 10+ sous-agents à responsabilités découpées. Parallélisme à 2 niveaux (3-5 sous-agents simultanés ; 3+ tool calls parallèles par sous-agent) : jusqu'à -90 % de temps de recherche. Extended thinking pour la planification du lead, thinking intercalé après les résultats d'outils chez les workers. [source](https://www.anthropic.com/engineering/multi-agent-research-system) ✓
**Paramètres** :
- Simple : 1 agent / 3-10 tool calls ; comparaison : 2-4 subagents / 10-15 calls ; complexe : 10+ subagents
- 3-5 sous-agents en parallèle ; 3+ tool calls parallèles par sous-agent
**Pièges** : délégation vague (duplication et trous de couverture) ; sur-investissement (dizaines de sous-agents pour une requête simple) ; évaluer étape par étape des chemins pourtant valides (préférer end-state + checkpoints discrets).

### Agents OpenAI (GPT-5.x) et AGENTS.md
**État courant** — Guides de prompting GPT-5 et GPT-5.2 (cookbook OpenAI) : reasoning_effort par défaut « none » pour GPT-5.1 ET GPT-5.2 (GPT-5 avait « medium »). AGENTS.md : 60 000+ projets, géré par l'Agentic AI Foundation (Linux Foundation).
**Pratiques clés**
- **Calibrer l'agentivité par blocs explicites** — Bloc de persistance (verbatim GPT-5 : « You are an agent - please keep going until the user's query is completely resolved, before ending your turn and yielding back to the user ») + documenter les hypothèses au lieu de demander confirmation ; pour réduire l'avidité : early stop criteria (« you can name exact content to change », convergence ~70 % des hits) et budget explicite (« absolute maximum of 2 tool calls » pour les cas à faible latence). [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide) ✓
- **GPT-5.2 : effort minimal par défaut, structure XML, compaction** — reasoning_effort « none » par défaut, n'augmenter qu'après régression mesurée en éval ; blocs XML (output_verbosity_spec, design_and_scope_constraints, uncertainty_and_ambiguity) ; user updates de 1-2 phrases aux phases majeures, chacune avec un outcome concret ; `POST /v1/responses/compact` pour compacter après les phases tool-heavy. [source](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide) ✓
- **AGENTS.md : documenter le repo pour les agents** — Commandes de build/test exactes, style de code, sécurité, conventions de commit/PR ; fichiers imbriqués par sous-projet dans les monorepos (l'agent lit le plus proche de la zone de travail, le plus spécifique gagne) ; les instructions explicites du chat priment toujours sur le fichier. Lu par Codex, Jules, Cursor, Factory, entre autres. Traiter comme documentation vivante. [source](https://agents.md/) ✓
**Paramètres** :
- reasoning_effort: none (défaut GPT-5.1 et GPT-5.2 ; GPT-5 : medium) → augmenter après éval seulement
- Migration : GPT-4o/4.1 → GPT-5.2 reasoning_effort none ; GPT-5.1 minimal → none
- `POST /v1/responses/compact` pour compaction en cours de session
- Contexte long : au-delà de ~10k tokens, outline interne + re-position des contraintes avant de répondre
**Pièges** : instructions contradictoires (gaspillent des tokens de raisonnement à réconcilier) ; narration des appels d'outils routiniers dans les user updates ; sur-verbosité par défaut (fixer une spec de verbosité explicite).

### Computer use (Claude)
**État courant** — Beta header `computer-use-2025-11-24` (Claude Sonnet 5, Opus 4.8/4.7/4.6, Sonnet 4.6, Opus 4.5) ; `computer-use-2025-01-24` pour les modèles antérieurs (Sonnet 4.5, Haiku 4.5, Opus 4.1...). Doc consultée le 2026-07-19.
**Pratiques clés**
- **Vérification visuelle explicite après chaque étape** — Verbatim doc : « After each step, take a screenshot and carefully evaluate if you have achieved the right outcome. Explicitly show your thinking: "I have evaluated step X..." If not correct, try again. Only when you confirm a step was executed correctly should you move on to the next one. » Préférer des tâches simples et bien définies ; raccourcis clavier pour dropdowns/scrollbars ; inclure screenshots + tool calls d'exécutions réussies dans le prompt pour les tâches répétables ; consignes récurrentes dans le system prompt ; credentials en balises XML dédiées (risque d'injection de prompt accru, à isoler) ; prompts positionnels (« le bouton Submit bleu en bas à droite ») quand des éléments se ressemblent. [source](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool) ✓
**Paramètres** :
- Beta header `computer-use-2025-11-24` ; `enable_zoom: true` pour inspecter les petits textes
- Screenshots ≈ 1000-1800 tokens d'input chacun ; caching : 1 breakpoint cache_control après system prompt + tools, jusqu'à 3 sur les derniers tool_result, avancés à chaque tour
- Overhead system prompt du beta : 466-499 tokens
**Pièges** : Claude suppose le résultat d'une action sans vérifier (d'où le prompt de vérification obligatoire) ; les instructions contenues dans pages/images peuvent surpasser les vôtres (prompt injection — isoler données et actions sensibles) ; dropdowns/scrollbars à la souris peu fiables (raccourcis clavier) ; sessions longues : vérification end-to-end dès le début de session, pas seulement après implémentation.

**Non établi faute de source fiable** :
- Patterns multi-agents officiels OpenAI (manager pattern, handoffs, décentralisé — « A practical guide to building agents », doc Agents SDK) : non consultés directement dans cette session.
- Guide officiel de prompting agentique spécifique aux tout derniers modèles Claude (Sonnet 5 / Opus 4.8) au-delà des docs outils : la page « prompt engineering » plateforme n'a pas été fetchée.
- Guide officiel modelcontextprotocol.io dédié à la rédaction des descriptions de tools : les pages « best practices » trouvées sont communautaires, non officielles.
- Contenu détaillé du billet Anthropic « Effective harnesses for long-running agents » (référencé par la doc computer use) : non consulté.
