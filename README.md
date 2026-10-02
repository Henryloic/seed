# Guide fonctionnel

Documentation détaillée de l'outil de normalisation des mesures bancs.
Public visé : exploitants d'essais familiers du traitement du signal.

L'outil est une page HTML sans dépendance serveur. Tout le traitement a lieu en mémoire
dans le navigateur ; aucune donnée ne sort du poste. Les fichiers sont lus en flux
(`ReadableStream` + `TextDecoder`), ce qui permet de charger des enregistrements de plusieurs
centaines de méga-octets sans saturer la mémoire pendant le décodage.

---

## 1. Chargement d'un fichier de mesure

Carte **Fichier**, ligne `CHARGEMENT`. Un seul fichier de mesure est actif à la fois ;
en charger un nouveau réinitialise entièrement la session.

| Bouton | Format attendu | Encodage | Base de temps |
|---|---|---|---|
| `.M00x` | Morphée, tabulation, 4 lignes d'entête | sélectionnable | voie `Timer for free recording`, horodatage absolu reconstruit depuis `Date`/`Heure` |
| `BRBC3` | CSV virgule | UTF-8 | colonne `TEMPS` (s), instant initial lu dans la colonne `Time` |
| `Soufflerie` | CSV point-virgule, 1 ligne d'entête | windows-1252 | colonne 1 (s), date/heure en colonnes 3 et 2 de la première ligne de données |
| `.seed` | TSV, 1 ligne d'entête | windows-1252 | colonne `TEMPS` (s) |

### Détection de l'entête

Le format `.M00x` est strictement positionnel : ligne 1 `MNTPFILE`, ligne 3 les noms de voies,
ligne 4 les unités. Le format BRBC3 est variable : l'outil balaie les 30 premières lignes à la
recherche de la ligne commençant par `TEMPS` (noms de voies), la ligne suivante portant les
unités, et mémorise au passage la ligne `NUMERO_ESSAI` et ses valeurs associées.

### Reconstruction de la base de temps

Pour BRBC3, Soufflerie et SEED, le pas est estimé par `t[2] − t[1]` et la base de temps est
**régénérée uniformément** (`index × Δt`). Cela élimine la gigue d'horodatage du système
d'acquisition et garantit un pas strictement constant, prérequis de l'export et de tout
traitement fréquentiel ultérieur. Le pas retenu est affiché dans la barre d'état.

Pour `.M00x`, l'horodatage est reconstruit à partir du timer de l'enregistrement libre,
ce qui préserve les éventuelles ruptures de cadence.

### Sélection des voies

Les colonnes entièrement non numériques (texte, booléens, horodatages) sont écartées :
une voie n'est conservée que si elle contient au moins une valeur finie. Les valeurs
illisibles (`NaN`, champs vides, `True`/`False`) deviennent `null` et créent une
**discontinuité réelle** dans le tracé (`connectgaps: false`) — elles ne sont jamais
interpolées silencieusement.

### Normalisation des noms de voies

Les noms bruts sont traduits vers la nomenclature commune (`VITESSE_BANC`,
`EFFORT_VEH_TOTAL`, `DISTANCE_BANC_AV`, `COEFF_F0_BANC`…) via la table
`CORRESPONDANCES_VOIES`, recopie de `data_labelling.xlsx` / onglet `LABEL`, avec une colonne
par banc. La correspondance est exacte, avec tolérance sur la graphie espace / souligné.
Les voies non référencées conservent leur nom d'origine.

Spécificités BRBC3 : les espaces et les `~` (séparateur des voies CAN) sont remplacés par `_`,
et les constantes d'entête (`INERTIE`, `F0`, `F1`, `F2`) sont injectées comme voies
constantes afin d'être disponibles à l'export.

### Encodage

Le sélecteur `Encodage` ne s'applique qu'aux fichiers `.M00x`, dont la source peut varier.
Les autres formats ont un encodage imposé, déduit de leur outil générateur.

---

## 2. Correction : offset et mise à l'échelle

Section **Correction**, masquée pour les fichiers `.seed` (déjà normalisés).

### Convention d'instrumentation

Une voie est éligible si son nom suit `TYPE_<désignation>_<coefficient>` :

