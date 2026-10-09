# VerifSiret

Est-ce que mes SIRET et n° de TVA intracom sont corrects ?

Cet outil vérifie, à partir de la base SIRENE ([API Recherche d'entreprises](https://recherche-entreprises.api.gouv.fr), données data.gouv.fr), que vos SIREN, SIRET et n° de TVA intracommunautaire sont corrects.

## Utilisation

1. Ouvrez `index.html` dans un navigateur.
2. Collez vos données depuis Excel (avec ou sans ligne d'en-têtes). Les colonnes sont détectées automatiquement (nom / raison sociale, code postal, SIRET, SIREN, TVA) et peuvent être corrigées à la main.
3. Lancez l'analyse, puis filtrez (pastilles du résumé, ouverts/fermés, anomalies) et exportez le résultat vers Excel (.xlsx, lignes affichées).

## Quota de l'API

Aucune limite de débit n'est imposée a priori : 4 lignes sont traitées en parallèle, les SIRET identiques ne sont interrogés qu'une fois. Si l'API répond **HTTP 429** (quota dépassé), toutes les analyses se mettent en pause (`Retry-After` si fourni, sinon 2 s, doublées à chaque nouveau refus, jusqu'à 60 s) puis reprennent ; après 8 refus consécutifs la ligne passe en « Erreur réseau ». La durée affichée en en-tête est une estimation (≈ 5 requêtes/seconde). Les constantes `CONCURRENCY`, `MAX_RETRIES` et `MAX_BACKOFF_MS` sont en tête de la section API du script.

## Fonctionnement

1. **Validation** des identifiants saisis : SIRET (14 chiffres, clé de Luhn, exception La Poste), SIREN (9 chiffres, Luhn), TVA française (format et clé `(12 + 3 × (SIREN mod 97)) mod 97`). Les zéros initiaux perdus par Excel sont restaurés. Les TVA des autres pays ne sont pas vérifiées.
2. **Reconstitution** de ce qui manque, par ordre de priorité SIRET > SIREN > TVA : SIREN depuis un SIRET ou une TVA valides, TVA française calculée depuis le SIREN. Une TVA invalide est corrigée depuis le SIREN retenu.
3. **Contrôle dans SIRENE** du SIRET : existence, établissement fermé / entreprise cessée, cohérence avec le nom saisi.
4. **Recherche** si aucun SIRET n'est retenu (absent, invalide ou introuvable), avec ce que l'on sait : SIREN (saisi ou reconstitué), puis nom et code postal. Les propositions s'affichent dans la dernière colonne ; **Choisir** en retient une. Les colonnes ignorées (« — ignorer — ») sont rappelées sous le nom saisi pour les lignes qui ont des propositions, afin de les comparer aux données trouvées par l'API.
5. **Export Excel** : valeurs saisies (les colonnes ignorées sont reprises telles quelles après le code postal), valeurs retenues (SIREN, SIRET, TVA) avec leur origine (saisi, recalculé, choisi), statuts, état ouvert/fermé et propositions. La proposition choisie est celle qui est exportée. La colonne « Résultat ligne » indique « Corrigé » quand une proposition a été choisie ou quand un identifiant saisi diffère de celui retenu (un identifiant simplement complété n'est pas une correction).

## Stack technique

Application JS en fichier seul (`index.html`), sans dépendance ni serveur. Seuls les SIRET, noms et codes postaux nécessaires sont envoyés à l'API publique.
