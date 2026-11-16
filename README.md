# Alcolo

Jeux à boire pour toute la table : **La Pyramide**, **Inter-Exter**, **Cheucheuille**, avec un mode spectateur en direct.

L'appli est une seule page web (`index.html`). Hébergement gratuit sur Cloudflare Pages, mode spectateur sur Firebase.

## Contenu du dossier

| Fichier | Rôle |
|---|---|
| `index.html` | Tout le jeu |
| `firebase-config.js` | La config de ton projet Firebase, à remplir une fois |
| `database.rules.json` | Les règles de sécurité à coller dans Firebase |
| `manifest.webmanifest`, `icon-*.png` | L'installation sur l'écran d'accueil (nom, icône, plein écran) |

## Mise en ligne, étape par étape

### 1. GitHub
1. Crée un compte sur github.com.
2. **New repository** → nom `alcolo` → Public → *Create repository*.
3. **uploading an existing file** → glisse tous les fichiers de ce dossier → *Commit changes*.

### 2. Firebase (mode spectateur)
1. console.firebase.google.com → **Créer un projet** → nom `alcolo` → Google Analytics : non.
2. Menu **Créer → Realtime Database** → *Créer une base de données* → emplacement **europe-west1 (Belgique)** → *Démarrer en mode verrouillé*.
3. Onglet **Règles** → remplace tout par le contenu de `database.rules.json` → *Publier*.
4. ⚙️ **Paramètres du projet** → *Vos applications* → icône **</>** (Web) → nom `alcolo` → *Enregistrer* (pas besoin de Firebase Hosting).
5. Copie les valeurs de `firebaseConfig` (apiKey, authDomain, databaseURL, projectId, appId) dans `firebase-config.js`. Vérifie que `databaseURL` est bien présent : sinon il est affiché en haut de l'onglet Realtime Database.
6. Sur GitHub, ouvre `firebase-config.js` → ✏️ → colle → *Commit changes*.

### 3. Cloudflare Pages (hébergement)
1. dash.cloudflare.com → crée un compte gratuit.
2. **Workers & Pages → Create → Pages → Connect to Git** → autorise GitHub → choisis `alcolo`.
3. Framework preset : **None**. Build command : *(vide)*. Build output directory : **/** → *Save and Deploy*.
4. Ton jeu est en ligne sur `https://alcolo.pages.dev` (ou un nom proche si celui-ci est pris).

### 4. Installer sur les téléphones
- **iPhone** : ouvrir le lien dans Safari → Partager → *Sur l'écran d'accueil*.
- **Android** : ouvrir dans Chrome → ⋮ → *Installer l'application*.

## Mettre à jour
Modifie `index.html` (dans VS Code ou directement sur GitHub) puis *commit + push* : Cloudflare remet le site en ligne tout seul en une minute environ.
Sur les téléphones, fermer et rouvrir l'appli suffit en général. Sinon, rouvrir le lien une fois dans le navigateur.

## Mode spectateur
- Celui qui lance une partie voit un bouton doré **Code XXXX** en haut de l'écran.
- Les autres : accueil → **Regarder une partie** → taper le code. Ils voient l'écran en direct, sans pouvoir toucher.
- Les mains secrètes de la Pyramide ne sont jamais envoyées.
- Gratuit jusqu'à 100 téléphones connectés en même temps (plan Spark de Firebase).
- Les parties ne contiennent aucune donnée personnelle, seulement l'écran du jeu. Un code de 4 caractères suffit à rejoindre ou à écraser une partie : ça convient pour jouer entre potes, pas pour quelque chose de sensible.