| Préfixe | Grandeur | Unité appliquée |
|---|---|---|
| `I` | courant | A |
| `U` | tension | V |
| `T` | température | °C |
| `P` | pression | bar |
| `D` | déplacement | mm |

### Calcul de l'offset

Le fichier d'offset doit être **du même banc** que la mesure (le filtre et le libellé du bouton
s'adaptent automatiquement). Pour chaque voie dont le nom comporte un préfixe de type,
l'outil calcule la **moyenne sur les 60 dernières secondes** de l'enregistrement d'offset.
C'est la fenêtre de référence au repos : elle suppose que l'acquisition d'offset se termine
par un palier stabilisé.

### Formule appliquée

```
valeur_corrigée = (valeur_brute − offset) × coefficient / 10 × signe
```

L'offset n'est **pas** soustrait aux voies de type `T` : une température absolue n'a pas de
zéro instrumental à compenser.

Le compte rendu indique le nombre de voies mises à l'échelle, le nombre d'offsets
effectivement appliqués et le nombre de voies sans correspondance (offset nul par défaut).
La correction repart toujours des valeurs brutes mémorisées : elle n'est jamais cumulative.
Le bouton `Réinitialiser` restaure l'état d'origine.

---

## 3. Fusion des calibres de pinces ampèremétriques

Disponible après la mise à l'échelle. Une famille regroupe les voies partageant la racine
`I_<désignation>` avec des calibres différents en suffixe.

**Sélection des pinces actives** : seules les voies dont l'écart-type dépasse `0,25 A` sont
retenues, ce qui écarte les pinces non branchées ou saturées à zéro.

**Reconstruction** : les pinces sont triées par calibre croissant. On part de la plus
sensible, puis pour chaque calibre supérieur on substitue les échantillons dont la valeur
absolue dépasse `calibre_précédent − 10`. Le résultat est une voie unique qui conserve la
résolution du petit calibre dans les faibles amplitudes et bascule sur le grand calibre
avant saturation.

La voie produite est marquée comme dérivée et porte le suffixe `(fusion des calibres)`.

---

## 4. Sources externes et synchronisation

Section **Synchronisation**, disponible dès qu'un fichier de mesure est chargé.

### Formats lus

| Bouton | Extension | Séparateur | Entête | Colonne temps |
|---|---|---|---|---|
| `CSV CAN` | `.csv` | `,` ou `;` auto-détecté | 1 ligne | ignorée (alignement par index) |
| `CarScanner` | `.csv` | `,` | 1 ligne | `HH:mm:ss.SSS`, passage de minuit géré |
| `OBD Facile` | `.txt` | `;` | 3 lignes (noms L1, unités L3) | secondes, notation française (`.` milliers, `,` décimale) |
| `Diagra` | `.csv` | `;` | 3 lignes (noms L2, unités L3) | millisecondes |

### Adaptation du pas de temps

Le pas médian de la source est comparé à celui du fichier de mesure. Si l'écart relatif
dépasse `10⁻³`, les voies sont **rééchantillonnées par interpolation linéaire** sur une grille
`0, Δt, 2Δt…` calée sur le pas cible. En deçà, les données sont conservées telles quelles
pour éviter un lissage inutile.

L'interpolation ne crée jamais de valeur hors du support : en dehors de la plage temporelle
de la source, ou à cheval sur un trou, l'échantillon reste `null`.

> Limite à connaître : l'interpolation linéaire est un filtre passe-bas implicite. Sur une
> source sous-échantillonnée par rapport au banc (CAN à 1 Hz contre banc à 10 Hz), le signal
> reconstruit ne restitue évidemment pas la dynamique manquante.

### Mise à l'échelle des voies importées

Bouton `Mettre à l'échelle`. Applique le **facteur seul**, sans offset :

```
valeur = valeur_brute × coefficient / 10
```

sur les voies de la source suivant la convention `TYPE_…_coefficient`. Le bouton reste grisé
si aucune voie éligible n'est détectée, ou après application (pour empêcher un double
facteur). Si la synchronisation a déjà eu lieu, les voies déjà fusionnées sont mises à jour
elles aussi.

### Alignement temporel

