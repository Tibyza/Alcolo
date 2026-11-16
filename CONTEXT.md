# Alcolo — contexte du projet

Ce document donne tout le contexte nécessaire pour reprendre le développement d'Alcolo, que ce soit dans une nouvelle conversation ou avec une autre IA, sans rien réexpliquer.

## En bref

Alcolo est une appli web de jeux à boire entre potes, créée par Rémi. Une seule personne tient le téléphone et mène la partie, les autres jouent autour de la table. Elle est hébergée gratuitement :

- le site est servi par **Cloudflare Pages**, branché sur ce dépôt GitHub ;
- le mode spectateur passe par **Firebase Realtime Database**.

| | |
|---|---|
| Production | https://alcolo.pages.dev (branche `main`) |
| Test | aperçu Cloudflare de la branche `test` (onglet Deployments du projet) |
| Dépôt | https://github.com/Tibyza/Alcolo |
| Firebase | projet `alcolo-21d86`, Realtime Database en `europe-west1` |

## Règles de travail (à respecter absolument)

1. **Toute modification se fait sur la branche `test`.** Cloudflare publie automatiquement un aperçu, que Rémi teste sur son téléphone.
2. **Rien ne passe sur `main` sans la validation explicite de Rémi**, par exemple « passe test sur main ». Comme `test` part de `main`, le passage est une simple avance rapide.
3. Pas de build, pas de framework, pas de dépendance npm : tout tient dans un seul fichier `index.html`, publié tel quel.
4. Rémi n'est pas développeur. Il faut lui expliquer simplement ce qui a changé et comment le tester, en français.

## Principes de design (non négociables)

- **Pensé pour des joueurs bourrés.** Gros boutons, gros textes, contrastes forts. Une action courante doit se faire en **2 taps maximum**.
- **Pas de score, pas de classement, pas de points.** On s'en fiche, l'appli sert à faire boire, pas à compter.
- **Rien ne doit faire « généré par une IA ».** Le style vise un tapis de jeu ou de casino vivant et des cartes qui ressemblent à de vraies cartes. On évite les dégradés violets génériques, les emojis partout et les textes creux.
- **Responsive** sur iPhone, iPad, en portrait comme en paysage. Les grandes cartes sont dimensionnées avec `--vh` (= `innerHeight`), pour que les boutons restent toujours visibles sans défiler.
- **Animations courtes qui créent du suspense.** Pour révéler une carte, la carte suivante tremble dans la pyramide ou le plateau, puis s'envole en plein écran et se retourne (`flyReveal`). Un glissement vers le bas ferme la vue plein écran.
- **Sons doux** synthétisés en WebAudio (swoosh, cloche, notes douces, grelots à Noël), sans fichier audio. Vibrations sur mobile. Le son peut se couper.
- **Les fenêtres et vues plein écran se ferment** par un bouton en haut ou par un glissement vers le bas.
- Le clavier se ferme avec Entrée sur un champ vide, en touchant ailleurs ou en faisant défiler.

## Les modes de jeu

L'accueil propose trois modes. Choisir un mode change le titre et l'aperçu en haut de l'accueil, et c'est seulement à ce moment-là que ses réglages de partie apparaissent. Sans mode choisi, l'accueil affiche « Alcolo » et son logo. On peut avoir de 1 à 20 joueurs (`MAXP`).

### 1. La Pyramide (`classic`)
- Chaque joueur reçoit une main secrète. Au début de la partie, chacun la mémorise à tour de rôle.
- Une pyramide de cartes face cachée vaut 1, 2, 3… gorgées par rangée en partant du bas (doublées en option).
- À chaque carte retournée, celui qui a la même valeur distribue les gorgées à voix haute. Il peut aussi **bluffer**.
- **Bluff ?** On touche le nom du joueur : l'appli montre seulement la carte concernée, jamais toute sa main. Si le joueur bluffait, il boit le double. S'il disait vrai, c'est l'accusateur qui boit le double.
- **Revoir ses cartes en cours de partie** (option, autorisé par défaut) :
  - il faut garder le doigt appuyé, et la vue dure **2 s** au maximum ;
  - ça coûte les gorgées de la rangée en cours ;
  - le joueur ne peut plus regarder pendant **2 min** (`PEEK_MS`, `PEEK_COOLDOWN`).
