# Prototype "Service Recovery" — Maison Galzin

Page QR : landing avec smileys → bifurcation → formulaire interne → confirmation.
Testé et fonctionnel (flow complet vérifié en local).

## Déployer sur Vercel (5 minutes)

Prérequis : avoir Node.js installé, et un compte Vercel (le même que pour Cibo Vino).

```bash
# 1. Depuis ce dossier
npm install -g vercel   # si pas déjà installé

# 2. Se connecter (ouvre le navigateur)
vercel login

# 3. Déployer
vercel

# Répondre aux questions :
#   Set up and deploy? -> Y
#   Which scope? -> ton compte
#   Link to existing project? -> N
#   Project name? -> galzin-avis (ou ce que tu veux)
#   Directory? -> ./ (racine, laisser par défaut)
#   Override settings? -> N (c'est un site statique, rien à configurer)

# 4. Une fois déployé en preview, mettre en prod :
vercel --prod
```

Vercel te donne une URL du type `https://galzin-avis.vercel.app`.
Tu peux ensuite brancher un domaine perso (ex. `avis.galzin.fr`) dans le dashboard Vercel → Settings → Domains.

## Tester après déploiement

- Carnon : `https://TON-URL.vercel.app/?boutique=carnon`
- Saint-Éloi : `https://TON-URL.vercel.app/?boutique=saint-eloi`

Vérifier :
1. Le nom de la boutique s'affiche bien en haut
2. 😊 redirige vers la vraie fiche Google (après 1,4s)
3. 😐 / 😞 affichent le formulaire, le bouton "Envoyer" reste grisé tant qu'un motif n'est pas choisi
4. Après envoi, l'écran de confirmation s'affiche avec un encart debug (à retirer avant vrai lancement terrain)

## Avant le vrai lancement (pas juste la démo)

Ce prototype stocke les signalements dans le `localStorage` du téléphone du client — **ça ne sert qu'à la démo**, ces données ne remontent à personne. Avant Carnon/Saint-Éloi en vrai :

1. **Créer un projet Firebase** (ou réutiliser celui de l'appli d'évaluation staff)
2. Dans `index.html`, remplacer la fonction `saveSignalement()` par :
   ```js
   import { initializeApp } from "firebase/app";
   import { getFirestore, collection, addDoc } from "firebase/firestore";

   const app = initializeApp({ /* config Firebase */ });
   const db = getFirestore(app);

   async function saveSignalement(payload) {
     await addDoc(collection(db, "signalements"), payload);
   }
   ```
3. Créer une Firebase Function déclenchée sur la création d'un document dans `signalements`, qui envoie un mail/Slack au responsable de la boutique concernée (`payload.boutique`)
4. Retirer le bloc `debug-panel` dans `renderConfirm()` (visible uniquement en démo)
5. Générer les QR codes pointant vers `https://TON-URL/?boutique=carnon` et `.../?boutique=saint-eloi`

## Structure du fichier

Tout est dans `index.html` (HTML + CSS + JS, aucune dépendance externe hors Google Fonts).
La config des boutiques est isolée en haut du `<script>` dans l'objet `BOUTIQUES` — pour ajouter une boutique, il suffit d'ajouter une entrée avec son `placeId` Google.
