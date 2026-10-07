# Petite formation HTML, JavaScript et CSS avec SEED

**Objectif :** comprendre une petite portion de code, savoir la modifier et vérifier le résultat. Aucun prérequis n'est supposé. Pas besoin de tout retenir : ce document sert aussi de fiche de rappel.

Le [guide des compétences](COMPETENCES_SEED.md) explique les fonctions du programme. Ici, on apprend les outils utilisés **à l'intérieur** de ces fonctions.

Trois rôles à distinguer : **HTML crée les éléments**, **JavaScript les fait agir et calcule**, **CSS organise leur apparence**. HTML et CSS n'ont pas de fonctions de calcul comme JavaScript ; on y utilise surtout des balises, des attributs et des règles.

Dans l'application actuelle, les trois sont réunis dans [index.html](index.html) : le CSS entre `<style>` et `</style>`, les éléments HTML dans `<body>`, le JavaScript dans le bloc `<script>` de fin de page. Travaille sur une copie de la page pour les exercices qui modifient le fichier.

**Pour essayer du JavaScript :** ouvre l'application dans le navigateur, appuie sur `F12` et choisis **Console**. Essaie les exemples un bloc à la fois. Si le navigateur demande de confirmer le collage, suis ses indications uniquement pour du code que tu comprends. Un exemple qui commence par `const` peut provoquer « already been declared » si tu le relances : recharge la page pour repartir à zéro. Cela efface les mesures chargées. Les blocs purement numériques ci-dessous n'ont pas besoin d'importer un fichier.

## 1. HTML : les éléments que tu vois à l'écran

### 1.1 Une balise : créer un élément

```html
<button type="button">Synchroniser</button>
```

`<button>` ouvre un bouton ; `</button>` le ferme. Entre les deux se trouve le texte visible. `type="button"` indique une commande ordinaire, pas une validation de formulaire. **À retenir :** le texte seul ne dit pas au navigateur quel calcul effectuer.

### 1.2 `id` et `class` : identifier ou regrouper

```html
<button type="button" id="ouvrirSync" class="compact">Synchroniser</button>
```

`id="ouvrirSync"` donne une identité unique à ce bouton : JavaScript peut le retrouver. `class="compact"` lui applique un style partagé avec d'autres commandes. **Un `id` ne doit apparaître qu'une fois dans la page ; une classe peut être utilisée plusieurs fois.** Changer le texte est simple ; changer l'`id` exige d'adapter le code qui le recherche.

### 1.3 `input` et `label` : demander une valeur

```html
<label for="seuilSync">Seuil de synchronisation</label>
<input type="number" id="seuilSync" value="5" step="any" required>
```

`input` crée le champ. `type="number"` demande un nombre, `value="5"` donne la valeur initiale, `step="any"` autorise les décimales et `required` rend la saisie obligatoire pour valider le formulaire. `for="seuilSync"` relie le texte au champ portant cet `id`. **La valeur saisie sera lue par JavaScript ; le HTML ne calcule pas le décalage.**

### 1.4 `input type="file"` : choisir un fichier

```html
<label for="fichierCsv">Charger un CSV CAN</label>
<input type="file" id="fichierCsv" accept=".csv">
```

Le navigateur ouvre un sélecteur de fichier. JavaScript récupère ensuite le fichier dans `files[0]`, c'est-à-dire le premier fichier choisi. `accept` aide à choisir la bonne extension, mais ne vérifie pas le contenu : un fichier nommé CSV peut quand même être incorrect.

### 1.5 `select` et `option` : proposer des choix

```html
<select id="famillePinces">
  <option value="I_BAT">I_BAT</option>
  <option value="I_AUX">I_AUX</option>
</select>
```

Chaque `option` est un choix. Son texte est visible ; sa `value` est la valeur récupérée par le programme. Dans SEED, ces options sont fabriquées après l'importation : on ne connaît pas les familles à l'avance. Pour choisir plusieurs voies à afficher, le sélecteur `variables` utilise aussi l'attribut `multiple`.

