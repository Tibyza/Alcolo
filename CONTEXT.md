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
- **Pas de score affiché en gros.** L'appli sert à faire boire, pas à compter. Seule exception, voulue par Rémi : en Inter-Exter et Cheucheuille, un bouton podium discret dans la barre du haut ouvre une pop-up de classement des gorgées bues et données.
- **Rien ne doit faire « généré par une IA ».** Le style vise un bar de nuit : surfaces en verre dépoli (« liquid glass ») posées sur des lumières de bar qui dérivent lentement et des bulles qui montent, avec des cartes qui ressemblent à de vraies cartes. On évite les dégradés violets génériques, les emojis partout et les textes creux.
- **Responsive** sur iPhone, iPad, en portrait comme en paysage. Les grandes cartes sont dimensionnées avec `--vh` (= `innerHeight`), pour que les boutons restent toujours visibles sans défiler.
- **Animations courtes qui créent du suspense.** Pour révéler une carte, la carte suivante tremble dans la pyramide ou le plateau, puis s'envole en plein écran et se retourne (`flyReveal`). Un glissement vers le bas ou le bouton « ‹ Revenir à la pyramide » (`pyrBtn`) ferme la vue plein écran.
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
- Au lancement, une fenêtre **Ordre de pioche** montre les joueurs dans l'ordre. On glisse un nom pour le déplacer (`chuOrder`, `dragStart`, `chuOrderOk`), et le nouvel ordre est enregistré dans la liste des joueurs.

