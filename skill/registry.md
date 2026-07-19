# Promptor — Registre des livraisons (fichier d'état UNIQUE)

Une ligne par cycle livré. Ce registre alimente `--again`, `--from`, `--list`, le debrief,
la clinique `retour` et l'auto-recalibration. Ne jamais créer de fichier d'état parallèle.

Statuts : livré → testé-OK / à-retoucher / corrigé (mis à jour au debrief ou en Phase R).

| date | slug | famille/outil | objectif (court) | calibrage compact | itér. | note | statut | chemin |
|---|---|---|---|---|---|---|---|---|