### 1.6 `disabled` et `hidden` : bloquer ou masquer

```html
<button type="button" id="fusionnerPinces" disabled>Fusionner les calibres</button>
<p id="messageExercice" hidden>Le traitement est terminé.</p>
```

Le bouton `disabled` est visible mais inutilisable. Le paragraphe `hidden` est masqué. JavaScript peut retirer ces états lorsque les données sont prêtes. **Attention :** `disabled="false"` désactive quand même le bouton, car c'est la présence de l'attribut qui compte. Pour l'activer depuis le code, on écrit `.disabled = false`.

### 1.7 `form` et `dialog` : valider un petit formulaire

Un `dialog` est une boîte de dialogue ; un `form` regroupe les champs à valider. Un bouton `type="submit"` déclenche sa validation. Dans SEED, JavaScript reçoit cet événement, vérifie les paramètres et empêche le rechargement de page avec `preventDefault()`. C'est le mécanisme utilisé pour la synchronisation et la correction unitaire.

### Exercice HTML : changer sans casser

Dans la copie de la page, cherche le bouton `fusionnerPinces`. Remplace seulement son texte par « Fusionner cette famille ». Recharge la copie et importe des données comme d'habitude. **Résultat attendu :** le libellé change ; les conditions d'activation et le calcul restent identiques, parce que l'`id` n'a pas changé.

## 2. JavaScript : manipuler les données et déclencher les actions

### 2.1 `const` et `let` : donner un nom à une valeur

```javascript
const coefficientExemple = 100;
let signeExemple = 1;
signeExemple = -1;
console.log(coefficientExemple, signeExemple);
```

Résultat : `100 -1`. `const` interdit de réaffecter le nom ; `let` autorise cette réaffectation. On utilise `const` par défaut, et `let` quand la valeur doit être remplacée. Une nuance : un tableau déclaré avec `const` peut recevoir de nouveaux éléments ; c'est remplacer le tableau par un autre qui est interdit.

### 2.2 Tableau et objet : stocker une série et décrire une voie

```javascript
const voieExemple = {
  nom: "I_BAT_100",
  unite: "A",
  valeurs: [10, 20, null, 40]
};
console.log(voieExemple.nom);
console.log(voieExemple.valeurs[1]);
```

Résultats : `I_BAT_100`, puis `20`. Un **objet** rassemble des informations nommées ; un **tableau** rassemble une suite ordonnée. `voieExemple.valeurs` accède au tableau de la voie. Les positions commencent à **zéro** : `[0]` est le premier élément, `[1]` le deuxième. Dans SEED, la même position doit représenter le même instant dans toutes les voies.

### 2.3 `function`, paramètres et `return` : écrire une recette réutilisable

```javascript
function corrigerExemple(valeur, offset, gain){
  return (valeur - offset) * gain;
}
console.log(corrigerExemple(12, 2, 3));
```

Résultat : `30`. La fonction reçoit trois **paramètres**, effectue le calcul et renvoie le résultat avec `return`. La définir n'exécute pas encore le calcul ; `corrigerExemple(12, 2, 3)` l'appelle. `console.log` affiche le résultat pour toi, mais ne le stocke pas dans une voie.

### 2.4 `if`, comparaisons et logique : décider

```javascript
const nombreCalibresExemple = 1;
if(nombreCalibresExemple >= 2){
  console.log("Fusion proposée");
}else{
  console.log("Pince seule : pas de fusion");
}
```

Résultat : `Pince seule : pas de fusion`. `>=` veut dire « supérieur ou égal ». `===` compare deux valeurs sans convertir leur type ; `=` affecte une valeur, ce n'est pas une comparaison. `&&` signifie « et », `||` signifie « ou », `!` signifie « non ». Dans `if(!fichier) return;`, on quitte la fonction si aucun fichier n'a été choisi.

### 2.5 `null`, `Number.isFinite` et `??` : ne pas inventer une mesure