### Pendant toutes les parties
- **Reprise** : l'état complet de la partie est enregistré à chaque étape dans le localStorage `pyr_game` (`getState`, `setState`, `saveGame`). À la réouverture de l'appli, `resumeAsk()` propose « Reprendre la partie » si une partie de moins de 12 h n'est pas terminée. La sauvegarde est effacée en fin de partie (`onGameOver`) et au retour à l'accueil (`toSetup`). Le code spectateur est conservé quand c'est possible (`LIVE.resume`).
- **Annuler** : avant chaque action importante (retourner une carte, répondre en Inter-Exter, piocher, vérifier un bluff, juger une annonce, ajouter ou retirer un joueur, « qui a perdu »), `snap()` garde l'état. Un bouton « Annuler » (`#undoBtn`, hors de `#game`, donc jamais envoyé aux spectateurs) reste 9 s et ramène un coup en arrière (`undo`).
- **Joueurs en cours de partie** (Cheucheuille, et Inter-Exter une fois au plateau) : `playersEdit`, `addMid`, `removeMid`. En Cheucheuille, le nouveau venu pioche en dernier. Toujours en Cheucheuille, on glisse un nom dans cette même fenêtre pour changer l'ordre de pioche (`moveMid`, avec la liste de `drawOrder` en mode `ORDMID`). Chacun garde ses compteurs et ses pouvoirs, et celui dont c'est le tour garde la main. En Inter-Exter, il reçoit 4 cartes au hasard.
- **Fin de partie** : `onGameOver()` lance au bout de 3 s une **pause eau** (`waterBreak`), avec un message tiré au hasard parmi 4 (`WATER`), jamais deux fois le même d'affilée. La barre de fin propose Accueil · **Récap** · Rejouer (`endDock`).
- **Récap** (`showRecap`, `recapTitles`) : des titres décalés (L'éponge, Le mécène, Le fantôme, Le plongeur, Le cocktail maison, Démasqué, Le suspect idéal, Le discret, Le héros de la soirée pour le SAM). « Partager » dessine une image au format story de 1080 × 1920 aux couleurs du thème (`shareRecap`), envoyée par le partage du téléphone ou téléchargée.

### Classement des gorgées (Inter-Exter et Cheucheuille)
- Le bouton podium `#rankBtn`, dans la barre du haut, ouvre la pop-up `showRank()`. On peut trier par « Qui a le plus bu » ou « Qui a le plus donné ». Les culs secs sont comptés à part.
- Les compteurs `ST[i] = {bu, cs, don, dcs}` sont remis à zéro à chaque partie (`resetStats`) et alimentés par `addSt` :
  - Inter-Exter : chaque réponse (bonne = donne, mauvaise = boit), puis chaque carte du plateau pour ceux qui ont la valeur.
  - Cheucheuille : As, trou et 4e Roi (cul sec), 6 et 8 (1 gorgée, comptée comme donnée par le tireur), 7 (1 gorgée pour tous). Pour la valise (bouton « Qui a perdu ? » sur la carte 3) et pour les pouvoirs Valet, Dame et 10 (fenêtre ouverte après le pouvoir), on touche le perdant : `lostAsk`, `lostPick`. Le perdant boit, et le détenteur du pouvoir « donne ».
  - Pyramide (pas de bouton podium, seulement pour le récap) : `bl` compte les bluffs démasqués, `inno` les accusations à tort, `cs` les culs secs (annonce ratée, sommet).
  - `ST[i]` contient `{bu, cs, don, dcs, bl, inno}` (`ZST()`).

## Apparence

- **Roue crantée en haut à droite** (`appSettings`) : thème, design des cartes, sons.
- **Démarrage sans flash** : un petit script placé juste après `<body>` lit `pyr_settings` et pose `data-felt`/`data-deck` sur la page avant tout affichage. Il faut le garder aligné avec la liste des thèmes et la règle `noel26`.
- **Thèmes** (`THEMES`, attribut `body[data-felt]`) : Noël (`xmas`), Absinthe (`green`), Lagon (`blue`), Spritz (`red`) et Casino (`casino`).
  - **Noël est le thème d'accueil** (`DEF.felt='xmas'`, deck `noel`). Les joueurs qui avaient déjà un autre thème y sont basculés une seule fois (drapeau `S.noel26`), puis leur choix est respecté.
  - **Absinthe, Lagon, Spritz et Noël** partagent le style « soirée en verre » : nuit (`--bg0`), 3 lumières qui dérivent (`#amb`, couleurs `--o1`, `--o2`, `--o3`), surfaces en verre dépoli. Pour des raisons historiques, l'accent s'appelle `--gold`/`--gold2`, avec `--onacc` pour le texte posé dessus. Les règles de ce style sont dans le bloc CSS `SOIRÉE EN VERRE`, sous `body:not([data-felt="casino"])`.
  - Absinthe, Lagon et Spritz ont en plus des bulles qui montent (`#fizz`, créées par `fizz()`).
  - **Noël** : nuit d'hiver, lumières rouge, dorée et sapin, accent doré, neige (`snow()`), guirlande lumineuse en haut de l'accueil (`garland()`, élément `.garland.xo`), givre léger sur le haut des encarts. Les anciens cadeaux, rubans, sapin, boules et chapeau ont été retirés. En Pyramide, la pyramide devient un **sapin** (`decorTree`, classe `.pyr.xtree`) : silhouette en étages, étoile au sommet, tronc, guirlandes lumineuses entre les rangées et boules sur les bords. Le dessin suit la position réelle des cartes et se refait quand l'écran change, y compris côté spectateur. Les éléments `.xo` n'apparaissent qu'en Noël et les `.so` y sont masqués.
  - **Casino** : le tapis vert et l'or d'origine, sans verre ni lumières (bloc CSS `CASINO`).
  - Lumières, bulles, neige et guirlande s'arrêtent si le téléphone demande moins d'animations. La couleur de la barre du téléphone suit le thème (`barColor`).
- **Decks** (`DECKS`, attribut `body[data-deck]`) : Classique, Élégant, Moderne, Rétro et Noël.
  - Les cartes sont dessinées en CSS et SVG, avec le vrai nombre de symboles disposés comme sur de vraies cartes.
  - Les figures sont dessinées à la main et en miroir haut/bas.
  - Le deck Noël a un sapin au dos, le Père Noël en Roi, la Mère Noël en Dame, un renne en Valet et un cadeau pour l'As.
- Le choix du deck change aussi les cartes affichées sur l'accueil (`renderFan`).
- **Boutons en verre** (style « liquid glass ») : fond translucide flouté, reflet en haut, liseré clair. Le bouton principal garde la classe `.btn.gold` : il est éclairé par la couleur d'accent du thème, et redevient doré en Casino. Rouge et vert restent pour Bluff / Non / Oui, en verre teinté.
- **Verre de chaque joueur** : à la place d'un rond coloré avec l'initiale, chaque joueur a une boisson (`GLASS_ORDER` : shot par défaut, bière, vin, champagne, cocktail, whisky), dessinée en SVG (`GLASS_SVG`, `drinkSVG`, `av`). Les boissons **Soft** et **SAM** (logo voiture, pour celui qui conduit) sont sans alcool : `isSoft(i)`. Une petite mention `softTag(i)` (« soft » ou « SAM · soft ») s'affiche à côté du prénom quand il doit boire. On change la boisson en touchant le joueur sur l'accueil : un petit encart s'ouvre avec les 6 boissons (`glassPick`, `setGlass`, élément `#gpop`). Elle est retenue par prénom dans le localStorage `pyr_glasses`.

## Mode spectateur (miroir)

- **Côté hôte :** au lancement d'une partie, l'appli génère un code de 4 caractères, affiché sur un bouton doré en haut (`#liveBtn`).
  - Un `MutationObserver` surveille `#game`, `#stage` et `#modal`, puis `livePush` envoie l'écran sérialisé dans `live/<CODE>`, avec un délai de 300 ms.
  - Les cartes deviennent de petits repères `<b data-pc="rang|signe|classes">` pour alléger l'envoi.
  - **La main secrète (`#hold`) n'est jamais envoyée.**
- **Côté spectateur :** l'accueil propose « Regarder une partie », puis le code.
  - Le spectateur reçoit une copie en lecture seule : `expand()` redessine les cartes, les attributs `on*` sont supprimés et tous les événements sont bloqués.
  - La classe `body.spectator` affiche une barre « Spectateur · Quitter ».
- **Base de données :** Firebase via le SDK compat 10.12.2 chargé depuis gstatic, **en fin de page** pour ne pas retarder l'affichage. `initLDB()` est appelé une fois les scripts chargés. Si une partie a déjà démarré, elle récupère alors son code spectateur. La config est dans `firebase-config.js`, et un petit adaptateur imite l'API `doc(p).set/onSnapshot`.
  - Si la config est absente, l'appli se rabat sur la base des artefacts Claude.
  - Sans aucune des deux, le mode spectateur est désactivé proprement.
- **Règles Firebase :** voir `database.rules.json`. Seul `live/<CODE>` est accessible, avec un code de 4 caractères et un format d'écriture validé. Ces règles doivent être publiées dans la console Firebase, onglet Realtime Database → Règles.

## Organisation du code (`index.html`)

Tout est dans un seul fichier : le CSS, puis le HTML, puis un seul script.

| Zone | Éléments clés |
|---|---|
| Réglages et stockage | `DEF`, `S` (réglages), `saveS()`, `names`, `GLASSES`. localStorage : `pyr_settings`, `pyr_names`, `pyr_glasses` |
| Cartes | `face(c,cls,deck)`, `cardBody`, `courtSVG`, `modernCourt`, `noelCourt`, `noelAce`, `backCard` |
| Sons | `ac()`, `bell`, `soft`, `swooshAt`, objet `sfx`, `vib()` |
| Accueil | `renderSetup`, `setMode`, `settingsHTML`, `seg`, `setOpt`, `renderFan`, `appSettings`, `setFelt`, `setDeck`, `glassPick`, `setGlass` |
| Ambiance | `fizz` (bulles), `snow` (neige), `garland` (guirlande de Noël), `barColor` (couleur de la barre du téléphone), `decorTree` (sapin de la Pyramide en Noël) |
| Partie (commun) | `start`, `render`, `next`, `flyReveal`, `bigCard`, `stage()`, `modal()`, `lock()` (anti double tap) |
| Pyramide | `startClassic`, `buildPyr`, `decorTree`, `cFlip`, `showReveal`, `bluffPicker`, `liarCheck`, `bindHold`, `showHand`, `startAnnounce`, `annStep`, `annJudge`, `showSummit` |
| Inter-Exter | `startIE`, `renderQuiz`, `answer`, `nextPlayer`, `renderIEBoard`, `ieNext`, `showIEReveal` |
| Classement | `ST`, `ZST`, `resetStats`, `addSt`, `showRank`, bouton `#rankBtn` |
| Reprise, annulation, fin | `getState`, `setState`, `saveGame`, `resumeAsk`, `resumeGame`, `snap`, `undo`, `onGameOver`, `waterBreak`, `showRecap`, `shareRecap`, `playersEdit`, `lostAsk` |
| Cheucheuille | `startChu`, `chuOrder`, `dragStart`, `chuOrderOk`, `renderChu`, `chuDraw`, `chuRule`, `gapCount`, `showChuReveal` |
| Spectateur | `liveStart`, `livePush`, `liveStop`, `joinAsk`, `specStart`, `specRender`, `specStop` |

Les autres fichiers :
- `firebase-config.js` : config web Firebase (publique par nature, la sécurité repose sur les règles) ;
- `database.rules.json` : règles de sécurité de la base ;
- `manifest.webmanifest`, `icon-*.png` : installation sur l'écran d'accueil du téléphone (PWA). L'icône montre un as de cœur, un dos de carte et un shot sur fond de nuit. Pour la refaire, il faut repartir d'une source en 1024 px ;
- `og-image.jpg` : vignette affichée quand on partage le lien (WhatsApp, Messenger, iMessage…), déclarée par les balises `og:` dans `<head>` ;
- `README.md` : guide de mise en ligne.

## Pour tester

- **En local :** ouvrir `index.html` dans un navigateur suffit. Le mode spectateur demande une connexion internet. Pour les tests automatiques de la reprise, mieux vaut servir le dossier par un petit serveur (`python3 -m http.server`) plutôt qu'en `file://` : le stockage du navigateur de test y est peu fiable d'un rechargement à l'autre.
- **Sur téléphone :** utiliser l'adresse d'aperçu de `test`. Tester en portrait et en paysage, avec plusieurs joueurs, et vérifier que les boutons principaux restent visibles sans défiler.
- **Tester chaque thème** (Noël, Absinthe, Lagon, Spritz, Casino), car le style Casino est séparé des autres. Vérifier aussi que la page ne dépasse jamais en largeur, surtout la barre du haut quand le code spectateur s'affiche.

## Historique des évolutions

- **Boissons des joueurs** : un verre par joueur (shot, bière, vin, champagne, cocktail, whisky) à la place des ronds colorés, choisi dans un petit encart.
- **Refonte « soirée en verre »** : thèmes Absinthe, Lagon et Spritz, avec surfaces en verre, lumières de bar et bulles.
- **Thème Casino** : l'ancien style tapis vert et or, gardé tel quel.
- **Noël refait** : style en verre, nuit d'hiver, guirlande, neige, sapin en Pyramide. C'est le thème d'accueil.
- **Bouton « Revenir à la pyramide »** sur la vue d'une carte en grand.
- **Classement discret** des gorgées en Inter-Exter et Cheucheuille.
- **Ordre de pioche** réglable au début du Cheucheuille.
- **Correctif** : la barre du haut ne déborde plus sur iPhone quand le code spectateur s'affiche.
- **Correctif** : plus de flash vert au démarrage (thème posé dès l'ouverture, Firebase chargé en fin de page).
- **Nouvelle icône d'appli** et **vignette de partage du lien**.
- **Reprise de partie**, bouton **Annuler**, **joueurs en cours de partie**.
- Boissons **Soft** et **SAM**, **pause eau** en fin de partie, **récap** avec image à partager, « **Qui a perdu ?** » au Cheucheuille.
