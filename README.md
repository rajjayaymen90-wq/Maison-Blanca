# ATELIER SELECT

Boutique multimarque en français avec React 19, Vite, React Router et API serverless Stripe. Projet structuré, déployable sur **Vercel ou Netlify**, utilisable immédiatement en mode démo.

## Démarrage

Node.js **22.12+** et npm sont nécessaires.

```sh
npm ci
cp .env.example .env
npm run dev
```

Sous PowerShell, remplacer la deuxième commande par `Copy-Item .env.example .env`.

Ouvrir **http://127.0.0.1:5173**. `npm run dev` lance Vite et l’API locale sur le port 3001. Les requêtes `/api` sont relayées automatiquement par Vite. Garder `SITE_URL` identique à l’origine utilisée dans le navigateur.

```sh
npm run build    # validation catalogue, compilation, sitemap et robots.txt
npm run preview  # aperçu du frontend compilé ; ne lance pas l’API
npm test        # calculs, stock, validation des requêtes et signatures webhook
```

Le projet est livré sans clé privée, sans compte externe configuré et sans publication automatique.

## Fonctionnalités

- Accueil éditorial : hero, nouveautés, meilleures ventes, promotions et catégories.
- 12 produits d’exemple et 4 marques ; recherche, marque, taille, couleur, bornes de prix et quatre tris. Les critères sont conservés dans l’URL.
- Fiches produits : galerie avec vue générale et recadrage de détail, zoom au survol, tailles indisponibles, couleurs, quantité, guide indicatif et suggestions.
- Panier en panneau latéral et page complète, quantités, suppression, persistance locale, promo `BIENVENUE10` et seuil de livraison offerte.
- Adresse, e-mail, téléphone, livraison standard/express, commande locale simulée ou Stripe Checkout en mode test.
- Confirmation détaillée et compte client local avec inscription, connexion, déconnexion et historique des commandes passées en étant connecté.
- Interface responsive, images différées hors hero, routes secondaires chargées à la demande, dialogs accessibles au clavier, métadonnées par route et sitemap.
- Outils WebMCP facultatifs `read_catalog` et `add_to_cart` lorsque le navigateur les prend en charge.

## Structure

```text
api/                      # points d’entrée Vercel : checkout, session, webhook
server/                   # logique API commune, validation, adaptateur Vercel
netlify/functions/        # adaptateurs Netlify vers les mêmes handlers
data/
  products.json           # catalogue : prix, photos, tailles, couleurs, stocks
  shop.json               # nom, contact, promos, livraison, visuel hero
src/
  components/             # navigation, panier, récapitulatif, modal, SEO
  context/ShopContext.jsx # panier, session client et notifications
  lib/                    # calculs communs et stockage démo
  pages/                  # 11 vues, dont fiches produit et page 404
  App.jsx                 # routes
  styles.css              # design et règles responsive
scripts/                  # serveur local, contrôle catalogue, génération SEO
tests/                    # tests métier et contrats API
public/favicon.svg
vite.config.js
vercel.json
netlify.toml
.env.example
package-lock.json
```

## Modifier le catalogue sans toucher aux composants

Éditer `data/products.json`, puis reconstruire/redéployer le projet. Les modifications sont relues automatiquement pendant le développement.

Les montants sont exprimés en **centimes d’euro** : `3500` = 35 €. Exemple :

```json
{
  "id": "tee-exemple",
  "slug": "t-shirt-exemple",
  "name": "T-shirt Exemple",
  "brand": "Votre marque",
  "price": 3500,
  "oldPrice": 4500,
  "category": "t-shirts",
  "isNew": true,
  "popularity": 90,
  "createdAt": "2026-09-16",
  "fit": "Coupe droite",
  "material": "100 % coton",
  "description": "Description exacte du produit.",
  "images": ["/images/tee-face.jpg", "/images/tee-dos.jpg"],
  "sizes": ["S", "M", "L"],
  "colors": [{ "name": "Noir", "hex": "#272727" }],
  "stock": { "Noir": { "S": 4, "M": 8, "L": 0 } }
}
```

- `id` et `slug` doivent être uniques. Conserver un identifiant stable pour les articles déjà dans des paniers.
- `category` accepte `t-shirts` et `pulls`. Les pulls comprennent les sweats/hoodies.
- Chaque couple taille/couleur doit avoir un stock entier positif ou nul. `0` désactive cette taille pour la couleur choisie.
- `oldPrice` et `isNew` sont facultatifs. La présence de `oldPrice` crée une promotion.
- La popularité et la date sont des données de démonstration modifiables, pas des statistiques de ventes réelles.
- Placer vos photographies dans `public/images/`, ou utiliser des URL HTTPS autorisées.
- Le premier visuel est utilisé dans les cartes. Les autres alimentent la galerie. Les visuels actuels sont des illustrations, pas les photographies officielles des références nommées.