```javascript
console.log(Number.isFinite(0));
console.log(Number.isFinite(null));
console.log(null ?? 0);
console.log(0 ?? 99);
```

Résultats : `true`, `false`, `0`, `0`. `null` désigne une mesure absente ; zéro est une vraie valeur. `Number.isFinite` reconnaît un nombre valide et fini. `??` fournit un remplacement seulement si la valeur est `null` ou `undefined`. Le remplacer par `||` change le comportement : `0 || 99` donne 99. **Ne remplace pas les absences par zéro sans raison métier.**

### 2.6 `=>` : une petite fonction écrite plus court

```javascript
const doublerExemple = valeur => valeur * 2;
console.log(doublerExemple(7));
```

Résultat : `14`. Lis la flèche comme « reçoit une valeur et renvoie son double ». Avec une expression directement après `=>`, le retour est automatique. Avec un bloc `{ ... }`, il faut écrire `return` pour renvoyer une valeur. Cette écriture sert beaucoup dans les opérations sur tableaux ci-dessous.

### 2.7 `map` : transformer chaque point sans changer la longueur

```javascript
const brutsExemple = [12, null, 22];
const corrigesExemple = brutsExemple.map(valeur =>
  valeur === null ? null : (valeur - 2) * 3
);
console.log(corrigesExemple);
```

Résultat : `[30, null, 60]`. `map` crée un nouveau tableau en appliquant une fonction à chaque élément. `condition ? résultatSiVrai : résultatSiFaux` est un petit `if` écrit sur une expression. Ici, l'absence reste une absence. **Dans SEED :** correction, inversion de signe et création de voies utilisent ce principe. La longueur reste la même, donc l'alignement temporel est préservé.

### 2.8 `filter` : ne garder que certains éléments

```javascript
const valeursFiltreesExemple = [10, null, 20].filter(Number.isFinite);
console.log(valeursFiltreesExemple);

const famillesExemple = [
  { nom: "I_BAT", voies: [100, 1000] },
  { nom: "I_AUX", voies: [50] }
];
const fusionnablesExemple = famillesExemple.filter(famille => famille.voies.length >= 2);
console.log(fusionnablesExemple.map(famille => famille.nom));
```

Résultats : `[10, 20]`, puis `["I_BAT"]`. `filter` conserve les éléments pour lesquels la condition est vraie ; `.length` donne le nombre d'éléments. C'est le principe de notre correction sur les pinces seules. **Attention :** filtrer les points d'une seule voie raccourcit son tableau et casse son alignement avec les dates. Filtrer pour calculer une statistique temporaire est différent de remplacer la série active.

### 2.9 `reduce` : transformer une liste en un résultat unique

```javascript
const pointsMoyenneExemple = [10, null, 20].filter(Number.isFinite);
const sommeExemple = pointsMoyenneExemple.reduce((total, valeur) => total + valeur, 0);
const moyenneExemple = pointsMoyenneExemple.length ? sommeExemple / pointsMoyenneExemple.length : null;
console.log(sommeExemple, moyenneExemple);
```

Résultat : `30 15`. Le `0` final est le total de départ. On ajoute 10, puis 20, puis on divise par le nombre de points valides. Le test de longueur évite une division par zéro. **Dans SEED :** cette opération sert notamment aux offsets et aux moyennes. Souviens-toi : `map` transforme chaque élément, `filter` choisit les éléments, `reduce` les rassemble en un résultat.

### 2.10 `find`, `some` et `sort` : chercher et ordonner

```javascript
const calibresExemple = [1000, 100, 50];
console.log(calibresExemple.find(calibre => calibre > 90));
console.log(calibresExemple.some(calibre => calibre === 50));
console.log(calibresExemple.slice().sort((avant, apres) => avant - apres));
```

