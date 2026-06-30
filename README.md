<div align="center">

# 🔧 Larka — Gestion de Maintenance Assistée par Ordinateur (GMAO)

**Plateforme web _multi-tenant_ de gestion de maintenance.**
Backend PHP sans framework, base PostgreSQL par client, frontend JavaScript sans build, installable en PWA.

[![Licence](https://img.shields.io/badge/licence-propri%C3%A9taire-red.svg)](LICENSE)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-soutenir%20le%20projet-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/chipsoreo)
[![PHP](https://img.shields.io/badge/PHP-8.3-777BB4.svg?logo=php&logoColor=white)](https://www.php.net/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-%E2%89%A514-336791.svg?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Frontend](https://img.shields.io/badge/frontend-Vanilla%20JS%20(no%20build)-F7DF1E.svg?logo=javascript&logoColor=black)](#stack-technique)
[![PWA](https://img.shields.io/badge/PWA-installable-5A0FC8.svg)](#)
[![Multi-tenant](https://img.shields.io/badge/architecture-multi--tenant-0a1628.svg)](#architecture)

</div>

---

## 📋 À propos

**Larka** est une application web de gestion de maintenance pensée pour héberger plusieurs organisations
(_tenants_) sur une même installation, **chaque client disposant de sa propre base de données** (isolation
physique, pas seulement logique). Elle couvre le parc de biens et d'équipements, les demandes et
interventions, les contrats, le stock, le suivi énergétique et carbone, l'archivage et les plans de bâtiments.

Le projet fait des choix volontairement **sobres et robustes** : aucun framework PHP ni Composer requis,
aucune étape de build côté frontend (JavaScript natif), une seule dépendance tierce embarquée (lecture de
codes-barres / QR). Surface d'attaque réduite, montées de version simplifiées.

> 📐 L'architecture complète (composants, réseau, sécurité, exploitation) est décrite dans le
> **Dossier d'Architecture Technique** (`DAT-Larka-SharePoint.docx`).

---

## ✨ Fonctionnalités

**Métier**

- 🏢 **Multi-tenant** — une base PostgreSQL par client, routage automatique selon le domaine (web ou e-mail).
- 🧰 **Parc** — biens, équipements, parc matériel, déclarations d'inventaire.
- 🛠️ **Maintenance** — interventions préventives et correctives, devis, lignes de stock liées, demandes d'intervention.
- 📄 **Contrats** de maintenance et listes de référence.
- 📦 **Stock** avec seuils et mouvements.
- ⚡ **Énergie & carbone** — compteurs, relevés, réseaux de chaleur, bilan carbone (facteurs d'émission).
- 🗺️ **Plans** de bâtiments (étages, éléments, liens cartographiques).
- 🗃️ **Archives** physiques (dossiers, boîtes, bordereaux) et **documents** (PDF/images, déduplication SHA-256).
- 📊 **Tableau de bord** et statistiques avancées.
- 🇫🇷 **Connecteurs réglementaires** — Légifrance et Chorus Pro via la plateforme PISTE ; SharePoint/OneDrive.
- 🔔 **Notifications push** (Web Push natif, RFC 8030/8291/8292).

**Technique**

- 🔐 **Sécurité par défaut** — HTTPS/HSTS, CSP, chiffrement AES-256-GCM des secrets, RBAC, journal de sécurité.
- 🔑 **Authentification fédérée** — OAuth 2.0 / OpenID Connect (Microsoft Entra ID, Google) + comptes locaux (bcrypt).
- 📱 **PWA** installable (poste et mobile), notifications push iOS 16.4+ en mode écran d'accueil.
- ⚡ **Chargement paresseux** du frontend (lazy-loading) avec préchargement intelligent selon le rôle.
- 🎨 **Écran de connexion personnalisable** par le super-administrateur (couleurs, fond animé, CSS filtré — voir plus bas).


## 🧱 Stack technique

| Couche | Technologie | Version |
|---|---|---|
| Frontend | JavaScript (ES2017+), HTML5, CSS3 — SPA + PWA, **sans build** | — |
| Frontend (lib.) | ZXing-browser (codes-barres / QR) — _composant tiers_ | embarquée |
| Backend | PHP (sans framework, sans Composer) | 8.3 |
| Serveur applicatif | PHP-FPM | 8.3 |
| Serveur web | Nginx (reverse proxy, TLS, fichiers statiques) | ≥ 1.18 |
| Base de données | PostgreSQL (une base par tenant) | ≥ 14 |
| BDD (repli admin) | SQLite / MariaDB | — |
| TLS | Let's Encrypt (Certbot) | — |
| OS | Ubuntu / Debian | 22.04+ / 12+ |

---

## 🏗️ Architecture

```
Navigateur (PWA, HTTPS)
        │
        ▼
   Nginx  ──(socket Unix, FastCGI)──►  PHP-FPM (API REST, JSON)
        │                                   │
   fichiers statiques                 TenantResolver
                                            │  résout le tenant (domaine / session / e-mail)
                                            ▼
                                  PostgreSQL — 1 base par tenant
                                  (+ base d'administration centrale)

Appels sortants optionnels (HTTPS) : Microsoft Graph · Google · PISTE (Légifrance/Chorus) · Web Push · SMTP
```

- **Client-serveur en API REST** : le serveur ne produit que des données JSON (`{ success, data }`).
- **Multi-tenant à bases cloisonnées** : isolation **physique** des données par organisation.
- **Sans dépendance lourde** : routage, accès données (PDO), auth, chiffrement et Web Push en PHP natif.

---

## 🚀 Démarrage rapide (développement)

> Prérequis : **Debian 12+ / Ubuntu 22.04+**, **PHP 8.1+** (8.3 recommandé) avec `pgsql mbstring xml curl gd zip intl bcmath`, **PostgreSQL 14+**.
> `./start.sh install` installe automatiquement PHP, PostgreSQL et les extensions sur Debian/Ubuntu.

Depuis la **racine** du projet (le dossier qui contient `start.sh`) :

```bash
./start.sh install     # installe PHP + PostgreSQL, crée les bases, génère .env + config.json
./start.sh start       # démarre l'application  →  http://127.0.0.1:8000
```

Laissez le terminal ouvert (**Ctrl+C** pour arrêter), puis ouvrez **http://127.0.0.1:8000**.

**Super Admin** : `http://127.0.0.1:8000/?superadmin` — login `superadmin`,
mot de passe `SuperAdmin2025!` (**à changer immédiatement**).

Raccourcis `make` :

```bash
make install        # = ./start.sh install
make up             # démarre en arrière-plan (démon)
make stop           # arrête le démon
make doctor         # diagnostic (PHP, extensions, PostgreSQL, config)
make up PORT=9000 HOST=0.0.0.0
```

---

## 🏭 Installation en production

```bash
sudo ./start.sh prod --domain gmao.monentreprise.fr
```

Met en place PostgreSQL, **PHP-FPM**, **Nginx**, un certificat **HTTPS Let's Encrypt**, les tâches **cron**
(renouvellement TLS, rotation des logs, nettoyage des sessions), et génère un `.env` (secrets, `chmod 600`)
+ un `config.json` (sans secret) adaptés au domaine.

Pour un déploiement autonome plus léger (serveur PHP intégré géré par systemd, sans Nginx, p. ex. derrière
un reverse-proxy existant), un gabarit est fourni : [`deploy/gmao.service`](deploy/gmao.service).
L'installation **manuelle** pas-à-pas est décrite en tête de [`deploy/install.sh`](deploy/install.sh).

> ⚠️ **Erreur classique** : `cd deploy && ./install.sh` échoue (mauvais dossier, root requis).
> Restez à la **racine** et utilisez `./start.sh prod`.

---

## ⚙️ Configuration & secrets

| Donnée | Emplacement |
|---|---|
| **Secrets** : mots de passe BDD, clé secrète, secrets OAuth/SMTP, clé privée VAPID | **`.env`** (`chmod 600`, jamais committé) |
| **Connexions OAuth** : `client_id`, `tenant_id`, `redirect_uri` | **`.env`** (éditables dans l'UI) |
| Hôtes, ports, base/utilisateur BDD, CSP, CORS, activation des modules, logs… | **`config.json`** |

Modèles fournis : [`.env.example`](.env.example) et [`config.example.json`](config.example.json).
Les fichiers générés (`.env`, `config.json`) sont exclus par le [`.gitignore`](.gitignore).

**Précédence des secrets** (la plus haute gagne) : variable d'environnement système → `.env` → résidu dans `config.json`.

> ⚠️ Si vous récupérez ce projet depuis une archive ayant pu contenir des secrets, **régénérez-les**
> (clé secrète, clé privée VAPID via l'interface, mots de passe BDD).

---

## 🛡️ Sécurité

- **Transport** : HTTPS imposé (HSTS 1 an), redirection systématique du port 80.
- **Secrets au repos** : `config.json` chiffré en **AES-256-GCM** ; clé dans `config.key` (`chmod 600`).
- **Mots de passe** : hachage **bcrypt** ; jamais stockés en clair.
- **Contrôle d'accès (RBAC)** : rôles Admin · Gestionnaire · Technicien · Visionneur · Demandeur, appliqués **côté serveur**.
- **Durcissement web** : CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy ; liste blanche des points d'entrée PHP (anti-webshell) ; fonctions système désactivées au niveau PHP-FPM.
- **Anti-CSRF** (`Content-Type: application/json` sur les mutations), **anti-bruteforce** (limitation par IP et par compte), **requêtes paramétrées** (anti-injection SQL), **échappement HTML** systématique côté client.
- **Journal de sécurité** structuré au format JSONL (`data/security/audit.log`).

---

## 🎨 Super administration & personnalisation de la connexion

- **Super Admin** (`…/?superadmin`) — gestion des tenants (une base PostgreSQL par tenant), des utilisateurs et monitoring. Connexion locale ou Microsoft.
- **Onglet « 🎨 Personnalisation »** (barre Super Admin) — l'écran de connexion se personnalise via un sélecteur d'**apparence** :
  - **Classique** : couleurs de thème (roue chromatique HSV), **fond animé** ou dégradé, logo (texte, ou image PNG/JPEG/GIF/WEBP/SVG ≈ 500 Ko).
  - **CSS personnalisé** (prioritaire) : votre feuille de style est **filtrée côté serveur** (rejet de `url()`, `@import`, `@font-face`, `position:fixed/sticky`, `pointer-events`… ; 8 Ko max) puis appliquée à une **couche décorative non interactive**, derrière la carte de connexion.

  > 🔒 L'ancien mode « HTML brut » a été **retiré** (risque de XSS sur une page accessible avant authentification).
  > La mention de droit d'auteur et de licence affichée sur l'écran de connexion est **codée en dur** et non personnalisable.

- **OAuth Microsoft / Google** : `client_id`, `tenant_id`, `client_secret`, `redirect_uri` se configurent dans **Configuration → Serveur** (écrit dans `.env`) ; seul l'interrupteur `actif` reste dans `config.json`. Les utilisateurs Microsoft se connectent avec le rôle **Demandeur** par défaut.

---

## 🧪 Diagnostic & dépannage

`start.sh` signale les échecs avec un code **`GMAO-Exx`** (également le code de sortie : `echo $?`).
`./start.sh doctor` teste PHP, les extensions, PostgreSQL et la configuration, et annote chaque problème.

| Code | Cause probable | Correction |
|---|---|---|
| `GMAO-E11` | Extensions PHP manquantes | `apt install php-pgsql php-mbstring php-curl php-gd php-xml` |
| `GMAO-E21` | PostgreSQL non démarré | `sudo systemctl start postgresql` |
| `GMAO-E23` | Authentification `pg_hba.conf` refusée | `./start.sh setup --fix-pg-auth` |
| `GMAO-E24` | Rôle `gmao` sans `CREATEDB` (création de tenant) | `sudo -u postgres psql -c "ALTER ROLE gmao CREATEDB;"` |
| `GMAO-E40` | Port déjà utilisé | `./start.sh start --port 9001` |

Repartir de zéro (**destructif**) : `./start.sh reset && ./start.sh start`.

---

## 📁 Structure du projet

```
gmao/
├── index.html              Coquille SPA / PWA
├── manifest.webmanifest    Métadonnées PWA
├── sw.js                   Service Worker (push)
├── start.sh  Makefile      Lanceur dev + raccourcis
├── api/                    Backend PHP (API REST)
│   ├── index.php           Point d'entrée / routeur
│   ├── TenantResolver.php  Résolution multi-tenant
│   ├── ConfigCrypto.php    Chiffrement AES-256-GCM
│   ├── SecurityLog.php     Journal de sécurité (JSONL)
│   ├── WebPush.php         Web Push natif (VAPID)
│   └── routes/             Modules fonctionnels (auth, inventaire, maintenance, …)
├── js/                     Frontend (cœur + pages + filtres, lazy-loadés)
├── css/                    Feuilles de style
├── oauth/                  Callbacks OAuth (microsoft.php, google.php)
├── deploy/                 install.sh · nginx.conf · php-fpm-gmao.conf · gmao.service
└── data/                   Runtime : backups, logs, sessions, security  (hors web)
```

---

## 🤝 Contribuer

Les contributions sont les bienvenues. Quelques points utiles :

- Lancez `./start.sh doctor` avant d'ouvrir une issue d'installation.
- Avant un commit : `php -l` sur les fichiers PHP modifiés et `node --check` sur les fichiers JS.
- Conservez les **en-têtes de licence SPDX** ; pour les (ré)appliquer : `bash scripts/add-license-headers.sh`.
- Toute **modification** du logiciel requiert l'**autorisation écrite préalable** de l'auteur (voir [`LICENSE`](LICENSE), §5).

---

## 📜 Licence

Larka est un **logiciel propriétaire**. © 2025-2026 Mickaël Larcin (« Chipsoreo ») — **Tous droits réservés**.
Son utilisation est régie par la **Licence d'utilisation Larka** (identifiant SPDX `LicenseRef-Larka-Proprietary`) ;
voir le fichier [`LICENSE`](LICENSE).

> ⚠️ Ce n'est **pas** un logiciel libre / open source. Sauf **autorisation écrite et préalable**
> de l'auteur, toute reproduction, distribution, modification, déobfuscation ou ingénierie
> inverse est interdite. Le logiciel est fourni « en l'état », **sans aucune garantie**.

La bibliothèque tierce **ZXing-browser** conserve sa propre licence d'origine et n'est pas couverte par le
droit d'auteur du présent logiciel.

---

## 👤 Auteur

**Mickaël Larcin** (alias _Chipsoreo_) — © 2025-2026.
La liste des auteurs est tenue dans le fichier [`AUTHORS`](AUTHORS).
