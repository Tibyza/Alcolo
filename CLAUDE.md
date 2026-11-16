# Alcolo — instructions pour Claude Code

Lis d'abord **CONTEXT.md** : règles des 3 modes, principes de design, organisation du code, mode spectateur.

## Règles de travail
- Travaille **toujours sur la branche `test`**. Cloudflare Pages en publie un aperçu que Rémi teste sur son téléphone.
- **Ne pousse jamais sur `main`** sans que Rémi l'ait explicitement demandé, par exemple « passe test sur main ». Le passage se fait alors en avance rapide de `test` vers `main`.
- L'appli tient dans un seul fichier `index.html` (CSS, HTML et JS ensemble), sans build, sans framework et sans npm. Garde cette structure.
- Ne modifie pas `firebase-config.js` ni `database.rules.json` sans raison. Si les règles changent, rappelle à Rémi de les republier dans la console Firebase.
- Réponds à Rémi en français, simplement. Il n'est pas développeur : dis-lui ce qui a changé et comment le tester.

## Rappels de design
- Pensé pour des joueurs bourrés : gros boutons, 2 taps maximum par action.
- Pas de score affiché en gros. Seule exception, voulue par Rémi : en Inter-Exter et Cheucheuille, un classement des gorgées bues et données, visible uniquement dans une pop-up via un petit bouton podium.
- Rien qui fasse « généré par une IA ».
- Responsive iPhone, iPad, portrait et paysage. Les boutons principaux restent visibles sans défiler.
- Ne jamais envoyer la main secrète (`#hold`) au mode spectateur.

## Thèmes (voir CONTEXT.md › Apparence)
- Noël (thème d'accueil), Absinthe, Lagon et Spritz utilisent le style « soirée en verre » : bloc CSS `SOIRÉE EN VERRE`, sous `body:not([data-felt="casino"])`. Casino garde l'ancien style tapis vert et or (bloc `CASINO`).
- La couleur d'accent de chaque thème s'appelle toujours `--gold`/`--gold2`, avec `--onacc` pour le texte posé dessus. Pour un nouvel accent, utilise ces variables plutôt qu'une couleur écrite en dur.
- Après une modification, vérifie les 5 thèmes, en portrait et en paysage.
