# Comprendre et faire évoluer SEED : compétences HTML, JavaScript et CSS

Ce guide explique les compétences nécessaires pour comprendre, maintenir et faire évoluer ton programme. Les bases sont volontairement courtes ; l'accent porte sur l'importation, la correction des mesures, la fusion des calibres et la synchronisation.

**Point de départ :** la page exécutée est [index.html](index.html). Elle embarque son CSS dans `<style>` et son JavaScript dans `<script>`. Les fichiers [script.js](script.js) et [style.css](style.css) existent aussi, mais ne sont pas chargés par cette page et ne sont pas entièrement identiques à son contenu. Pour modifier le comportement actuel, commence donc par la page elle-même. Le [GUIDE.md](GUIDE.md) complète ce document du point de vue de l'utilisation.

Le parcours à comprendre est : **fichier brut → lecture et décodage → voies normalisées → correction → fusion éventuelle → synchronisation éventuelle → graphique et export SEED**. Les calculs sont réalisés dans le navigateur, sans serveur de traitement. Plotly est toutefois chargé depuis Internet pour les graphiques.

## 1. HTML : construire les commandes et les points de liaison

### Les bases, rapidement

**Structurer la page : `main`, `section`, `header`, titres et classes.** Le HTML décrit les éléments de l'écran, pas les calculs. Une `section` regroupe une partie fonctionnelle ; une `class` permet de partager une présentation CSS ; un `id` identifie un élément que JavaScript doit retrouver. Compétence essentielle : savoir passer d'un bouton visible à son `id`, puis au gestionnaire JavaScript associé.

### Importer et piloter les traitements