Résultats : `1000`, `true`, puis `[50, 100, 1000]`. `find` renvoie le premier élément correspondant, ou `undefined` ; `some` dit seulement s'il en existe un. `sort` trie **le tableau sur lequel il est appelé**, d'où la copie avec `slice` dans cet exemple. Pour des nombres, donne une comparaison numérique : sans cela, le tri est fondé sur leur représentation en texte. SEED trie les pinces avant de les fusionner.

### 2.11 `slice` et `...` : copier sans mélanger les sauvegardes

```javascript
const sourceCopieExemple = { nom: "I_BAT", valeurs: [10, 20, 30] };
const copieExemple = { ...sourceCopieExemple, valeurs: sourceCopieExemple.valeurs.slice() };
copieExemple.valeurs[0] = 99;
console.log(sourceCopieExemple.valeurs[0]);
console.log(sourceCopieExemple.valeurs.slice(1, 3));
```

Résultats : `10`, puis `[20, 30]`. `slice()` copie un tableau ; `slice(1, 3)` extrait les positions 1 et 2, **sans inclure la position 3**. `...sourceCopieExemple` copie les propriétés de l'objet, mais ne copie pas les tableaux qu'il contient : le second `slice()` est indispensable ici. Dans SEED, ces copies protègent les valeurs brutes et les données avant synchronisation.

### 2.12 `for` : refaire une action pour chaque point

```javascript
const pointsBoucleExemple = [4, 8, 12];
for(let position = 0; position < pointsBoucleExemple.length; position++){
  console.log(position, pointsBoucleExemple[position]);
}
```

Résultats : `0 4`, `1 8`, `2 12`. Lis la ligne ainsi : « commence à zéro ; continue tant que la position est dans le tableau ; avance d'une position après chaque tour ». `position++` ajoute 1. C'est utile quand le calcul doit comparer le point courant au précédent, comme la détection d'un franchissement de seuil.

### 2.13 `Map` et `Set` : retrouver par un nom et éviter les doublons

```javascript
const offsetsExemple = new Map();
offsetsExemple.set("I_BAT_100", 0.2);
console.log(offsetsExemple.get("I_BAT_100"));
console.log(offsetsExemple.has("I_AUX_50"));

const selectionExemple = new Set([3, 3, 7]);
console.log([...selectionExemple]);
```

Résultats : `0.2`, `false`, puis `[3, 7]`. Une `Map` associe une clé à une valeur ; ici, un nom de voie à son offset. `set` écrit, `get` lit, `has` vérifie la présence. Un `Set` stocke chaque valeur une seule fois : pratique pour les identifiants sélectionnés. `[...]` avec `...` transforme ici le contenu du `Set` en tableau.

### 2.14 Comprendre la vraie ligne qui exclut les pinces seules

Dans `famillesPinces`, la nouvelle ligne est :

```javascript
const famillesReellesExemple = new Map([
  ["I_BAT", [{ nom: "I_BAT_100" }, { nom: "I_BAT_1000" }]],
  ["I_AUX", [{ nom: "I_AUX_50" }]]
]);
const resultatFamillesExemple = new Map(
  [...famillesReellesExemple].filter(([, colonnes]) => colonnes.length >= 2)
);
console.log([...resultatFamillesExemple.keys()]);
```

Résultat : `["I_BAT"]`. Décompose sans chercher à tout lire d'un coup : `[...]` transforme la `Map` en une liste de paires `[racine, colonnes]` ; `filter` garde les paires dont la liste de voies contient au moins deux éléments ; `new Map` reconstruit une table avec ces seules familles. `([, colonnes])` signifie « ignore le premier élément de la paire et donne le nom `colonnes` au second ». C'est un raccourci de lecture, pas une opération mystérieuse.

### 2.15 `split`, `trim`, `replace` et les expressions régulières : lire du texte

```javascript
const nomTexteExemple = " I_BAT_100 ".trim();
const morceauxExemple = nomTexteExemple.split("_");
console.log(morceauxExemple);
console.log(Number(morceauxExemple[morceauxExemple.length - 1]));
console.log("VITESSE BANC".replace(/\s+/g, "_"));
```