- **Le sommet** (option « Annonces + cul sec ») :
  - avant de retourner la dernière carte, chaque joueur annonce sa main de mémoire, puis la révèle en grand ;
  - la table valide avec « Oui, tout bon » ou « Non · cul sec » ;
  - ensuite on retourne le sommet, et ceux qui ont la valeur boivent cul sec.
- Réglages :
  - rangées : 4 à 9 ;
  - cartes en main : 3 à 5 ;
  - gorgées : normales ou doublées ;
  - sommet : annonces + cul sec, ou carte normale ;
  - revoir ses cartes : autorisé ou interdit.

### 2. Inter-Exter (`ie`)
- **Phase 1 : quatre tours de questions.** Les joueurs répondent chacun leur tour, en tour de table, une question par tour :
  1. Rouge ou noir ?
  2. Plus ou moins que ta 1re carte ?
  3. Inter ou exter de tes 2 premières ?
  4. Quel signe ?

  Une mauvaise réponse fait boire 1, 2, 3 puis 4 gorgées (doublées en option). L'as est toujours la carte la plus forte.
- **Phase 2 : le plateau.** Deux lignes, **Donne** et **Bois**. On retourne les cartes colonne par colonne, en alternant Donne puis Bois. La colonne n vaut n gorgées. Ceux qui ont la valeur donnent ou boivent, une fois par carte qu'ils possèdent. La dernière colonne vaut un **cul sec** (option).
- Réglages : cartes par ligne de 3 à 9, gorgées normales ou doublées, dernière colonne cul sec ou normale.

### 3. Cheucheuille (`chu`)
- Un jeu de 52 cartes posées en cercle autour d'un verre. Chacun pioche à son tour en touchant une carte, ou avec le bouton qui pioche au hasard.
- Ouvrir un **trou dans le cercle** (option) vaut un **cul sec**.
- Le dernier joueur à avoir tiré Valet, Dame ou 10 garde le pouvoir correspondant.
- Les cartes :
  - **As** : cul sec ;
  - **3** : « Dans ma valise il y a… » ;
  - **6** : le joueur d'avant boit 1 gorgée ;
  - **7** : tournée générale ;
  - **8** : le joueur d'après boit 1 gorgée ;
  - **10** : roi des pouces ;
  - **Valet** : « Cheucheuille ! », le dernier à imiter boit 2 ou 3 gorgées ;
  - **Dame** : Freeze, celui qui bouge boit 1 ou 2 gorgées ;
  - **Roi** : on verse ce qu'on veut dans le verre du milieu. Le **4e roi** boit le verre du milieu, ce qui termine la partie ;
  - autres cartes : rien.
- Réglages : gorgées du Valet (2 ou 3), gorgées de la Dame (1 ou 2), trou dans le cercle (cul sec ou pas de pénalité).

## Apparence

- **Roue crantée en haut à droite** (`appSettings`) : thème, design des cartes, sons.
- **Thèmes** (`THEMES`, attribut `body[data-felt]`) : vert, bleu, rouge et Noël.
  - Le thème Noël est pensé comme un vrai Noël, pas un « Noël de casino ».
  - Les encarts et boutons y sont des cadeaux avec ruban et nœud, sur un fond nuit avec sapin, boules et neige.
  - Les éléments `.xo` n'apparaissent qu'en thème Noël, et les éléments `.so` sont masqués en thème Noël.
- **Decks** (`DECKS`, attribut `body[data-deck]`) : Classique, Élégant, Moderne, Rétro et Noël.
  - Les cartes sont dessinées en CSS et SVG, avec le vrai nombre de symboles disposés comme sur de vraies cartes.
  - Les figures sont dessinées à la main et en miroir haut/bas.
  - Le deck Noël a un sapin au dos, le Père Noël en Roi, la Mère Noël en Dame, un renne en Valet et un cadeau pour l'As.
