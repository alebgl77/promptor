# Agent Skills (SKILL.md) — bonnes pratiques de prompting
Dernière mise à jour : 2026-07-20 · Origine : recherche sourcée + vérification adversariale

## Pratiques
- Rédiger la description à la 3e personne, jamais « I can help you... » : elle est injectée dans le system prompt et la 1re personne casse la découverte.
- La description couvre « ce que ça fait » ET « quand l'utiliser », avec les termes déclencheurs : c'est le seul signal de sélection parmi 100+ skills.
- name en forme gérondive recommandée (processing-pdfs), minuscules/chiffres/tirets.
- Progressive disclosure : le corps de SKILL.md ne se charge que quand le skill devient pertinent ; les fichiers annexes seulement au besoin (zéro coût contexte avant lecture).
- Fichiers de référence liés à UN seul niveau de profondeur : Claude prévisualise avec head -100 et lit incomplet si imbriqué.
- Table des matières pour tout fichier de référence de plus de 100 lignes.
- Préciser l'intention sur les scripts : « Run analyze_form.py » (exécuter) vs « See analyze_form.py » (lire).
- Construire 3 scénarios d'évaluation AVANT de rédiger la doc.
- Calibrer les degrés de liberté : instructions texte (haute), pseudocode (moyenne), script exact sans flags additionnels (basse).

## Paramètres
- SKILL.md : corps < 500 lignes.
- name : ≤ 64 caractères (minuscules, chiffres, tirets).
- description : ≤ 1024 caractères.
- Outils MCP référencés en nom qualifié ServerName:tool_name.

## Pièges
- Chemins Windows avec backslash : cassent sur Unix.
- Offrir trop d'options de librairies au lieu d'un défaut + échappatoire.
- Infos périmables (dates, versions) dans le corps hors section « old patterns ».
- Constantes magiques non justifiées dans les scripts.
- Terminologie incohérente (mélanger field/box/element).