Résultats : `["I", "BAT", "100"]`, `100`, puis `VITESSE_BANC`. `trim` retire les blancs aux extrémités ; `split` découpe ; `Number` convertit en nombre ; `replace` remplace un motif. `/\s+/g` signifie « tous les groupes de blancs ». Une expression régulière sert aussi à reconnaître la forme `I_racine_calibre` ; au début, apprends surtout à identifier son rôle. Pour un CSV avec guillemets, un simple `split` ne suffit pas : SEED utilise `champsCsv`.

### 2.16 `getElementById`, `$` et les propriétés : lire ou changer l'écran

```javascript
const champSeuilExemple = document.getElementById("seuilSync");
console.log(champSeuilExemple.value);
console.log(champSeuilExemple.valueAsNumber);
```

Sur la page SEED avec le seuil initial, les deux affichent 5, mais `value` est du **texte**, tandis que `valueAsNumber` est un **nombre**. Un champ vide peut donner `NaN` avec `valueAsNumber` : vérifie avec `Number.isFinite`. Le raccourci `$` de l'application fait la même recherche que `document.getElementById` ; ce n'est pas une bibliothèque externe.

Les propriétés courantes sont `textContent` pour écrire un message, `disabled` pour bloquer une commande et `hidden` pour masquer un bloc. Par exemple, dans la copie de la page, `document.getElementById("etat").textContent = "Mon premier message"` change le message d'état. Cela change l'écran, pas les mesures.

### 2.17 `addEventListener` : dire quoi faire au clic

```javascript
function signalerClicExemple(){
  console.log("Le bouton de fusion a reçu un clic.");
}
document.getElementById("fusionnerPinces").addEventListener("click", signalerClicExemple);
```

Après cet essai, un clic sur le bouton **activé** affiche le message, en plus de son action habituelle. Recharge ensuite pour retirer cet écouteur d'exercice. `click` signifie clic, `change` changement de valeur ou de fichier, `submit` validation de formulaire. On transmet `signalerClicExemple` **sans parenthèses** : on veut l'appeler au futur clic, pas l'exécuter maintenant. Voilà pourquoi ajouter un bouton HTML seul ne suffit pas à lui donner une action.

### 2.18 `async`, `await`, `try` et `catch` : attendre et gérer l'échec

```javascript
async function lirePetitFichierExemple(fichier){
  try{
    const texte = await fichier.text();
    console.log(texte);
  }catch(erreur){
    console.log("Lecture impossible :", erreur.message);
  }
}
```

Cet exemple définit une fonction ; il ne choisit pas de fichier tout seul. `await` attend le texte avant de passer à la ligne suivante ; `async` autorise cette attente dans la fonction. `try` contient l'opération risquée et `catch` reçoit son éventuelle erreur. SEED utilise une lecture par blocs avec `stream()` et `TextDecoder` plutôt que `text()` pour les gros fichiers. Retenir ce principe suffit pour commencer ; ne remplace pas son lecteur par cet exemple simplifié.

### 2.19 `Date`, `Math.abs` et les unités : éviter les erreurs de facteur

```javascript
const debutTempsExemple = Date.parse("2026-10-07T10:00:00");
const finTempsExemple = Date.parse("2026-10-07T10:00:03");
console.log((finTempsExemple - debutTempsExemple) / 1000);
console.log(Math.abs(-95) > 90);
```

Résultats : `3`, puis `true`. `Date.parse` produit des millisecondes ; diviser par 1000 donne des secondes. `Math.abs` retire le signe : une amplitude de -95 A dépasse aussi un seuil de 90 A. C'est utilisé pour fusionner les calibres dans les deux sens de courant. Avant un calcul, note les unités des valeurs : secondes ou millisecondes, km/h ou m/s, signal brut ou ampères.

### 2.20 Plotly : utiliser une bibliothèque plutôt que dessiner les courbes