- Le choix du deck change aussi les cartes affichées sur l'accueil (`renderFan`).

## Mode spectateur (miroir)

- **Côté hôte :** au lancement d'une partie, l'appli génère un code de 4 caractères, affiché sur un bouton doré en haut (`#liveBtn`).
  - Un `MutationObserver` surveille `#game`, `#stage` et `#modal`, puis `livePush` envoie l'écran sérialisé dans `live/<CODE>`, avec un délai de 300 ms.
  - Les cartes deviennent de petits repères `<b data-pc="rang|signe|classes">` pour alléger l'envoi.
  - **La main secrète (`#hold`) n'est jamais envoyée.**
- **Côté spectateur :** l'accueil propose « Regarder une partie », puis le code.
  - Le spectateur reçoit une copie en lecture seule : `expand()` redessine les cartes, les attributs `on*` sont supprimés et tous les événements sont bloqués.
  - La classe `body.spectator` affiche une barre « Spectateur · Quitter ».
- **Base de données :** Firebase via le SDK compat 10.12.2 chargé depuis gstatic. La config est dans `firebase-config.js`, et un petit adaptateur imite l'API `doc(p).set/onSnapshot`.
  - Si la config est absente, l'appli se rabat sur la base des artefacts Claude.
  - Sans aucune des deux, le mode spectateur est désactivé proprement.
- **Règles Firebase :** voir `database.rules.json`. Seul `live/<CODE>` est accessible, avec un code de 4 caractères et un format d'écriture validé. Ces règles doivent être publiées dans la console Firebase, onglet Realtime Database → Règles.

## Organisation du code (`index.html`)

Tout est dans un seul fichier : le CSS, puis le HTML, puis un seul script.

| Zone | Éléments clés |
|---|---|
| Réglages et stockage | `DEF`, `S` (réglages), `saveS()`, `names`. localStorage : `pyr_settings`, `pyr_names` |
| Cartes | `face(c,cls,deck)`, `cardBody`, `courtSVG`, `modernCourt`, `noelCourt`, `noelAce`, `backCard` |
| Sons | `ac()`, `bell`, `soft`, `swooshAt`, objet `sfx`, `vib()` |
| Accueil | `renderSetup`, `setMode`, `settingsHTML`, `seg`, `setOpt`, `renderFan`, `appSettings`, `setFelt`, `setDeck`, `snow` |
| Partie (commun) | `start`, `render`, `next`, `flyReveal`, `bigCard`, `stage()`, `modal()`, `lock()` (anti double tap) |
| Pyramide | `startClassic`, `buildPyr`, `cFlip`, `showReveal`, `bluffPicker`, `liarCheck`, `bindHold`, `showHand`, `startAnnounce`, `annStep`, `annJudge`, `showSummit` |
| Inter-Exter | `startIE`, `renderQuiz`, `answer`, `nextPlayer`, `renderIEBoard`, `ieNext`, `showIEReveal` |
| Cheucheuille | `startChu`, `renderChu`, `chuDraw`, `chuRule`, `gapCount`, `showChuReveal` |
| Spectateur | `liveStart`, `livePush`, `liveStop`, `joinAsk`, `specStart`, `specRender`, `specStop` |

Les autres fichiers :
- `firebase-config.js` : config web Firebase (publique par nature, la sécurité repose sur les règles) ;
- `database.rules.json` : règles de sécurité de la base ;
- `manifest.webmanifest`, `icon-*.png` : installation sur l'écran d'accueil du téléphone (PWA) ;
- `README.md` : guide de mise en ligne.

## Pour tester

- **En local :** ouvrir `index.html` dans un navigateur suffit. Le mode spectateur demande une connexion internet.
- **Sur téléphone :** utiliser l'adresse d'aperçu de `test`. Tester en portrait et en paysage, avec plusieurs joueurs, et vérifier que les boutons principaux restent visibles sans défiler.
