# DIOKOO — application Android

> Note : le nom affiché de l'application est **DIOKOO**. Le nom de package
> interne (`com.electrosoft.dioko`) et certains noms de code Kotlin
> (`DiokoApp`, `DiokoTheme`...) ont été conservés tels quels — ce sont des
> identifiants techniques invisibles pour l'utilisateur final, qu'il n'est
> pas nécessaire de renommer pour que l'app s'appelle DIOKOO. Dites-moi si
> vous préférez que je les aligne aussi.

Application de petites annonces (terrains, véhicules, pièces détachées,
location de villas, entrepôts...) pour le marché sénégalais, avec
géolocalisation, recherche par zone, messagerie intégrée et publication
payante (500 FCFA via Wave / Orange Money).

Projet Kotlin + Jetpack Compose, architecture MVVM, prêt à ouvrir dans
**Android Studio** (Hedgehog ou plus récent recommandé).

## Obtenir le fichier .apk installable

Ce projet est le **code source complet** de l'application. Je ne peux pas
compiler moi-même un `.apk` dans cet environnement de conversation (pas
d'accès aux serveurs Android/Google depuis ce sandbox). Deux façons simples
d'obtenir le vrai fichier :

**Option A — la plus simple, sans rien installer sur votre ordinateur :**
1. Créez un dépôt gratuit sur [github.com](https://github.com) et déposez-y
   le contenu de ce dossier (glisser-déposer via l'interface web fonctionne,
   pas besoin de ligne de commande).
2. Un fichier est déjà prêt pour ça : `.github/workflows/build-apk.yml`.
   Dès que le code est sur la branche `main`, GitHub compile automatiquement
   l'application sur ses propres serveurs (qui ont accès à tout ce qu'il
   faut, contrairement à mon environnement).
3. Onglet **Actions** de votre dépôt → le build « Build DIOKOO APK » →
   une fois terminé (~3-5 minutes), téléchargez l'artefact
   **DIOKOO-debug-apk** → c'est votre `.apk`, prêt à transférer sur un
   téléphone Android et à installer (il faudra autoriser « sources
   inconnues » dans les paramètres Android, comme pour toute app hors
   Play Store).

**Option B — avec Android Studio :**
Ouvrez le projet (voir ci-dessous) puis *Build → Build Bundle(s) / APK(s)
→ Build APK(s)*. L'APK apparaît dans
`app/build/outputs/apk/debug/app-debug.apk`.

L'option A est recommandée si vous n'êtes pas développeur : aucune
installation, aucune ligne de commande.

## Ouvrir le projet

1. Décompressez l'archive.
2. Android Studio → *Open* → sélectionnez le dossier `DiokoApp`.
3. Laissez Gradle synchroniser (première synchro : Android Studio régénère
   automatiquement le wrapper Gradle si besoin).
4. Lancez sur un émulateur ou un appareil (API 26+ / Android 8.0+).

Aucune clé API n'est requise pour lancer l'app : toutes les données
(annonces, conversations) sont des données de démonstration en mémoire.

## Ce qui est implémenté

| Exigence | Où |
|---|---|
| Nom + prénom du vendeur sur chaque annonce | `model/Listing.kt`, affiché sur chaque carte et fiche détail |
| Contact (téléphone + email de l'annonceur) | `model/Listing.kt` (`phone`, `email`) ; affichés et actionnables (appel, email) sur la fiche annonce |
| Géolocalisation du bien / vendeur | Chaque `Listing` porte `latitude`/`longitude` ; capturée automatiquement à la publication via `LocationHelper` (bouton « Utiliser ma position actuelle ») |
| Écran d'accueil avec toutes les catégories | `HomeScreen.kt` — chips Terrains, Villas/Location, Voitures, Motos, Pièces détachées, Entrepôts |
| Barre de recherche | `DiokoSearchBar.kt`, filtre en direct sur le titre et la ville |
| Recherche géolocalisée (logements/terrains) | Bouton 📍 dans la barre de recherche + curseur de rayon (km), calcul de distance par formule de Haversine (`util/DistanceUtils.kt`) |
| Recherche par zone géographique | Menu déroulant « Zone » sur l'accueil (Dakar, Pikine, Guédiawaye, Thiès, Mbour...) |
| Publication payante (500 FCFA, Wave / Orange Money) | `PublishScreen` → `PaymentScreen` (`ui/screens/publish/`) — voir note d'intégration ci-dessous |
| Messagerie acheteur ↔ annonceur | Bouton « Message » sur chaque fiche annonce (`ListingDetailScreen`) ouvrant une conversation réelle (`MessagesScreen`, `ChatScreen`) |
| Design distinctif | Palette DIOKOO (indigo / terre cuite / ocre), voir `ui/theme/Color.kt` |
| Logo | `branding/dioko_icon_mark.svg` (icône) et `branding/dioko_logo_lockup.svg` (logo complet avec le nom) — déjà intégré comme icône de lancement adaptative de l'app |

## Note importante — paiement Wave / Orange Money

Le flux de paiement dans `PaymentViewModel.kt` est **simulé** (délai de 1,6s
puis succès) afin que l'application soit démontrable sans backend ni compte
marchand. C'est un point d'intégration clairement isolé : pour la mise en
production, il faut remplacer `PaymentViewModel.pay()` par un appel à votre
backend, qui lui-même appelle :

- l'API Checkout de Wave (paiement web/marchand), ou
- l'API Web Payment d'Orange Money,

puis confirme la transaction via leur webhook côté serveur **avant** de
publier réellement l'annonce — ne jamais se fier à un simple « succès »
côté client pour du paiement réel. Contactez Wave Business / Orange Money
Sénégal pour obtenir les identifiants marchands nécessaires.

## Limites connues (prototype)

- Les données (annonces, conversations) sont en mémoire uniquement — tout
  est réinitialisé au redémarrage de l'app. Étape suivante naturelle :
  remplacer `ListingRepository` par un vrai backend (API REST / Firestore)
  avec une base de données géospatiale (PostGIS, Firestore GeoPoint +
  geohash, etc.) pour des recherches par rayon performantes à grande échelle.
- Pas de compte utilisateur / authentification (le profil est fixe :
  Moussa Ndiaye). À ajouter avant mise en production.
- Le bouton d'appel utilise `ACTION_DIAL` (ouvre le composeur, ne lance pas
  l'appel automatiquement — plus sûr et n'exige pas la permission
  `CALL_PHONE` à l'exécution).

## Structure du projet

```
app/src/main/java/com/electrosoft/dioko/
├── model/        Listing, Category, Conversation
├── data/         ListingRepository (source de données en mémoire) + données de démo
├── location/     LocationHelper (Fused Location Provider)
├── util/         distance (Haversine), formatage prix/FCFA
├── ui/theme/     couleurs, typographie, thème Compose
├── ui/components/ cartes, barre de recherche, chips, navigation basse
├── ui/navigation/ graphe de navigation
└── ui/screens/   home, detail, publish (+ payment), messages (+ chat), profile
```
