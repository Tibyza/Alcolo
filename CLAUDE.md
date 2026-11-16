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
- Pas de score ni de classement.
- Rien qui fasse « généré par une IA ».
- Responsive iPhone, iPad, portrait et paysage. Les boutons principaux restent visibles sans défiler.
- Ne jamais envoyer la main secrète (`#hold`) au mode spectateur.
