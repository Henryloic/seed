# Normalisation des mesures bancs

Page HTML autonome (aucune installation, aucun serveur) pour lire les fichiers de mesure des
bancs d'essai, normaliser les noms de voies, les corriger, les synchroniser avec des sources
externes et exporter un fichier `.seed`.

Ouvrir [index.html](index.html) dans un navigateur. Seule dépendance réseau :
Plotly, chargé depuis un CDN pour l'affichage des courbes.

## Sources de données

### Fichiers de mesure (un seul à la fois)

| Banc | Extension | Séparateur | Entête | Temps |
|---|---|---|---|---|
| BPERF | `.M00x` | tabulation | 4 lignes | voie `Timer for free recording` |
| BRBC3 | `.csv` | virgule | détection de la ligne `TEMPS` | colonne `TEMPS` (s) |
| Soufflerie froide | `.csv` / `.txt` | point-virgule | 1 ligne | colonne 1 (s) |
| SEED | `.seed` | tabulation | 1 ligne | colonne `TEMPS` (s) |

Le fichier `.seed` est le format produit par l'outil : le relire permet de reprendre un travail
sans refaire les corrections.

### Sources externes à synchroniser

| Source | Extension | Séparateur | Entête | Temps |
|---|---|---|---|---|
| CAN | `.csv` | `,` ou `;` détecté | 1 ligne | alignement par index |
| CarScanner | `.csv` | virgule | 1 ligne | `HH:mm:ss.SSS` |
| OBD Facile | `.txt` | point-virgule | 3 lignes | secondes, virgule décimale |
| Diagra | `.csv` | point-virgule | 3 lignes | millisecondes |

Le pas de temps de la source est comparé à celui du fichier de mesure ; s'il diffère de plus de
0,1 %, les voies sont rééchantillonnées par interpolation linéaire sur le pas cible.

## Renommage des voies

Les noms bruts sont traduits vers les noms normalisés (`VITESSE_BANC`, `EFFORT_VEH_TOTAL`,
`DISTANCE_BANC_AV`…) à partir de la table reprise de `data_labelling.xlsx`, onglet `LABEL`,
avec une colonne par banc. La table est recopiée dans `CORRESPONDANCES_VOIES` au début de
[script.js](script.js) : c'est là qu'il faut intervenir si le classeur évolue.

## Traitements disponibles

- **Offset et échelle** : à partir d'un second fichier du même banc, moyenne de la dernière
  minute par voie, puis mise à l'échelle selon la convention `TYPE_…_coefficient`
  (`I` → A, `U` → V, `T` → °C, `P` → bar, `D` → mm). Masqué pour les fichiers `.seed`,
  déjà corrigés.
- **Fusion des calibres de pinces** : reconstitue une voie unique à partir des différents
  calibres d'une même famille `I_…`.
- **Synchronisation** : alignement sur le premier franchissement d'un seuil, conservation de la
  plage commune, ajout des voies de la source externe.
- **Inversion de signe**, **renommage**, **suppression**, **moyenne** et **somme** de voies.
- **Remise à zéro** complète via le bouton corbeille de la carte « Fichier ».

## Export `.seed`

TSV encodé en **windows-1252** (comme les fichiers de référence), première colonne `TEMPS`
partant de zéro et incrémentée du pas de temps, valeurs arrondies à 5 décimales avec
suppression des zéros de fin pour limiter le volume sans perte de précision.

Nom généré : `SEED_<fichier source>[_synchronise].seed`.

## Fichiers du projet

| Fichier | Rôle |
|---|---|
| [index.html](index.html) | structure de la page |
| [script.js](script.js) | lecture, traitements, export |
| [style.css](style.css) | thème Renault (palette du brand book, typographie NouvelR) |
| `style_rno/` | emblème et chartes graphiques de référence |
| `SEED/` | code MATLAB d'origine ayant servi de référence pour les formats |