**Choisir un fichier : `input type="file"` et `label for`.** Les entrées `fichier`, `fichierBrbc3`, `fichierSf` et `fichierSeed` donnent accès à un objet `File` choisi par l'utilisateur. Le `label` sert de commande visible lorsque l'entrée native est masquée. L'attribut `accept` filtre le sélecteur, mais ne garantit pas la validité du contenu : le lecteur JavaScript doit toujours contrôler le format. Voir [les commandes d'importation](index.html#L282).

**Choisir le décodage : `select#encodage`.** Le sélecteur fournit `windows-1252` ou `utf-8` au lecteur BPERF. À comprendre : un fichier contient des octets ; le décodage transforme ces octets en caractères. Un mauvais encodage peut casser les accents et empêcher une correspondance de nom de voie, même si les nombres semblent corrects.

**Autoriser une action : `disabled` et `hidden`.** `disabled` rend une commande inutilisable ; `hidden` retire un bloc de l'affichage. Le programme s'en sert pour guider l'ordre des opérations : pas d'export avant l'importation, pas de fusion avant la correction. Ce sont des états d'interface, à compléter par des vérifications dans les fonctions de calcul.

**Afficher l'avancement : `progress` et zones d'état.** `progression` reçoit une fraction entre 0 et 1 ; `etat`, `etatCorrection`, `etatPinces` et `etatSync` expliquent le résultat ou l'erreur. `role="status"` et `aria-live="polite"` rendent les changements accessibles aux lecteurs d'écran. Compétence utile : distinguer l'avancement de la lecture, la réussite du traitement et les avertissements sur les données.

**Sélectionner des voies : `select multiple#variables`.** JavaScript crée les options à partir des voies disponibles et place leur identifiant dans `option.value`. Le texte visible peut changer sans changer cet identifiant. Cette séparation permet de renommer une voie sans perdre son identité dans les traitements. Voir [la sélection des variables](index.html#L388).

**Paramétrer la synchronisation : `dialog#syncDialog` et `form#syncForm`.** La boîte rassemble deux voies et un seuil numérique. `required` impose une saisie ; `step="any"` accepte les décimales. Le formulaire déclenche `submit`, intercepté par JavaScript pour calculer l'alignement sans recharger la page. À maîtriser : ouvrir avec `showModal()`, valider, afficher une erreur et fermer avec `close()`. Voir [le formulaire de synchronisation](index.html#L355).

**Paramétrer une correction unitaire : `dialog#correctionManuelleDialog`.** Le formulaire recueille un offset et un gain pour créer une voie `_mod`. Le HTML collecte les paramètres ; JavaScript vérifie leur validité et conserve la voie source. C'est un modèle réutilisable pour ajouter un traitement paramétré. Voir [le formulaire de correction](index.html#L372).

**Héberger le graphique : `div#monGraphe`.** Cette zone est un conteneur vide que Plotly remplit. Le HTML fournit son emplacement, le CSS sa taille et JavaScript les séries et axes. Cette séparation est importante : déplacer le conteneur ne doit pas changer le calcul des mesures.

**Définir une plage : `plageDebut` et `plageFin`.** Les champs donnent des bornes en secondes relatives au début de l'enregistrement actif. Ils sont reliés au zoom du graphique et à l'export partiel. À comprendre : une borne temporelle n'est pas encore un indice de tableau ; une fonction doit effectuer cette conversion.

### Un premier exercice HTML

Change uniquement le texte visible d'un bouton, en gardant son `id`. Vérifie que son action fonctionne toujours. Ensuite, suis le trajet `id du bouton → addEventListener → fonction appelée`. Tu auras identifié le lien essentiel entre l'interface et le programme.

## 2. JavaScript : le cœur du traitement des mesures

### Les notions indispensables avant les fonctions

**Le modèle de données.** `dates` contient les horodatages et `mesures` contient les objets représentant les voies. Une voie possède notamment `index`, `nom`, `nomAffiche`, `label`, `unite` et `valeurs`. L'invariant principal est que `colonne.valeurs[k]` correspond à `dates[k]` pour toutes les voies actives. Une suppression de ligne, une découpe ou un rééchantillonnage doit préserver cet alignement.

**Identité, affichage et origine.** `index` identifie une voie ; `nom` conserve son nom source, utilisé pour les conventions d'instrumentation ; `nomAffiche` sert à l'affichage et à l'export. `valeursInitiales` permet de recalculer sans cumuler les corrections. `derivee` et `externe` distinguent une voie calculée d'une voie importée. Renommer l'affichage n'est donc pas changer le coefficient instrumental inscrit dans le nom source.

**Tableaux, objets, `Map` et `Set`.** Savoir lire `map` pour transformer, `filter` pour sélectionner, `reduce` pour cumuler et `slice` pour copier ou découper suffit à comprendre une grande partie du programme. Une `Map` associe une clé à une valeur, par exemple un nom de voie à son offset ; un `Set` conserve des identifiants uniques. Attention : `{ ...colonne }` copie l'objet, mais pas ses tableaux ; il faut aussi copier `valeurs` quand on veut une sauvegarde indépendante.

**Valeur absente et valeur nulle.** `null` représente une mesure absente ou illisible ; `0` est une vraie mesure. Les confondre fausse les statistiques. Le programme préserve souvent `null` dans les transformations, mais la détection des pinces actives les remplace par zéro : c'est un choix local à connaître, pas une règle générale à reproduire.

**Asynchronisme et erreurs.** `async` et `await` permettent d'attendre une lecture de fichier ; `try/catch/finally` sépare réussite, erreur et nettoyage. Le compteur `chargement` aide à ignorer un ancien import lorsque l'utilisateur a déjà changé de session. Cela ne rend pas les calculs parallèles : de grosses boucles peuvent encore ralentir l'interface.

### Importation : transformer les fichiers en données exploitables

#### `$` : retrouver un élément de l'écran

Ce raccourci appelle `document.getElementById`. C'est le pont entre les fonctions et les champs HTML : `$("seuilSync").valueAsNumber` lit le seuil. À maîtriser rapidement : `value`, `textContent`, `disabled`, `hidden` et `selectedOptions`.

#### `tableRenommage` : traduire les noms entre bancs

La fonction construit une `Map` depuis `CORRESPONDANCES_VOIES`, en ignorant les entrées `NC` et en acceptant une variante avec soulignés à la place des espaces. Sa valeur ajoutée est d'offrir une nomenclature commune aux lecteurs BPERF, BRBC3 et Soufflerie. Pour reconnaître une nouvelle voie, commence par la table de correspondances plutôt que par une condition dans chaque lecteur. Voir [la table et sa construction](index.html#L417).

#### `preparerColonnes` : fabriquer les objets de voie

Elle associe noms, noms normalisés, unités, indices et tableaux de valeurs, puis écarte les premières colonnes selon le format. À comprendre : le lecteur adapte un fichier particulier à un modèle commun ; les calculs suivants n'ont plus à connaître son organisation originale.

#### `lireFichier` : lire en flux et décoder correctement

Elle utilise `fichier.stream().getReader()` et `TextDecoder`, garde les morceaux de ligne incomplets dans `reste`, puis appelle `ligneLue` pour chaque ligne entière. Le décodage continu évite de casser un caractère entre deux blocs. L'intérêt est de ne pas charger tout le texte brut d'un coup, mais les tableaux numériques restent stockés en mémoire. Le `finally` libère le lecteur. Voir [la lecture en flux](index.html#L756).

#### `champsCsv` : respecter les séparateurs entre guillemets

Elle découpe une ligne en suivant l'état `guillemets`, de sorte que `"nom, détaillé",12` forme deux champs et non trois. Elle gère aussi les guillemets doublés à l'intérieur d'un champ. Notion essentielle : un CSV n'est pas toujours un simple `split(",")`. Limite de ce lecteur : il traite une ligne à la fois, pas les champs contenant un retour à la ligne.

#### `analyserFichier` : importer un fichier Morphée `.M00x`

Elle contrôle `MNTPFILE`, les colonnes `Date`/`Heure`, les unités et la présence du timer, puis reconstruit les dates avec `instantInitial + (timer - timerInitial) × 1000`. Elle ignore les lignes de largeur incorrecte ou sans timer valide et transforme les champs non numériques en `null`. Compétence à acquérir : distinguer une erreur bloquante de format et une ligne de données rejetée. Voir [le lecteur BPERF](index.html#L499).

#### `dateMesure` : interpréter une date et une heure source

Elle reconnaît une date jour/mois/année et une heure, construit une chaîne de type ISO et vérifie qu'elle est interprétable. L'expression régulière décrit les formats autorisés. À connaître : ces chaînes ne précisent pas de fuseau horaire et sont interprétées localement ; elles ne constituent pas une référence UTC universelle.

#### `dateDepuisInstant` : reformater un instant en horodatage

Elle convertit des millisecondes en chaîne date/heure locale avec trois chiffres pour les millisecondes. Les fonctions internes de remplissage avec `padStart` garantissent une présentation stable. Garde toujours en tête les unités : `Date` travaille en millisecondes, les timers du banc en secondes.

#### `analyserFichierBrbc3` : adapter un en-tête variable

Elle cherche la ligne `TEMPS` dans les 30 premières lignes, récupère les unités, les données et l'origine temporelle dans `Time`, avec repli sur `fichier.lastModified`. Elle transforme aussi les constantes reconnues de l'en-tête en voies constantes, puis recalcule la distance. Sa valeur ajoutée est de conserver des paramètres d'essai même s'ils ne figurent pas dans les lignes de mesure. Voir [le lecteur BRBC3](index.html#L597).

#### `nettoyerNomBanc` : rendre les noms utilisables

Elle retire les retours à la ligne et les espaces de bord ; pour BRBC3, elle remplace aussi espaces et `~` par `_`. Notion utile : nettoyer la graphie et traduire un nom métier sont deux opérations différentes, respectivement assurées ici et par la table de renommage.

#### `instantBrbc3` : extraire l'origine temporelle BRBC3

Elle reconnaît `jj/mm/aaaa hh:mm:ss`, avec fractions éventuelles et virgule ou point décimal. Elle renvoie des millisecondes ou `null`. Une absence d'heure exploitable ne doit pas être confondue avec un instant égal à zéro.

#### `baseTempsReguliere` : reconstruire une grille uniforme

Elle prend `temps[2] - temps[1]`, ou `temps[1] - temps[0]` si seuls deux points existent, puis génère une date par `index × pas`. Ce n'est ni une moyenne de tous les pas ni une interpolation des mesures : les valeurs sont conservées, leurs dates sont régénérées. C'est adapté à une acquisition supposée régulière, mais cela ne préserve pas les irrégularités réelles ou les trous temporels. Voir [la reconstruction temporelle](index.html#L570).

#### `recalculerDistanceBrbc3` : intégrer la vitesse

Elle accumule `vitesse / 3.6 × pas` pour produire une distance en mètres, si les voies vitesse et distance sont présentes. C'est une intégration par rectangles : à 36 km/h et avec un pas de 0,1 s, chaque point ajoute 1 m. Une vitesse absente contribue ici comme zéro ; cette hypothèse doit être comprise avant d'utiliser la distance reconstruite.

#### `analyserFichierSoufflerie` : lire un export français

Elle lit un fichier séparé par `;` en Windows-1252, convertit les nombres au format français et prend la date et l'heure de la première ligne exploitable. Elle normalise les noms et reconstruit une base régulière. Pour adapter un nouveau format, identifie d'abord le séparateur, l'encodage, les lignes d'en-tête et les colonnes de temps. Voir [le lecteur Soufflerie](index.html#L714).

#### `analyserFichierSeed` : relire les données déjà normalisées

Elle attend un fichier tabulé Windows-1252 commençant par `TEMPS` et ne réapplique pas de renommage métier. Le fichier n'enregistrant pas l'horodatage absolu ni les unités dans une ligne dédiée, la relecture utilise `lastModified` comme origine et des unités vides. Notion importante : l'export sauvegarde des valeurs et des noms, pas tout l'état de la session.

#### `analyserCsv` : importer une source CAN générique

Elle détecte `,` ou `;`, considère la première colonne comme temps et conserve les autres colonnes numériques. Les temps servent notamment au calcul de l'offset ; la synchronisation de ce lecteur reste fondée sur les indices, sans rééchantillonnage automatique. Il faut donc fournir une cadence compatible avec le banc. Attention également : les valeurs des voies sont ici attendues avec un point décimal, même si le séparateur est `;`. Voir [le lecteur CSV CAN](index.html#L802).

#### `nombreSource` : convertir sans accepter un nombre partiel

Elle valide toute la chaîne avec une expression régulière avant `Number`, puis ne conserve que les valeurs finies. En mode français, elle retire les points de milliers et remplace la virgule décimale : `1.234,5` devient `1234.5`. Ce mode ne convient pas à un fichier où `1.234` veut dire 1,234 : le format numérique est une convention d'entrée, pas une détection automatique.

#### `instantSource` : mettre les temps dans une unité commune

Elle convertit des secondes, des millisecondes ou une heure `HH:mm:ss.SSS` en secondes. La conversion explicite évite un décalage d'un facteur 1000 entre fichiers. Une heure seule ne contient pas le jour : la gestion de minuit est réalisée par le lecteur temporel.

#### `analyserSourceTemporelle` : adapter CarScanner, OBD Facile et Diagra

Elle utilise `CONFIG_SOURCES` pour choisir encodage, séparateur, lignes d'en-tête et format de temps. Elle gère le passage de minuit des horaires, crée un temps relatif, puis compare le pas source au pas cible. Si l'écart dépasse `pasCible × 0.001`, elle rééchantillonne les voies. La compétence centrale est la programmation pilotée par configuration : des formats proches partagent un même lecteur. Voir [le lecteur temporel](index.html#L894).

#### `pasMedian` : estimer la cadence d'une source

Elle calcule les écarts strictement positifs, les trie numériquement et retient l'élément central supérieur. Cette estimation résiste mieux qu'une moyenne à quelques grands écarts. Elle ne prouve cependant pas que toute l'acquisition est régulière et ne valide pas à elle seule l'ordre des temps.

#### `interpolerLineaire` : calculer les valeurs sur une nouvelle grille

Elle avance un curseur dans les temps sources et calcule entre deux points `avant + (après - avant) × fraction`. Entre 10 A à 0 s et 20 A à 1 s, elle produit 15 A à 0,5 s. Elle ne remplit pas hors du support ni lorsque l'une des deux valeurs est `null`. À comprendre : augmenter la fréquence d'échantillonnage n'invente pas de dynamique mesurée ; le lecteur suppose des temps ordonnés et n'ajoute pas de filtre anti-repliement pour une réduction de cadence. Voir [l'interpolation](index.html#L877).

#### `chargerFichier` : orchestrer un nouvel import banc

Elle réinitialise la session, choisit le lecteur via `LECTEURS_BANCS`, suit la progression et sauvegarde les valeurs initiales. Elle expose ensuite les commandes adaptées au banc. Le contrôle `session !== chargement` évite qu'un ancien import banc remplace une session plus récente. À retenir pour ajouter un banc : compléter sa configuration et son lecteur, plutôt que dupliquer toute l'orchestration. Voir [le chargement principal](index.html#L1078).

#### `reinitialiserInterface` : remettre les données et commandes à zéro

Elle vide mesures, dates, sources et sélections, ferme les dialogues et purge le graphique. Notion essentielle : vider seulement l'écran ne suffit pas ; les variables internes et les sauvegardes doivent aussi être réinitialisées pour ne pas mélanger deux essais.

#### `chargerSourceExterne` : importer la seconde acquisition

Elle choisit le lecteur CSV CAN ou le lecteur temporel, transmet le pas du banc, puis remplace `sourceExterne` et active les commandes utilisables. Elle contrôle aussi que la session et le fichier sélectionné sont toujours les mêmes après l'attente. Le programme conserve une source externe courante : ce n'est pas un gestionnaire de plusieurs sources simultanées.

### Correction : passer du signal brut à une grandeur physique

#### `calculerOffsets` : mesurer le zéro instrumental

Elle calcule, pour chaque voie éligible, la moyenne des valeurs finies sur les 60 dernières secondes du fichier d'offset, et stocke le résultat par nom dans une `Map`. Le principe métier suppose un palier final au repos et stable ; la fonction ne vérifie pas cette stabilité. Pour un CSV, la première colonne doit permettre de repérer cette fenêtre. Voir [le calcul des offsets](index.html#L1187).

#### `echelleVoie` : interpréter la convention d'instrumentation

Elle décompose le nom `TYPE_désignation_coefficient`, reconnaît `I`, `U`, `T`, `P` ou `D`, puis lit le coefficient final et l'unité associée. Le nom source est donc aussi un paramètre de calcul. Un suffixe non numérique rend la voie inéligible ; changer cette convention touche à la fois la correction et la fusion.

#### `appliquerCorrection` : corriger sans cumuler les facteurs

Elle repart de `valeursInitiales` et applique `(valeur - offset) × coefficient / 10 × signe`, sauf pour les températures dont l'offset n'est pas soustrait. Exemple : valeur brute 2, offset 0,2 et coefficient 100 donnent 18 dans l'unité de la voie, avec un signe positif. Elle laisse les voies externes à leur traitement dédié et retire les voies dérivées précédentes : il faut donc recréer celles dont on a besoin après correction. Voir [la correction banc](index.html#L1263).

#### `reinitialiserCorrection` : restaurer les valeurs de référence

Elle supprime les voies dérivées, restaure valeurs, unités et libellés de référence, en conservant le choix de signe. La sauvegarde `valeursInitiales` est ce qui rend la restauration possible. Notion utile : une opération réversible doit définir précisément ce qu'elle restaure et ce qu'elle conserve.

#### `appliquerEchelleSource` : corriger les voies externes

Pour CAN, elle utilise un fichier d'offset et applique offset plus coefficient ; pour les autres sources, elle applique le coefficient seul. Si les voies externes sont déjà intégrées par synchronisation, elle actualise aussi leurs valeurs dans `mesures`. `echelleAppliquee` empêche une seconde multiplication accidentelle. C'est un exemple de cohérence à maintenir entre une source et sa copie intégrée. Voir [la correction des sources](index.html#L1217).

### Fusion des calibres : conserver sensibilité et plage de mesure

#### `famillesPinces` : regrouper les calibres d'une même mesure

Elle reconnaît les noms `I_racine_calibre` par expression régulière et rassemble les voies sous la racine commune dans une `Map`. Ainsi, `I_BAT_100` et `I_BAT_1000` appartiennent à `I_BAT`. Elle exclut les voies déjà fusionnées pour ne pas réinjecter le résultat dans ses propres sources. Voir [le regroupement des pinces](index.html#L1334).

#### `remplirFamillesPinces` : proposer les familles disponibles

Elle crée les options du sélecteur et n'autorise le bouton que si une correction banc a été appliquée et une famille est choisie. La logique actuelle dépend de `correctionAppliquee`, pas uniquement de l'unité affichée ou d'une mise à l'échelle externe. C'est important pour comprendre pourquoi une famille détectée ne suffit pas à activer la fusion.

#### `fusionnerFamillePinces` : sélectionner les calibres échantillon par échantillon

Elle retient les pinces dont l'écart-type d'échantillon dépasse 0,25 A, les trie par coefficient croissant et copie la plus sensible. Pour chaque calibre supérieur, elle remplace la valeur fusionnée lorsque **la valeur absolue de ce calibre supérieur** dépasse `coefficient du calibre précédent - 10`. Avec des calibres 100 et 1000, une valeur de 95 A sur le grand calibre est utilisée, car elle dépasse 90 A. Le résultat n'est pas une moyenne : il conserve le petit calibre aux faibles amplitudes et bascule au-delà du seuil. Voir [l'algorithme de fusion](index.html#L1363).

**Les limites métier de cette fusion.** L'écart-type repère une variation, pas la santé du capteur : une pince valide mais constante peut être exclue ; une pince bruitée peut être retenue. Les `null` sont assimilés à zéro pour cette statistique. La marge de 10 A est fixe ; il n'y a ni hystérésis ni lissage au changement de calibre. Une valeur manquante du petit calibre n'est pas systématiquement remplacée si le grand calibre reste sous le seuil. Comprendre ces choix est nécessaire avant de changer les paramètres.

### Synchronisation : aligner un événement, puis garder la plage commune

#### `remplirVoiesSync` : proposer un signal de référence

Elle remplit les deux sélecteurs et privilégie `VITESSE_BANC` côté banc, `IVehicleSpeed` côté externe, puis cherche un nom contenant vitesse ou speed. Le choix automatique est une aide, pas une preuve que les signaux sont comparables. Avant l'alignement, vérifie qu'ils décrivent le même phénomène et utilisent la même unité.

#### `premierFranchissement` : détecter un front montant

Elle cherche le premier couple valide où la valeur précédente est strictement sous le seuil et la valeur courante au-dessus ou égale. Avec `[0, 2, 6, 8]` et un seuil de 5, le repère est l'indice 2. Si le signal démarre déjà au-dessus, aucun repère n'est trouvé sans passage ultérieur sous puis au-dessus du seuil. Il n'y a ni interpolation du moment exact ni filtrage du bruit. Voir [la détection du front](index.html#L960).

#### `plageSynchronisee` : calculer les découpes compatibles

Elle traduit l'écart entre les deux repères en indices de début et prend la longueur commune restante. Pour des repères banc 100 et externe 70, elle coupe 30 points au début du banc et zéro côté externe. Avec un pas commun de 0,1 s, cela représente 3 s. Cette fonction pure, sans accès à l'écran, est particulièrement facile à tester avec de petits nombres.

#### `synchroniserSources` : découper et réunir les acquisitions

Elle détecte les deux repères, calcule la plage commune, découpe toutes les voies aux indices correspondants et ajoute les voies externes avec de nouveaux identifiants. Elle sauvegarde une copie avant la première synchronisation pour permettre un nouvel alignement à partir de cette référence. Les dates retenues sont celles du banc. Cette méthode corrige un décalage constant à la résolution d'un échantillon, pas une dérive progressive entre deux horloges. Voir [la synchronisation complète](index.html#L1135).

**Condition à ne pas oublier.** Un alignement par indices suppose des cadences compatibles. CarScanner, OBD Facile et Diagra peuvent être rééchantillonnés ; le CSV CAN générique ne l'est pas. Vérifie aussi que le premier franchissement correspond au même événement physique dans les deux fichiers, surtout s'il y a plusieurs démarrages ou du bruit. Enfin, les modifications effectuées après la première synchronisation ne sont pas automatiquement reportées dans la sauvegarde `donneesAvantSync`.

### Voies calculées et opérations de préparation

#### `colonnesSelectionnees` : relier la sélection aux données

Elle transforme les options sélectionnées en un `Set` d'indices, puis filtre les voies correspondantes. C'est un petit utilitaire partagé : une fonction de calcul doit travailler sur les objets de voie, pas sur leur texte affiché.

#### `inverserSigneSelection` : changer une convention de signe

Elle multiplie les valeurs non absentes par -1 et mémorise `signe` pour que la correction banc respecte ce choix. Deux inversions rendent les valeurs initiales. C'est utile lorsqu'un courant ou un effort a été acquis avec une orientation opposée à celle souhaitée.

#### `renommerVoieSelectionnee` : modifier le nom visible

Elle exige une seule voie, demande un nom non vide et actualise `nomAffiche` et les libellés. Elle laisse `nom` intact pour préserver la convention d'instrumentation. Comme l'export utilise le nom affiché, le renommage change aussi l'en-tête exporté.

#### `supprimerVoieSelectionnee` : retirer une voie de la session

Elle demande confirmation, retire la voie de `mesures` et reconstruit la sélection et les familles. La suppression touche les données actives, pas seulement le graphique. Un résultat dérivé déjà calculé n'est pas un lien vivant qui se recalculera automatiquement après cette suppression.

#### `calculerVoieSelectionnee` : créer une moyenne ou une somme entre voies

Elle calcule à chaque instant la somme ou la moyenne des valeurs finies sélectionnées, en ignorant les absences et en produisant `null` si aucune valeur n'est valide. C'est une moyenne **entre signaux**, pas une moyenne temporelle ni un filtre glissant. Elle conserve l'unité si toutes les unités sont égales, mais n'interdit pas les mélanges incohérents : vérifier les grandeurs reste indispensable. Voir [les voies somme et moyenne](index.html#L1477).

#### `ouvrirCorrectionManuelle` : préparer une correction non destructive

Elle exige une seule voie et préremplit offset et gain, éventuellement avec les paramètres d'une correction déjà créée pour cette source. C'est la partie interface de l'opération : aucune valeur n'est encore transformée.

#### `appliquerCorrectionManuelle` : créer ou actualiser une voie `_mod`

Elle valide les deux paramètres puis calcule `(valeur - offset) × gain` à partir des valeurs actuelles de la source. La voie source reste intacte ; une voie dérivée est créée ou actualisée avec son `sourceIndex`. Exemple : 12, offset 2 et gain 3 donnent 30. C'est un modèle simple pour concevoir une transformation traçable. Voir [la correction unitaire](index.html#L1546).

#### `creerPhaseEssai` : segmenter un cycle Soufflerie

Elle travaille uniquement sur `VITESSE_CYCLE`, assimile les valeurs proches de zéro à zéro avec un seuil de 0,05 km/h, détecte les changements de variation et numérote des groupes. Elle fusionne certains groupes courts selon une durée de 200 s, puis utilise notamment un seuil de 500 **points**, distinct de 500 secondes. C'est une règle métier de segmentation, pas une détection universelle de phases ; contrôle le résultat sur un cycle connu avant de changer la cadence ou les seuils. Voir [la création des phases](index.html#L1599).

### Visualisation, plage et export

#### `remplirVariables` : reconstruire la sélection sans perdre les choix

Elle crée les options depuis les voies actives et utilise un `Set` d'identifiants pour conserver la sélection. Elle peut proposer vitesse et efforts par défaut. À retenir : après une modification de `mesures`, il faut actualiser l'interface ; changer un tableau ne modifie pas automatiquement le HTML.

#### `tracer` : visualiser beaucoup de points avec Plotly

Elle transforme les voies sélectionnées en traces `scattergl`, limite la sélection à 16 voies et appelle `Plotly.react`. `connectgaps: false` montre les absences comme des coupures. Deux unités distinctes donnent deux axes ; avec davantage d'unités, le graphique revient à un axe générique, ce qui demande de la prudence dans la comparaison. Les réglages Plotly, notamment les couleurs et axes, sont du JavaScript, pas du CSS. Voir [la construction du graphique](index.html#L1751).

#### `pasTemps` : estimer le pas du banc actif

Elle prend la médiane supérieure des écarts positifs parmi les premiers intervalles, jusqu'à 100, avec un repli à 1 s. Ce pas sert au rééchantillonnage externe, aux indices de plage et à l'export. Il faut donc distinguer « estimation de cadence » et « garantie de régularité de tout le fichier ».

#### `secondesDepuisDebut` : passer en temps relatif

Elle soustrait la première date à l'instant fourni et divise par 1000. Ce petit changement de repère permet à l'utilisateur de saisir des secondes simples, même si le graphique est tracé avec des dates absolues.

#### `indicesPlage` : convertir les bornes en indices

Elle divise les secondes par le pas estimé, arrondit, borne les résultats dans le tableau et remet début et fin dans l'ordre si nécessaire. Cette méthode suppose une grille régulière ; pour des dates irrégulières, chercher directement dans les horodatages serait une autre stratégie.

#### `majEtatPlage` : compter les points sélectionnés

Elle affiche `fin - debut + 1`, car les deux bornes sont incluses. Ce détail évite une erreur classique de comptage entre indices et nombre d'échantillons.

#### `reinitialiserPlage` : choisir toute la durée active

Elle met le début à zéro et la fin à la durée entre les première et dernière dates, puis actualise le compteur. Elle sert après un import et lors du retour à la plage complète.

#### `ecrirePlageDepuisGraphe` : traduire un zoom en bornes numériques

Elle reçoit l'événement Plotly `plotly_relayout`, récupère la plage X et la convertit en secondes relatives. C'est un exemple de synchronisation entre deux représentations d'un même état : le zoom et les champs numériques.

#### `appliquerPlageAuGraphe` : traduire les champs en zoom

Elle calcule les indices, puis appelle `Plotly.relayout` avec les dates correspondantes. Elle ne supprime aucune mesure ; elle change seulement la fenêtre visible. L'export partiel effectuera la sélection réelle des lignes.

#### `nomVoieSeed` : préparer les en-têtes exportés

Elle prend le nom affiché, avec repli sur le nom source, et remplace les espaces par `_`. À connaître : cette normalisation ne garantit pas l'unicité des noms ; deux noms affichés proches peuvent produire le même en-tête.

#### `formaterValeurSeed` : limiter la précision écrite

Elle arrondit à cinq décimales, retire les zéros inutiles et écrit un champ vide pour une valeur non finie. Le but est de réduire le poids du fichier sans longue écriture numérique. Cette précision concerne le fichier exporté, pas les calculs internes.

#### `contenuExportSeed` : produire une table exploitable par SEED

Elle écrit une première colonne `TEMPS`, toutes les voies actives séparées par des tabulations et les lignes de la plage demandée. Le temps exporté repart de zéro et est reconstruit par `index relatif × pas estimé` : les dates absolues et les éventuelles irrégularités ne sont pas conservées. Exporter une plage ne limite pas les colonnes aux seules voies tracées. Voir [le contenu SEED](index.html#L1018).

#### `encoderCp1252` : garantir l'encodage des octets exportés

Elle transforme les caractères en `Uint8Array` selon Windows-1252, avec une table pour certains caractères spécifiques. Un caractère non représentable devient `?`. Indiquer seulement `charset=windows-1252` dans un `Blob` ne convertit pas magiquement une chaîne : il faut produire les octets correspondants.

#### `exporterSeed` : déclencher un téléchargement local

Elle fabrique le contenu et le nom de fichier, crée un `Blob`, génère une URL temporaire avec `URL.createObjectURL`, puis clique sur un lien de téléchargement. `URL.revokeObjectURL` libère ensuite cette ressource. Compétence utile : distinguer produire le contenu, l'encoder et déclencher le téléchargement. Voir [le téléchargement](index.html#L1826).

#### `addEventListener` : relier toutes les commandes aux fonctions

Les gestionnaires de fin de script traduisent `change`, `click` et `submit` en appels aux fonctions précédentes. `preventDefault()` évite qu'un formulaire recharge la page. Pour comprendre une action, commence par chercher son `id` dans ces gestionnaires, puis suis la fonction appelée jusqu'à la modification des données. Voir [les branchements des imports](index.html#L1801).

### Exercices courts pour gagner en autonomie

1. **Lire un fichier minuscule.** Sur une copie d'un fichier valide, ne conserve que quelques lignes de mesure. Introduis un champ vide et vérifie qu'il devient une absence, pas zéro. Compare le nombre de dates et de valeurs de chaque voie.
2. **Vérifier un calcul à la main.** Choisis une voie instrumentée et trois points. Calcule toi-même `(brut - offset) × coefficient / 10`, puis compare avec le résultat. Pour un exercice plus simple, utilise la correction unitaire avec offset 2 et gain 3.
3. **Comprendre la fusion.** Repère deux calibres de la même famille et compare leurs courbes à la voie fusionnée autour du seuil. Vérifie que le grand calibre décide du remplacement et que le résultat n'est pas une moyenne.
4. **Tester la synchronisation sans gros fichier.** Dans la console du navigateur, appelle `premierFranchissement([0, 2, 6, 8], 5)` : résultat attendu 2. Appelle `plageSynchronisee(200, 180, 100, 70)` : débuts 30 et 0, longueur 170. Ces fonctions isolées permettent de comprendre l'algorithme avant de modifier l'interface.
5. **Ajouter une correspondance de voie.** Sur une copie de la page, ajoute une entrée pertinente dans `CORRESPONDANCES_VOIES`, importe un petit fichier et contrôle le nom affiché puis l'en-tête exporté. C'est une première modification métier sans toucher aux calculs.
6. **Relire un export.** Exporte une petite plage, ouvre-la comme texte et réimporte-la. Vérifie le temps remis à zéro, les noms, les champs absents et l'arrondi. Ne t'attends pas à retrouver automatiquement les unités ou les dates absolues.

Pour déboguer, utilise les outils développeur du navigateur : la **Console** pour les résultats et erreurs, **Sources** pour un point d'arrêt, et l'inspection de `mesures`, `dates` ou `sourceExterne` pendant une pause. Modifie un seul comportement à la fois, puis vérifie un cas nominal, une absence de valeur et un cas limite. Les résultats dérivés sont calculés à un instant donné : ils ne se recalculent pas tous automatiquement quand leurs sources changent.

## 3. CSS : rendre un outil de données lisible et stable

Le CSS n'a pas de fonctions métier comparables à celles du JavaScript. Les mini-paragraphes ci-dessous décrivent donc les règles qui remplissent une fonction visuelle. Elles se trouvent dans [le bloc de styles de la page](index.html#L7).

### Les bases, rapidement

**Sélecteurs et cascade.** `.row` cible une classe ; `#variables` cible un identifiant ; `button:disabled` cible un état. Plusieurs règles peuvent s'appliquer au même élément : la spécificité et l'ordre déterminent laquelle gagne. Avant de modifier une apparence, utilise l'inspecteur du navigateur pour identifier la règle réellement appliquée.

### Organiser l'espace de travail

**`:root` et `var()` : centraliser le thème.** Les variables `--accent`, `--txt`, `--line` et les couleurs Renault évitent de répéter les valeurs. Modifier une variable change plusieurs composants de manière cohérente. Attention : les couleurs du graphique Plotly sont définies séparément dans `tracer`.

**`box-sizing: border-box` : maîtriser la taille des éléments.** Cette règle inclut bordures et marges intérieures dans la largeur annoncée. Elle évite qu'un champ à `width:100%` déborde simplement à cause de son padding. C'est une petite base avec beaucoup d'effet sur la stabilité du layout.

**`.workspaceGrid` : répartir sélection et graphique.** La grille crée deux colonnes, avec `minmax(280px, 1fr)` pour les variables et `minmax(0, 2fr)` pour le graphique. `min-width:0` sur les enfants permet de réduire les blocs malgré un contenu long. Compétence utile : comprendre pourquoi un contenu peut imposer une largeur minimale et casser une grille.

**`.row` et `.ligneChargement` : organiser les commandes.** Flexbox aligne les éléments, `gap` les espace et `flex-wrap:wrap` permet leur retour à la ligne. La ligne de chargement réutilise cette règle avec un espacement plus compact. À maîtriser : Grid sert ici à la structure principale, Flexbox aux rangées de commandes.

**`.measureToolbar` : garder une barre d'actions exploitable.** Elle utilise une rangée sans retour à la ligne et un défilement horizontal. Les boutons gardent leur largeur avec `flex:0 0 auto` et leur texte reste sur une ligne. C'est un compromis explicite entre compacité, stabilité et accès aux actions sur un écran étroit.

**`.selectViewport`, `#variables` et les options : contenir les noms longs.** Le conteneur gère le défilement et le sélecteur autorise une largeur liée au contenu avec `width:max-content`. Les options ne coupent pas leurs noms. Le couple `direction:rtl` sur le sélecteur et `direction:ltr` sur les options est un réglage local particulier : teste noms longs et défilement avant de le simplifier.

**`.card` et `.processingTools` : rendre les groupes lisibles.** Bordures, espacement interne et séparateurs distinguent import, correction et synchronisation sans changer leur comportement. La compétence utile est de donner une hiérarchie visuelle aux commandes liées, tout en gardant assez de place pour les messages d'erreur.

### Rendre les états et interactions compréhensibles

**`[hidden]` : respecter la visibilité pilotée par JavaScript.** La règle `display:none !important` garantit qu'un bloc masqué ne réapparaît pas parce qu'une autre règle lui impose `display:flex`. C'est un contrat entre HTML, CSS et JavaScript, pas simplement une décoration.

**`button`, `.filebtn`, `.btn2` et `:disabled` : distinguer les actions.** Les styles différencient chargement, action principale et action secondaire ; l'état désactivé change les couleurs et le curseur. La règle de survol utilise `:not(:disabled)` pour ne pas suggérer une action disponible lorsqu'elle ne l'est pas.

**`:focus-visible` : conserver un repère au clavier.** Le contour indique quel élément reçoit la prochaine action clavier. C'est indispensable pour naviguer sans souris et comprendre où se trouve le focus après l'ouverture d'un dialogue.

**Les dialogues et `::backdrop` : isoler un traitement paramétré.** Leur largeur dépend de l'espace disponible, leur hauteur est bornée par la fenêtre et `overflow:auto` permet de défiler. Le fond assombri matérialise la modalité. Test utile : ouvrir le formulaire dans une petite fenêtre et vérifier que le bouton de validation reste accessible.

**`overflow-wrap:anywhere` et `[role=alert]` : afficher les erreurs utiles.** Les noms de fichiers peuvent se couper pour éviter un débordement ; les alertes deviennent visuellement distinctes. Le style signale l'erreur, mais le message lui-même est produit par JavaScript et doit expliquer ce que l'utilisateur peut corriger.

### Adapter la visualisation à l'écran

**`.plot` : fournir une surface stable au graphique.** Elle fixe une hauteur de 600 px et une largeur de 100 %, puis une hauteur de 460 px sur petit écran. Plotly est configuré avec `responsive:true` pour s'adapter à cette surface. Taille CSS et configuration JavaScript travaillent ensemble.

**`@media` : adapter l'espace sans changer les données.** Sous 800 px, la grille passe à une colonne ; sous 640 px, les marges diminuent et les commandes prennent toute la largeur. Ces règles changent l'organisation visuelle, pas l'importation ni les calculs. Vérifie surtout les noms longs, les dialogues et les barres d'actions. Voir [les adaptations aux petits écrans](index.html#L262).

### Un premier exercice CSS

Sur une copie de la page, modifie seulement la hauteur de `.plot`, puis la largeur relative des colonnes de `.workspaceGrid`. Compare dans une fenêtre large puis étroite. Si une commande déborde, inspecte d'abord `min-width`, `white-space`, `flex-wrap` et `overflow` : ce sont les réglages qui contrôlent le comportement, davantage qu'une réduction arbitraire de la police.

**Priorité d'apprentissage :** comprendre le modèle `dates + mesures`, puis `lireFichier`, les lecteurs, la correction, la fusion et les trois fonctions de synchronisation. HTML et CSS te permettront ensuite d'ajouter des commandes et de les présenter. Tu n'as pas besoin de maîtriser tout le langage pour commencer : savoir tracer le trajet d'une donnée et vérifier un petit résultat est le premier niveau d'autonomie.