Bouton `Synchroniser`. On choisit une voie dans chaque source et un seuil. L'outil repère le
**premier franchissement par valeurs croissantes** du seuil dans chacune (`v[i−1] < seuil`
et `v[i] ≥ seuil`), puis décale les deux séries pour faire coïncider ces deux indices.

Seule la **plage commune** aux deux sources est conservée, de part et d'autre du repère.
Les voies de la source externe sont ensuite ajoutées au jeu de mesures, préfixées par le nom
de la source.

Le choix du seuil est déterminant : prendre un front franc et non ambigu (typiquement le
démarrage véhicule sur une voie de vitesse). Un seuil trop bas déclenche sur le bruit de zéro.

La synchronisation est **réversible** : l'état antérieur est mémorisé, et une nouvelle
synchronisation repart des données non tronquées.

---

## 5. Opérations sur les voies

Barre d'outils apparaissant sous le nom du fichier.

| Bouton | Effet |
|---|---|
| `Inverser le signe` | multiplie par −1 les voies sélectionnées ; le signe est mémorisé et réappliqué après une correction d'offset |
| `Renommer la voie` | modifie le nom affiché et exporté (une seule voie à la fois) |
| `Supprimer la voie` | retire définitivement la voie du jeu de données |
| `Calculer la moyenne (_moy)` | moyenne échantillon par échantillon des voies sélectionnées |
| `Calculer la somme (_sum)` | somme échantillon par échantillon |

Les voies calculées ignorent les échantillons non finis : la moyenne porte sur les seules
voies valides à cet instant, et vaut `null` si aucune ne l'est. L'unité n'est propagée que si
toutes les voies sources partagent la même, afin de ne pas produire d'agrégat hétérogène
silencieux.

Une voie calculée est identifiée par son type et la liste de ses sources : relancer le même
calcul met à jour la voie existante au lieu d'en créer un doublon.

---

## 6. Visualisation

Jusqu'à **16 voies** simultanées, tracées en `scattergl` (rendu WebGL, nécessaire au-delà de
quelques dizaines de milliers de points).

- Si les voies sélectionnées se répartissent sur exactement **deux unités**, un second axe Y
  est créé automatiquement à droite.
- L'axe X est temporel (horodatage absolu), survol en `x unified` pour comparer les voies à
  un même instant.
- Les discontinuités (`null`) sont affichées comme telles.

---

## 7. Sélection d'une plage temporelle

Sous le graphique.

- Le **rangeslider** sous l'axe X offre deux poignées et affiche la courbe en miniature.
- Les champs `Début (s)` et `Fin (s)` donnent le calage exact au clavier ; ils sont
  synchronisés en continu avec le zoom et les poignées.
- `Plage complète` restaure l'étendue totale.
- Le compteur indique le nombre d'échantillons retenus.

Les bornes sont converties en indices via le pas de temps, réordonnées si elles sont
inversées, et bornées à l'étendue disponible.

---

## 8. Export `.seed`

| Bouton | Étendue |
|---|---|
| `Exporter SEED` (carte Fichier) | enregistrement complet |
| `Exporter la plage affichée` (sous le graphe) | plage sélectionnée, suffixe `_<début>s-<fin>s` |

**Format produit**, conforme aux fichiers de référence :

- TSV encodé en **windows-1252** (un accent = un octet) ;
- première colonne `TEMPS`, **repartant de zéro** au début de la plage exportée et incrémentée
  du pas de temps ;
- une colonne par voie, au nom normalisé, espaces remplacés par des soulignés ;
- valeurs arrondies à **5 décimales**, zéros de fin supprimés.

La suppression des zéros de fin réduit typiquement le volume de 50 à 70 % sans aucune perte :
la valeur relue est strictement identique. Une valeur absente reste un champ vide, jamais un
zéro.

Le fichier produit est relisible par le bouton `.seed`, ce qui permet de reprendre une session
sans refaire les corrections.

---

## 9. Remise à zéro

Bouton corbeille, à droite de la ligne `CHARGEMENT`. Vide l'ensemble des états : mesures,
source externe, offset, sélections, graphique, bornes de plage et champs de fichier.
Une confirmation est demandée si des mesures sont chargées.