SEED prépare des **traces**, avec `x` pour les dates et `y` pour les valeurs, puis les transmet à `Plotly.react`. `Plotly.relayout` change la plage visible ; `Plotly.purge` nettoie le graphique. Tu n'as pas besoin de savoir dessiner une courbe point par point : ton travail est surtout de donner des tableaux alignés et de choisir les axes. Les réglages du graphique se trouvent dans `tracer`, pas seulement dans le CSS.

### Exercice JavaScript : corriger trois points

Sans exécuter le code, prédis le résultat de `[5, null, 15].map(valeur => valeur === null ? null : (valeur - 5) * 2)`. Puis essaie dans la console. **Correction :** `[0, null, 20]`. Tu viens d'utiliser la même combinaison d'outils que la correction des voies : tableau, petite fonction, test d'absence et calcul.

### Exercice JavaScript : aligner deux fichiers

Dans la console de la page SEED, appelle `premierFranchissement([0, 2, 6, 8], 5)` : **résultat attendu 2**, la position du premier point qui passe au-dessus du seuil. Puis appelle `plageSynchronisee(200, 180, 100, 70)` : **résultat attendu** `{ debutMesure: 30, debutCsv: 0, longueur: 170 }`. On retire 30 points au banc, puis on garde les 170 points communs. Ces appels ne modifient pas tes mesures.

## 3. CSS : rendre l'écran lisible sans modifier les calculs

### 3.1 Sélecteur, propriété et valeur : la forme d'une règle

```css
.compact{
  padding:6px 12px;
  font-size:12px;
}
```

`.compact` sélectionne les éléments portant cette classe. `padding` ajoute de l'espace à l'intérieur : 6 px en haut et en bas, 12 px à gauche et à droite. `font-size` fixe la taille du texte. **Lire une règle CSS, c'est demander : quels éléments ? quelle propriété ? quelle valeur ?**

### 3.2 `.classe`, `#id` et nom de balise : choisir la cible

`button` cible tous les boutons ; `.compact` cible une classe ; `#monGraphe` cible un élément précis. Pour changer une seule zone, commence par son identifiant ou une classe dédiée. Modifier `button` peut toucher toutes les commandes, même celles auxquelles tu ne pensais pas.

### 3.3 `margin`, `padding`, `border` et `box-sizing` : comprendre l'espace

```css
.carteExercice{
  margin-bottom:18px;
  padding:20px;
  border:1px solid #D9D9D6;
  box-sizing:border-box;
}
```

`padding` est l'espace **dedans**, entre contenu et bordure ; `margin` est l'espace **dehors**, entre éléments. `border` dessine la limite. `border-box` fait compter padding et bordure dans la largeur annoncée : pratique pour éviter qu'un bloc de largeur 100 % déborde. Cette classe d'exemple ne s'applique que si un élément HTML la porte.

### 3.4 Variables CSS : changer une couleur à un seul endroit

```css
:root{
  --couleurExercice:#EFDF00;
}
.boutonExercice{
  background:var(--couleurExercice);
}
```

`--couleurExercice` est une valeur nommée ; `var(...)` l'utilise. SEED fait cela avec sa palette pour éviter de recopier les couleurs partout. Une variable CSS n'est pas une variable JavaScript : elles appartiennent à deux langages différents. Modifier la palette CSS ne change pas automatiquement les couleurs définies dans Plotly.

### 3.5 Flexbox : ranger une ligne de commandes

```css
.ligneExercice{
  display:flex;
  gap:12px;
  flex-wrap:wrap;
  align-items:center;
}
```

`display:flex` organise les enfants en ligne par défaut ; `gap` les espace ; `wrap` permet de passer à la ligne quand la place manque ; `align-items:center` les aligne verticalement. C'est le principe de `.row` dans SEED. Quand les boutons débordent, vérifie d'abord si le retour à la ligne est autorisé.

### 3.6 Grid : partager l'écran entre variables et graphique