Dans `data/shop.json`, personnaliser les coordonnées, la livraison, le seuil, les promotions et le hero. L’identité textuelle ATELIER SELECT dans les pages/layout et les métadonnées se renomme par recherche globale. Le style utilise les variables CSS en tête de `src/styles.css`.

## Règles de prix

- Livraison standard : 4,90 €, offerte à partir de 100 € de produits **après réduction**.
- Livraison express : 9,90 €, indépendamment du montant.
- `BIENVENUE10` : 10 % sur les articles, hors livraison. La réduction est arrondie au centime par unité, puis multipliée par la quantité. Le frontend et l’API partagent ce calcul.
- Maximum de 10 exemplaires d’une variante par commande, dans la limite du stock configuré.
- Les prix d’exemple sont affichés taxes incluses. Aucun moteur de TVA internationale ni facture fiscale n’est implémenté.

## Mode démo et compte client

`VITE_DEMO_MODE=true` active les commandes simulées, sans appel de paiement. Aucun débit, aucun e-mail et aucune expédition ne sont effectués.

Le panier, le compte et les commandes sont enregistrés sous le préfixe `atelier:` dans `localStorage`. Les mots de passe démo sont dérivés avec PBKDF2 et un sel aléatoire ; ils ne sont pas conservés en clair. **Cela reste une simulation d’authentification** : le stockage local est modifiable et ne peut servir d’autorisation à une API réelle.

Utiliser des données et mots de passe fictifs. L’historique n’est pas partagé entre appareils et peut être effacé avec les données du navigateur. Les commandes invité ne sont pas automatiquement rattachées à un nouveau compte ; leur confirmation reste disponible par son lien sur le même appareil.

Le formulaire de contact prépare un lien `mailto:` et ouvre la messagerie. Il ne prétend pas envoyer un message côté serveur. Remplacer `bonjour@votre-boutique.fr` avant utilisation.

## Activer Stripe Checkout en test

1. Copier `.env.example` en `.env` et renseigner une **clé secrète de test** depuis votre compte Stripe.
2. Mettre `VITE_DEMO_MODE=false`.
3. Renseigner `SITE_URL` et `VITE_SITE_URL` avec l’origine exacte du site, sans chemin final. En local : `http://127.0.0.1:5173`.
4. Redémarrer le serveur ou reconstruire le déploiement après modification des variables `VITE_*`.

```dotenv
STRIPE_SECRET_KEY=sk_test_votre_cle
STRIPE_WEBHOOK_SECRET=whsec_votre_secret
SITE_URL=https://votre-domaine.fr
VITE_SITE_URL=https://votre-domaine.fr
VITE_DEMO_MODE=false
```

**Ne jamais préfixer une clé secrète par `VITE_`** : les variables `VITE_*` sont intégrées au navigateur. La redirection s’effectue via l’URL créée par le serveur ; ce flux n’a pas besoin de clé publique Stripe dans le frontend.

`POST /api/checkout` valide les produits, variantes, stocks, coordonnées et promotions. Il recalcule les prix depuis le JSON de confiance ; un prix fourni par le navigateur est ignoré. Il applique les réductions directement aux prix unitaires Stripe, sans créer de coupon externe, et utilise une clé d’idempotence stable pour les nouvelles tentatives identiques.

Au retour de Stripe, `/api/session` vérifie côté serveur que la session est terminée et payée. Un jeton aléatoire gardé dans `sessionStorage` et dont l’empreinte est associée à la session Stripe protège la consultation. Un paramètre d’URL seul ne valide pas un paiement. Une session initiée sur un autre appareil ne peut pas être retrouvée avec le compte local démo.

