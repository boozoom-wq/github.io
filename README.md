# Le site des liens d'invitation

Un lien d'invitation Kokott ressemble à `https://boozoom-wq.github.io/invite/{token}`.
Parce qu'il commence par `https`, il est cliquable dans WhatsApp, les SMS et les e-mails.
L'ancien `kokott://` ne l'était presque nulle part, et c'est pour ça qu'il fallait un code.

Ce dossier est **tout le site**, images comprises : rien à compiler.

| Fichier | Rôle |
|---|---|
| `.well-known/assetlinks.json` | Dit à Android « ce site appartient à l'app `app.kokott` signée par telle clé ». Avec lui, toucher le lien ouvre Kokott directement. |
| `404.html` | La page de secours, quand le lien s'ouvre dans le navigateur (app absente, ou site pas encore vérifié). Son bouton « Ouvrir dans Kokott » lance l'app quand même. |
| `.nojekyll` | Sans lui, GitHub Pages ignore le dossier `.well-known` (il commence par un point). |
| `assets/logo.png` | Logo du bol fumant affiché sur la page d'invitation. |
| `assets/favicon.png` | Icône de l'onglet du navigateur (32 px). |
| `assets/apple-touch-icon.png` | Icône d'écran d'accueil (180 px). |

Le site vit dans un **dépôt GitHub séparé**, parce que GitHub Pages ne sert un fichier à la racine
du domaine (`/.well-known/…`, qu'Android exige) que depuis le dépôt nommé `boozoom-wq.github.io`.
Le dossier `site/` d'ici en est la source de référence, et la CI vérifie que son `assetlinks.json`
porte bien l'empreinte de la clé de release.

## Mettre le site en ligne (une seule fois)

1. Sur github.com, connecté en `boozoom-wq` : **New repository**, nom exactement
   `boozoom-wq.github.io`, **Public**, puis *Create repository*.
2. Dans ce dépôt, **Add file → Create new file**, trois fois, en recopiant les fichiers d'ici :
   - nom `.nojekyll`, contenu vide ;
   - nom `.well-known/assetlinks.json` (le `/` crée le dossier), contenu de ce fichier ;
   - nom `404.html`, contenu de ce fichier.
   Puis **Add file → Upload files** : ajouter le dossier `assets/` et ses trois images.
3. **Settings → Pages** : *Source* « Deploy from a branch », branche `main`, dossier `/ (root)`.
   Une ou deux minutes plus tard, `https://boozoom-wq.github.io/invite/test` affiche la page
   de secours.

## Mettre à jour le logo

La source est `app/assets/logo.png`. Depuis `app/`,
`python tool/generate_icon_layers.py` régénère les trois images de `site/assets/`
(Python avec Pillow). Elles sont commitées ici avec les icônes de l'application.
Recopier `404.html` et le dossier `assets/` dans `boozoom-wq.github.io` pour que
le site publié utilise la nouvelle marque. Les chemins d'images commencent par
`/assets/` afin de fonctionner aussi depuis `/invite/{token}`.

## Mettre l'empreinte de la clé

`assetlinks.json` contient d'abord `EMPREINTE_A_RECOPIER_DEPUIS_LA_CI`. Pour la vraie valeur :

1. GitHub → dépôt `kokott` → **Actions** → le dernier run → job **android** → étape
   **Empreinte pour le lien d'invitation**. Elle affiche une ligne du genre
   `AB:12:CD:…` (32 paires).
2. Remplacer `EMPREINTE_A_RECOPIER_DEPUIS_LA_CI` par cette ligne, **aux deux endroits** :
   `site/.well-known/assetlinks.json` ici (l'avertissement de la CI disparaît), et le même
   fichier dans `boozoom-wq.github.io`.

L'empreinte ne change jamais tant que le keystore de release reste le même.

## Vérifier

- `https://boozoom-wq.github.io/.well-known/assetlinks.json` affiche le JSON.
- L'outil de Google doit renvoyer l'empreinte, sans erreur :
  `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://boozoom-wq.github.io&relation=delegate_permission/common.handle_all_urls`
- Sur le téléphone, Android vérifie le site **à l'installation** de l'APK (réseau requis).
  Si l'APK a été installé avant que le site soit en ligne : Paramètres → Applis → Kokott →
  *Ouvrir par défaut* → activer `boozoom-wq.github.io` dans les liens pris en charge, ou
  réinstaller par-dessus.

Tant que rien de tout ça n'est fait, les liens marchent quand même, avec un toucher de plus :
ils ouvrent la page de secours, dont le bouton lance Kokott.

## Changer d'adresse (un domaine acheté, un autre hébergeur)

1. Publier ces mêmes fichiers sur le nouvel hôte, à sa racine.
2. Dans l'app, changer `inviteLinkHost` (`app/lib/core/invite_link.dart`) et le `android:host`
   du manifeste Android. Le test `android_manifest_test.dart` échoue si les deux ne sont pas
   identiques.
3. Construire et installer un nouvel APK.
4. Laisser l'ancien site en ligne une semaine : les liens déjà envoyés expirent au bout de 7 jours,
   et d'ici là sa page de secours les fait encore aboutir.
