# Prospecteur SC

Outil non officiel pour les mineurs de Star Citizen — version française et simplifiée
des données de la communauté **Regolith Co.** et **UEX Corp**.

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Stack technique](#stack-technique)
- [Démarrer en local](#démarrer-en-local)
- [Mettre à jour les prix](#mettre-à-jour-les-prix)
- [Générer le site et déployer](#générer-le-site-et-déployer)
- [SEO et indexation](#seo-et-indexation)
- [Structure du projet](#structure-du-projet)
- [Données](#données)
- [Maintenance](#maintenance)

---

## Fonctionnalités

### Minage vaisseau
- **Catalogue de minerais** classés par tier de rareté (Exotique, Rare, Précieux, Standard, Commun)
- **Catalogue de zones** organisé par système (Stanton, Pyro, Nyx) et type (planète, Lagrange, astéroïde, ceinture)
- **Fiches détaillées** : top spots par minerai, statistiques par zone
- **Calculateur de cargaison** : Prospector / Mole, brut vs raffiné, profit du raffinage

### Minage terrestre
- **Catalogue de GEMs** (Janalite, Hadanite, Feynmaline, Beradom, Dolivine, Aphorite, Glacosite)
- **Catalogue de lieux** (lunes/planètes des 3 systèmes)
- **3 méthodes différenciées** : à pied (Multi-Tool), véhicule (ROC / ROC DS), Atlas Geo
- **Habitat clair** : surface vs grottes
- **Tableaux séparés par méthode** pour éviter la confusion

### Transversal
- **Top revenus** : comparatif des activités vaisseau et terrestre par UEC/h
- **Prix marché** synchronisés avec UEX Corp (mise à jour automatique au build + au runtime)
- **Filtres et tri** : panneau latéral par système, type de lieu, rareté ou méthode ; état des filtres reflété dans l'URL (recherches partageables)
- **Mode sombre** immersif (palette acier spatial à accents ambrés)

---

## Stack technique

- **Nuxt 4** (mode static / SSG)
- **Nuxt UI 3** (composants)
- **Tailwind CSS 4**
- **TypeScript**
- **@iconify-json/lucide** (icônes Lucide embarquées en local, aucun appel CDN)

Le site fonctionne en **100% statique** — aucun backend requis. Les prix sont rafraîchis
automatiquement à chaque build (`npm run generate`) et au runtime via un cache 24h.
Aucune ressource externe (polices, icônes…) n'est chargée depuis un CDN tiers — tout
est servi depuis votre propre domaine, conforme RGPD sans configuration supplémentaire.

---

## Démarrer en local

### Prérequis

- **Node.js 22+**
- **npm 10+**

### Installation

```bash
npm install
```

### Mode développement

```bash
npm run dev
```

Le site est accessible sur `http://localhost:3000`. Le hot-reload est actif.

---

## Mettre à jour les prix

Les prix marchés sont synchronisés automatiquement depuis l'API publique d'**UEX Corp**.

### Synchronisation seule (sans build)

```bash
npm run sync-prices
```

Sortie type :

```
[sync-prices] Mapping chargé : 22 minerais
[sync-prices] Appel UEX : https://api.uexcorp.space/2.0/commodities
[sync-prices] Récupération UEX : 200 commodités reçues
[sync-prices] 22 minerais synchronisés (20 partiels), 0 ignorés
[sync-prices] ✓ Fichier mis à jour : app/utils/ore-prices.ts
[sync-prices] ✓ Date de fraîcheur : [timestamp ISO du build]
```

### Stratégie de prix

| Source | Priorité | Quand |
|---|---|---|
| **Runtime UEX** (live, cache 24h) | 1 (plus haute) | À chaque visite |
| **Build UEX** (sync au build) | 2 | À chaque `npm run generate` |
| **Fallback Regolith** (statique) | 3 (plus basse) | Si UEX ne répond pas |

Si l'API UEX est indisponible au build, le site continue de fonctionner avec les
valeurs de fallback. Aucune intervention manuelle n'est nécessaire.

---

## Générer le site et déployer

### 1. Générer le site statique

```bash
npm run generate
```

Cette commande exécute en séquence :

1. `npm run sync-prices` — récupère les prix actuels d'UEX (minerais + GEMs)
2. `nuxt generate` — pré-rend les 500+ pages en HTML statique
3. `npm run sitemap` — génère `sitemap.xml` et `robots.txt` à partir des routes
4. `npm run indexnow` — notifie Bing/IndexNow de toutes les URLs (HTTP ping)

Résultat : un dossier **`.output/public/`** d'environ 14 Mo, contenant tout le site.

### 2. Déployer sur un hébergement statique

Le dossier `.output/public/` contient l'intégralité du site. Pour le mettre en ligne :

1. Ouvrez votre client FTP préféré
2. Connectez-vous à votre hébergement avec vos identifiants FTP
3. Naviguez jusqu'au dossier public (souvent `www/` ou `htdocs/`)
4. **Uploadez tout le contenu de `.output/public/`** dans ce dossier
5. Le site est en ligne

> **Important** : uploadez le **contenu** du dossier `.output/public/`, pas le dossier
> lui-même. Le fichier `index.html` doit se trouver à la racine de votre espace web.

### 3. Cycle de mise à jour typique

Quand vous voulez actualiser le site en ligne (nouveau patch SC, nouveaux prix...) :

```bash
git pull               # récupérer les dernières modifications
npm install            # si dépendances modifiées
npm run generate       # build avec prix UEX du jour
```

Puis upload du dossier `.output/public/` sur votre hébergement.

### Hébergeurs compatibles

Le dossier `.output/public/` peut être déployé sur n'importe quel hébergement statique :

- **Hébergement mutualisé** (Apache / Nginx) : upload FTP classique
- **Netlify / Vercel / Cloudflare Pages** : déploiement automatique depuis Git
- **GitHub Pages** : via une GitHub Action
- **Serveur dédié** : copie simple du dossier vers le webroot

---

## SEO et indexation

Le site intègre tous les signaux pour être bien indexé par les moteurs de recherche :

### Côté HTML (généré automatiquement à chaque build)

- **Balise `<link rel="canonical">`** dynamique sur chaque page (basée sur l'URL courante)
- **Meta `robots`** : `index, follow, max-image-preview:large`
- **Données structurées JSON-LD** : `WebSite` + `Organization` (compréhension sémantique)
- **Open Graph + locale `fr_FR`** : partage Discord/Twitter cohérent
- **HTML pré-rendu** : toutes les pages sont en HTML statique (pas de JavaScript-only),
  donc lisibles immédiatement par tous les crawlers

### Sitemap et robots

Générés automatiquement après `nuxt generate` à partir des routes pré-rendues :

- `sitemap.xml` : ~250 URLs avec `<lastmod>`, `<changefreq>`, `<priority>`
- `robots.txt` : pointe vers le sitemap

À soumettre une seule fois dans [Bing Webmaster Tools](https://www.bing.com/webmasters)
et [Google Search Console](https://search.google.com/search-console).

### IndexNow (Bing)

Le script `scripts/indexnow-ping.mjs` notifie Bing/Yandex de toutes les URLs à chaque build,
pour accélérer l'indexation et signaler les mises à jour.

**Pré-requis** :
1. Une clé alphanumérique (8-128 caractères) déclarée dans `scripts/indexnow-ping.mjs`
2. Un fichier `{clé}.txt` à la racine du site contenant la même clé (déjà dans `public/`)
3. Le site doit être accessible publiquement pour que Bing puisse valider la clé

Au premier build sans clé en ligne, le ping renvoie `HTTP 403 SiteVerificationNotCompleted`.
Une fois la clé uploadée, les pings retournent `HTTP 200` et toutes les URLs sont signalées.

### Pour changer la clé IndexNow

1. Remplacer la valeur `KEY` dans `scripts/indexnow-ping.mjs`
2. Renommer le fichier `public/{ancienne-clé}.txt` avec la nouvelle clé
3. Mettre à jour son contenu pour qu'il corresponde au nom du fichier

---

## Structure du projet

```
prospecteur-sc/
├── app/
│   ├── pages/                          # Routes du site
│   │   ├── index.vue                   # Accueil
│   │   ├── minerais/                   # Vaisseau : liste + détail minerai
│   │   ├── zones/                      # Vaisseau : liste + détail zone
│   │   ├── gems/                       # Terrestre : liste + détail GEM
│   │   ├── lieux/                      # Terrestre : liste + détail lieu
│   │   ├── top-revenus.vue             # Comparatif vaisseau + terrestre
│   │   ├── calculateur.vue             # Calculateur de cargaison (vaisseau)
│   │   ├── a-propos.vue                # À propos
│   │   └── comment-ca-marche.vue       # Guide d'utilisation
│   │
│   ├── components/                     # Composants UI
│   │   ├── AppHeader.vue               # Navigation avec menus déroulants
│   │   ├── AppFooter.vue
│   │   ├── OreCard.vue                 # Carte minerai vaisseau
│   │   ├── LocationCard.vue            # Carte zone vaisseau
│   │   ├── GemCard.vue                 # Carte GEM terrestre
│   │   ├── PriceTable.vue              # Tableau prix minerai vaisseau
│   │   ├── GemPriceTables.vue          # 3 tableaux séparés pour GEMs
│   │   ├── ProbabilityBar.vue
│   │   ├── StatsPanel.vue
│   │   ├── TierBadge.vue
│   │   ├── GemTierBadge.vue
│   │   └── FilterPanel.vue             # Panneau de filtres réutilisable
│   │
│   ├── composables/                    # Logique réactive partagée
│   │   ├── useMiningData.ts            # Données minage vaisseau
│   │   ├── useGemData.ts               # Données minage terrestre
│   │   ├── useOrePrices.ts             # Façade prix vaisseau
│   │   ├── useLivePrices.ts            # Fetch UEX runtime + cache
│   │   └── useUrlFilters.ts            # Sync filtres/tri ↔ URL
│   │
│   ├── utils/                          # Données et helpers purs
│   │   ├── ore-classification.ts       # Tiers minerais vaisseau
│   │   ├── ore-prices.ts               # Prix minerais vaisseau
│   │   ├── ore-uex-mapping.ts          # Mapping UEX minerais
│   │   ├── gem-classification.ts       # Tiers, méthodes, habitat GEMs
│   │   ├── gem-prices.ts               # Prix GEMs terrestres
│   │   ├── gem-uex-mapping.ts          # Mapping UEX GEMs
│   │   └── location-hierarchy.ts       # Systèmes, planètes, types
│   │
│   ├── assets/
│   │   ├── css/main.css                # Thème (couleurs, polices, textures)
│   │   └── data/
│   │       ├── ore-locations.json      # Scans Regolith — zones vaisseau
│   │       └── gem-locations.json      # Scans Regolith — lieux terrestres
│   │
│   ├── app.vue                         # Layout principal
│   └── app.config.ts                   # Config Nuxt UI
│
├── scripts/
│   ├── sync-prices.mjs                 # Sync UEX (minerais + GEMs) au build
│   ├── generate-sitemap.mjs            # Génération sitemap + robots.txt
│   └── indexnow-ping.mjs               # Ping IndexNow (Bing) après build
│
├── public/
│   ├── .htaccess                       # Config Apache (HTTPS, cache, sécurité)
│   └── {clé}.txt                       # Fichier de validation IndexNow
│
├── nuxt.config.ts                      # Configuration Nuxt
├── package.json
└── README.md
```

---

## Données

### Sources

- **Zones et probabilités** : [Regolith Co.](https://regolith.rocks) (export manuel, JSON)
- **Prix marché** : [UEX Corp](https://uexcorp.space) (API publique, automatique)

### Couverture actuelle

**Minage vaisseau**
- **22 minerais** : Quantanium, Stileron, Savrilium, Riccite, Lindinium, Bexalite, Gold,
  Borase, Taranite, Beryl, Tungsten, Agricium, Laranite, Titanium, Torite, Hephaestanite,
  Quartz, Tin, Copper, Corundum, Aluminum, Iron, Silicon
- **191 zones** scannées (Stanton + Pyro + Nyx)

**Minage terrestre**
- **7 GEMs** : Janalite, Hadanite, Feynmaline, Beradom, Dolivine, Aphorite, Glacosite
- **25 lieux** scannés (lunes/planètes de Stanton et Pyro)
- **3 méthodes** : à pied (Multi-Tool), véhicule terrestre (ROC / ROC DS), Atlas Geo

### Hors périmètre

- **Hathor Orbital Laser** (Carinite Pure, Jaclium Ore, Saldynium Ore) — gameplay endgame
  avec missions et cavernes temporaires
- **Ice** — pas raffinable au sens classique
- **Diamond** — uniquement extractible via Gem Mining (ROC/FPS) avec un sous-gameplay distinct

### Mise à jour des scans

Le fichier `app/assets/data/ore-locations.json` provient d'un export Regolith Co.
Pour le mettre à jour, remplacer simplement ce fichier par un export plus récent
(même format JSON).

---

## Maintenance

### Modifier un minerai

1. Éditer `app/utils/ore-classification.ts` (tier, description)
2. Éditer `app/utils/ore-prices.ts` (fallback) si UEX ne couvre pas le minerai
3. Éditer `app/utils/ore-uex-mapping.ts` si vous voulez l'auto-sync depuis UEX

### Ajouter une nouvelle zone

Pas d'intervention nécessaire si le JSON Regolith est à jour : la classification
automatique (`app/utils/location-hierarchy.ts`) détecte les nouvelles zones et les
range dans le bon système selon leur préfixe (`HUR-*` → Stanton, `RMB-*` → Pyro, etc.).

Pour une zone nommée (planète, lune), ajouter une entrée dans `PLANET_NAMES` de
`location-hierarchy.ts`.

### Changer le thème

Tout le thème est dans `app/assets/css/main.css`. Les variables principales sont en
haut du fichier sous `@theme static` (`--color-rock-*`, `--color-ember-*`).

---

## Crédits

- **Données minage** : [Regolith Co.](https://regolith.rocks) — communauté de prospecteurs SC
- **Données prix** : [UEX Corp](https://uexcorp.space) — base de données économique communautaire
- **Star Citizen** : © Cloud Imperium Games — projet de fan non officiel

---

## Licence

Code source publié sous licence **GNU AGPL-3.0**. Vous êtes libre de l'utiliser,
le modifier et le redistribuer, à condition de partager vos modifications sous
la même licence.

## Transparence

Ce projet a été entièrement **vibecoder** avec [Claude Code](https://www.claude.com/product/claude-code)
(IA d'Anthropic). La vision produit, les choix fonctionnels et les orientations
sont humains ; le code a été généré par l'IA en dialogue itératif.