```css
.workspaceGrid{
  display:grid;
  grid-template-columns:minmax(280px, 1fr) minmax(0, 2fr);
  gap:18px;
}
```

Deux colonnes sont créées. `1fr` et `2fr` demandent approximativement une part pour la liste et deux pour le graphique, sous réserve de l'espace disponible et du minimum de 280 px. `minmax(0, 2fr)` permet à la seconde colonne de rétrécir. **Flexbox organise surtout une rangée ; Grid organise ici les grandes zones de l'écran.**

### 3.7 `overflow` et `min-width` : maîtriser les noms trop longs

`overflow:auto` propose un défilement quand le contenu déborde. `min-width:0` autorise un élément de grille ou de Flexbox à rétrécir malgré un contenu long. `white-space:nowrap` empêche le texte de passer à la ligne. Dans SEED, ces outils permettent de garder les noms de voies complets sans élargir toute la page. Réduire la police n'est pas toujours la bonne réponse à un débordement.

### 3.8 `:hover`, `:disabled` et `:focus-visible` : montrer l'état d'une commande

`button:hover` concerne le survol ; `button:disabled` le bouton désactivé ; `:focus-visible` indique l'élément actif au clavier. Ce sont des **états**, pas des classes à écrire dans le HTML. Une couleur grise seule ne désactive pas un bouton : c'est l'attribut HTML ou la propriété JavaScript `disabled` qui le fait.

### 3.9 `@media` : adapter l'écran aux petites fenêtres

```css
@media(max-width:800px){
  .workspaceGrid{
    grid-template-columns:minmax(0, 1fr);
  }
}
```

Lis cela comme : « si la fenêtre fait au plus 800 px de large, utilise une seule colonne ». Les données et calculs ne changent pas ; les zones sont simplement empilées. Pour vérifier, réduis la fenêtre du navigateur : tu dois encore pouvoir lire les noms, ouvrir les dialogues et atteindre les boutons.

### 3.10 Cascade : pourquoi ma règle semble ne rien faire ?

Plusieurs règles peuvent viser le même élément. Une règle plus précise peut gagner ; à précision égale, celle écrite plus tard gagne généralement. D'autres mécanismes existent, notamment `!important`, utilisé dans SEED pour `[hidden]`. Pour débuter, fais clic droit sur l'élément puis **Inspecter** : les styles barrés ont perdu. Ne rajoute pas `!important` partout ; cherche d'abord la règle qui contrôle vraiment l'élément.

### Exercice CSS : changer une dimension, pas un calcul

Dans le bloc `<style>` de la copie de la page, cherche `.plot` et remplace sa hauteur de 600 px par 500 px. Recharge et trace une voie. **Résultat attendu :** le graphique est moins haut, mais les valeurs et le nombre de points sont identiques. Sur une fenêtre étroite, une règle `@media` peut encore imposer 460 px : c'est un cas concret de règle conditionnelle à regarder dans l'inspecteur.

### Ton petit protocole pour modifier seul

1. **Nomme le changement souhaité.** « Ne pas proposer les pinces seules » est plus précis que « améliorer la fusion ».
2. **Trouve le point de départ.** Texte ou structure : HTML. Apparence : CSS. Données, calcul ou action : JavaScript.
3. **Change une seule chose.** Garde une copie et évite les remplacements globaux si tu ne sais pas qui utilise un nom.
4. **Prédis un résultat simple.** Une pince seule doit disparaître de la liste ; deux voies d'une même famille doivent rester.
5. **Vérifie ce résultat et une limite.** Essaie aussi sans donnée, avec une valeur absente ou avec un champ vide, selon la modification.

Le premier cap à viser n'est pas « écrire toute l'application ». C'est **lire une fonction courte, expliquer ses entrées et sa sortie, puis vérifier une petite modification**. Commence par `famillesPinces`, `premierFranchissement` et `plageSynchronisee` ; reviens ensuite au [guide des compétences](COMPETENCES_SEED.md) pour les traitements complets.