Carte de test courante : `4242 4242 4242 4242`, date future et CVC de test. Consulter les [scénarios officiels Stripe](https://docs.stripe.com/testing) pour les refus et l’authentification.

### Webhook

Déclarer l’URL `https://votre-domaine.fr/api/webhook` dans Stripe et écouter `checkout.session.completed` et `checkout.session.async_payment_succeeded`. Copier le secret de signature dans `STRIPE_WEBHOOK_SECRET`.

En local, avec Stripe CLI configuré :

```sh
stripe listen --forward-to http://127.0.0.1:3001/api/webhook
```

Le corps brut est conservé par les adaptateurs et la signature est vérifiée avant traitement. Le webhook fourni **valide et journalise les événements de test uniquement**. Il n’enregistre pas de commandes dans une base durable et ne déclenche pas d’expédition. Un accusé `received: true` ne signifie pas qu’une commande a été préparée.

La clé `sk_live_` est volontairement refusée tant qu’un vrai backend de commandes n’est pas ajouté. Pour vendre réellement, remplacer la démonstration par une authentification serveur, une base durable, une décrémentation transactionnelle des stocks, une déduplication des événements/commandes et une tâche d’expédition. Ajouter également les protections de débit et les conditions commerciales adaptées à votre activité. Ne pas simplement retirer le contrôle de clé sans ces traitements.

Références : [Checkout Sessions](https://docs.stripe.com/api/checkout/sessions), [confirmation](https://docs.stripe.com/payments/checkout/custom-success-page), [idempotence](https://docs.stripe.com/api/idempotent_requests), [signature webhook](https://docs.stripe.com/webhooks/signature), [traitement fiable des commandes](https://docs.stripe.com/checkout/fulfillment).

## Déployer sur Vercel

1. Placer **le contenu de ce dossier** dans un dépôt Git et importer le dépôt dans Vercel. Si vous importez le dossier parent du livrable, choisir `outputs/atelier-select` comme Root Directory.
2. Choisir Vite, Node 22, commande `npm run build`, sortie `dist`. `vercel.json` contient déjà ces réglages et les réécritures des routes SPA ; `/api` reste réservé aux fonctions.
3. Ajouter les variables ci-dessus dans les paramètres de l’environnement concerné. Pour explorer sans Stripe, conserver `VITE_DEMO_MODE=true`.
4. Déployer, puis définir les deux URL avec le domaine exact fourni et redéployer. Vérifier la navigation directe vers `/produit/t-shirt-heavy-ecru` et `/api/session`.
5. Pour Stripe test, déclarer le webhook avec le domaine stable utilisé pour le checkout. Les origines des previews éphémères doivent correspondre à leur propre `SITE_URL`.

Référence : [fonctions Node.js Vercel](https://vercel.com/docs/functions/runtimes/node-js).

## Déployer sur Netlify

1. Importer le même dépôt dans Netlify ; choisir ce dossier comme Base Directory si nécessaire.
2. `netlify.toml` définit le build `npm run build`, le dossier publié `dist` et les fonctions `netlify/functions`.
3. Ajouter les variables dans l’interface Netlify : secrets accessibles aux **Functions**, `VITE_*` accessibles au **Build**. `SITE_URL` doit être disponible pour les fonctions et la génération du sitemap.
4. Déployer. Les routes `/api/*` sont réécrites vers `/.netlify/functions/*`. Les routes React retombent sur `index.html`.
5. Utiliser le domaine exact pour les URL Stripe et déclarer le webhook.

Un simple glisser-déposer de `dist` ne déploie pas les fonctions : utiliser le dépôt Git ou Netlify CLI pour le paiement. Référence : [API des fonctions Netlify](https://docs.netlify.com/build/functions/api/).

## SEO et performance

`SEO.jsx` met à jour titre, description, canonical et règles d’indexation par route. Les pages panier, compte, commande et confirmation sont en `noindex`. Le build génère `dist/sitemap.xml` et `dist/robots.txt` à partir du catalogue et du domaine configuré.

C’est une SPA : les métadonnées propres à chaque route sont appliquées après exécution JavaScript. Pour un référencement et des aperçus sociaux pré-rendus, ajouter du pré-rendu statique ou migrer vers un framework SSR. Aucun visuel social spécifique n’est généré.

Les pages secondaires sont découpées en modules ; les images sous la ligne de flottaison utilisent `loading="lazy"`, et le hero est prioritaire. Les images Unsplash et Google Fonts nécessitent un accès réseau ; vous pouvez héberger vos assets et polices dans `public/` pour supprimer ces dépendances externes.

## Vérification du livrable

Les 11 tests métier/API passent : fusion/restauration du panier, stocks, prix de confiance, arrondis, livraison après remise, requêtes malformées, idempotence, configuration absente et signatures webhook.

Le parcours navigateur a été exercé avec des coordonnées fictives : filtres et tri, taille obligatoire, ajout au panier, code invalide puis valide, rechargement, inscription, livraison express, commande simulée, historique, déconnexion et reconnexion. L’affichage mobile est contrôlé à 390 px.

Dans l’environnement Windows de création, les sous-processus avec pipes sont refusés (`EPERM`). Le rendu a été compilé avec Vite/Rollup via une adaptation de validation locale, transformant JSX avec Babel et désactivant la minification. Les scripts livrés restent les scripts Vite usuels. La compilation standard `npm run build` doit donc être exécutée sur l’hébergeur ou une machine sans cette restriction ; aucun test Stripe distant n’a été effectué sans vos clés.

Voir `ASSETS.md` pour les sources photographiques. Les références produits, compositions, tarifs, accords de distribution, mentions légales, fiscalité et conditions de retour restent à renseigner avec vos informations réelles avant la vente.
