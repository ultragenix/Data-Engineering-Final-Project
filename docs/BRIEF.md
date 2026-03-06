# BRIEF.md — DVF+ La Réunion
## Cahier des charges pour l'agent Architecte
## Date : 06/03/2026

---

## 1. VISION

### Problème
La SAFER Réunion (organisme de gestion du foncier agricole) utilise Vigifoncier pour la veille des ventes en cours, mais n'a **aucun outil d'intelligence analytique** : pas d'analyse historique des prix, pas de croisement multi-sources, pas de dashboards interactifs, pas d'IA. L'analyse se fait dans Excel, les rapports pour le comité technique (12x/an) sont faits manuellement en PowerPoint, et l'évaluation des prix pour les préemptions repose sur l'expérience terrain.

### Utilisateurs cibles
- **Ariste Lauret** (DGD SAFER Réunion) — décideur, premier contact
- **Conseillers fonciers SAFER** — utilisateurs quotidiens (évaluation prix, préemptions, reporting)
- **Comité technique SAFER** — destinataire des rapports trimestriels
- Potentiellement : CDPENAF, collectivités locales (phase ultérieure)

### Proposition de valeur
Un outil d'intelligence foncière qui :
1. Visualise 10 ans de transactions immobilières/foncières à La Réunion (dashboards interactifs)
2. Croise les données DVF avec l'agriculture (RPG), les risques naturels (Géorisques) et les bâtiments (BDNB/DPE)
3. Permet de poser des questions en français via un chat IA (Text-to-SQL)
4. Se met à jour automatiquement à chaque publication DVF (semestriel)

### Positionnement
**Complémentaire à Vigifoncier, pas concurrent.**
- Vigifoncier = veille opérationnelle (que se passe-t-il maintenant ?)
- DVF+ = intelligence analytique (pourquoi et comment ça évolue ?)

---

## 2. DONNÉES

### Source principale — DVF+ Open Data (Cerema)
- **URL** : https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data
- **Format** : SQL PostgreSQL/PostGIS (modèle 17 tables relationnelles)
- **Période** : Janvier 2014 — Décembre 2024
- **Volume 974** : ~55 000-77 000 mutations (5 000-7 000/an)
- **Taille** : ~50-100 Mo SQL, largement dans les capacités du VPS
- **MAJ** : 2x/an (avril + octobre). Prochaine : avril 2026
- **Licence** : Ouverte v2 (libre d'utilisation)
- **Filtrage** : Département 974 uniquement

### Enrichissements

| Source | Priorité | Format | Volume estimé 974 | Clé de jointure |
|--------|----------|--------|-------------------|-----------------|
| **RPG** (parcelles agricoles) | 🔴 Must | GeoPackage | ~42 000 ha | Intersection géospatiale avec parcelles DVF |
| **Géorisques** (risques naturels) | 🔴 Must | API REST v2 | Par code INSEE/coordonnées | Code INSEE commune ou GPS |
| **BDNB** (bâtiments) | 🟡 Should | CSV/GeoPackage dept 974 | ~150 000 bâtiments | batiment_groupe_id via parcelle |
| **BAN** (adresses) | 🟡 Should | API REST (50 req/s) | Géocodage des transactions | Adresse → coordonnées |
| **DPE** (performance énergétique) | 🟡 Should | API ADEME | **À vérifier (rare outre-mer)** | Adresse ou parcelle |
| **INSEE** (socio-éco) | 🔵 Could | CSV | 24 communes | Code INSEE |

### Clés de jointure principales
- DVF ↔ Cadastre : `id_parcelle` (section + numéro)
- DVF ↔ RPG : intersection géospatiale (PostGIS)
- DVF ↔ Géorisques : code INSEE ou coordonnées
- DVF ↔ BDNB : `batiment_groupe_id` via parcelle
- DVF ↔ INSEE : code INSEE commune

### Vigilances données 974
- **Projection** : EPSG:2975 (RGR92/UTM zone 40S) — JAMAIS Lambert-93
- **Géolocalisation** : Taux à vérifier (zones rurales/montagneuses = risque de trous)
- **S2 2024** : Potentiellement incomplet — baser la démo sur 2014-2023 solide
- **DPE** : Volume très faible outre-mer — prévoir plan B via BDNB
- **Foncier agricole** : Identifier via codtypbien + sbati=0 + sterr>0 + croisement RPG

---

## 3. FONCTIONNALITÉS

### Must Have (démo SAFER)
1. **Dashboard "Vue d'ensemble"** — Carte choropleth des 24 communes colorée par prix/m², filtres commune/année/type de bien, chiffres clés (volume, prix médian, évolution)
2. **Dashboard "Focus agricole"** — Transactions foncier agricole, prix par type de culture (canne, maraîchage, élevage via RPG), comparaison aux barèmes officiels
3. **Dashboard "Risques"** — Impact des zones Géorisques sur les prix, carte des risques × transactions
4. **Chat IA Text-to-SQL** — Interface Streamlit en français, question → SQL → résultat formaté, 15-20 questions pré-testées
5. **Pipeline d'ingestion DVF** — Téléchargement, chargement PostgreSQL, transformations dbt (staging → marts)
6. **Infrastructure Docker** — Tous les services conteneurisés, accessible en HTTPS via Nginx

### Should Have (post-démo)
7. **Dashboard "Tendances"** — Évolutions temporelles, détection de pression foncière
8. **Enrichissement DPE/BDNB** — Impact classe énergie sur prix
9. **Pipeline Kestra orchestré** — Automatisation semestrielle complète
10. **Rapports exportables** — Export PDF des dashboards pour le comité technique

### Could Have (si contrat signé)
11. **Intégration DV3F** (données enrichies SAFER via Cerema)
12. **Alertes automatiques** de pression foncière
13. **Dashboard GFA** — Suivi du portefeuille des 37 GFA

---

## 4. CONTRAINTES TECHNIQUES

### Infrastructure
- **VPS OVH** : 4 vCores / 8 GB RAM / 75 GB SSD — déjà opérationnel
- **Budget infra** : ~7€/mois max
- **OpenClaw/DeepSeek** : Déjà installé sur le VPS pour le chat IA
- **Déploiement** : Docker Compose (tout conteneurisé)

### Stack imposée
| Composant | Technologie | Justification |
|-----------|-------------|---------------|
| Base de données | PostgreSQL + PostGIS | DVF+ livré en SQL PostgreSQL, données géographiques |
| Transformations | dbt (dbt-postgres) | Zoomcamp Module 4, modèle Kimball, testable |
| Orchestration | Kestra | Zoomcamp Module 2, batch semestriel |
| Dashboards | Metabase | Open source, SQL natif, carte choropleth, filtres |
| Chat IA | Streamlit + LangChain + DeepSeek | Text-to-SQL en français, déjà familier |
| Reverse proxy | Nginx + Let's Encrypt | HTTPS obligatoire pour la démo |
| Conteneurisation | Docker Compose | Reproductibilité, déploiement simple |

### Contraintes mémoire VPS (8 GB total)
| Service | RAM estimée |
|---------|-------------|
| PostgreSQL + PostGIS | ~1-1.5 GB |
| Metabase | ~1 GB |
| Streamlit + LangChain | ~500 MB |
| Kestra | ~1.5 GB |
| Nginx | ~50 MB |
| OS + marge | ~3-4 GB |

### Réseau
- Domaine cible : `dashboard.reunia.re`
  - `/` → Metabase (dashboards)
  - `/chat` → Streamlit (chat IA)
- HTTPS obligatoire (Let's Encrypt)
- Accès protégé par mot de passe

---

## 5. CONTRAINTES BUSINESS

### Délai
- **RDV 1 (écoute)** : Dans les prochaines semaines (date pas encore fixée)
- **RDV 2 (démo)** : 2-3 semaines après le RDV 1
- **Objectif réaliste** : Démo fonctionnelle d'ici fin mars / début avril 2026
- **Double objectif** : Démo client + Projet final Zoomcamp DE 2026

### Ce qui doit être impeccable pour la démo
1. Les 3 dashboards Must Have (vue d'ensemble, agricole, risques) — fluides, < 3s de chargement
2. Le chat IA qui répond correctement à 15+ questions en français
3. La carte choropleth des 24 communes (GeoJSON requis)
4. URL HTTPS accessible et protégée

### Ce qui peut être simplifié pour la démo
- Kestra : un flow basique suffit (pas besoin de scheduling complet)
- Enrichissements : RPG + Géorisques minimum. DPE/BDNB en bonus.
- Pipeline : chargement one-shot OK (pas besoin d'incremental pour la démo)

---

## 6. NON-FONCTIONNEL

### Performance
- Dashboards Metabase : < 3 secondes de chargement
- Chat IA : < 10 secondes par réponse (incluant appel DeepSeek)
- Ingestion DVF complète 974 : < 30 minutes

### Sécurité
- PostgreSQL : user read-only pour Metabase et le chat IA, user admin séparé
- Chat IA : validation SQL (SELECT uniquement), timeout 30s, rate limiting
- Pas de données personnelles exposées (DVF = données agrégées, mais attention aux adresses)
- Secrets dans .env, jamais dans le code

### Maintenabilité
- Code documenté, structure claire
- dbt tests sur tous les modèles
- Docker Compose reproductible (docker compose up = ça marche)
- README complet pour reprise par un tiers

---

## 7. MODÈLE DE DONNÉES CIBLE (Kimball)

### Table de faits
**fct_transactions** (grain = 1 mutation foncière)
- Mesures : valeur_fonciere, surface_bati, surface_terrain, prix_m2, nombre_lots
- FK : commune_id, type_bien_id, date_id, parcelle_id, dpe_id (nullable)

### Dimensions
- **dim_communes** : code_insee, nom, population, revenus_medians, geometry (GeoJSON)
- **dim_types_biens** : code, libelle (maison, appartement, terrain, local, dépendance)
- **dim_dates** : date_mutation, annee, trimestre, mois, jour_semaine
- **dim_parcelles** : id_parcelle, section, surface, zone_risque (Géorisques), type_culture_rpg
- **dim_dpe** : classe_energie, classe_ges, consommation (si données disponibles)

### Structure dbt
```
models/
├── staging/         -- Nettoyage, typage, filtrage 974
│   ├── stg_dvf__mutations.sql
│   ├── stg_dvf__dispositions.sql
│   ├── stg_dvf__locaux.sql
│   ├── stg_rpg__parcelles.sql
│   └── stg_georisques__risques.sql
├── intermediate/    -- Jointures, enrichissements
│   └── int_transactions__enriched.sql
└── marts/           -- Modèle dimensionnel final
    ├── fct_transactions.sql
    ├── dim_communes.sql
    ├── dim_types_biens.sql
    ├── dim_dates.sql
    └── dim_parcelles.sql
```

---

## 8. STRUCTURE PROJET CIBLE

```
dvf-reunion/
├── .claude/
│   └── agents/              -- Agents multi-workflow
├── .env.example             -- Template variables d'environnement
├── docker-compose.yml       -- Orchestration des services
├── CLAUDE.md                -- Configuration projet Claude Code
├── PLAN.md                  -- Plan de développement (généré par l'architecte)
├── STATE.md                 -- État du projet (généré par l'architecte)
├── CORRECTIONS.md           -- Backlog corrections
│
├── docker/
│   ├── postgres/
│   │   └── init.sql         -- Création schemas raw/staging/marts + users
│   ├── metabase/
│   │   └── metabase.db      -- Config Metabase (si pré-configuré)
│   ├── nginx/
│   │   └── default.conf     -- Routing reverse proxy
│   └── streamlit/
│       └── Dockerfile       -- Image custom Streamlit + deps
│
├── ingestion/
│   ├── download_dvf.py      -- Téléchargement DVF+ 974
│   ├── load_dvf.py          -- Chargement PostgreSQL (raw)
│   ├── load_rpg.py          -- Chargement RPG
│   └── load_georisques.py   -- Appels API Géorisques
│
├── dbt_dvf/
│   ├── dbt_project.yml
│   ├── profiles.yml
│   ├── models/
│   │   ├── staging/
│   │   ├── intermediate/
│   │   └── marts/
│   ├── tests/
│   ├── macros/
│   └── seeds/               -- Barèmes officiels, mapping communes
│
├── streamlit_app/
│   ├── app.py               -- Chat IA principal
│   ├── prompts/
│   │   └── system_prompt.py -- Schéma DB + instructions Text-to-SQL
│   └── utils/
│       ├── db.py            -- Connexion PostgreSQL read-only
│       └── sql_validator.py -- Validation SELECT only + timeout
│
├── kestra/
│   └── flows/
│       └── dvf_pipeline.yml -- Flow orchestration semestriel
│
├── geojson/
│   └── communes_974.geojson -- Contours 24 communes pour carte Metabase
│
├── tests/
│   ├── test_ingestion.py
│   ├── test_sql_validator.py
│   └── qa/                  -- Tests indépendants QA
│
├── REPORTS/                  -- Rapports agents (dev, qa, validation)
├── docs/
│   ├── BRIEF.md             -- Ce document
│   ├── REFERENCE.md         -- Contexte métier SAFER
│   ├── DATA_DICTIONARY.md   -- Dictionnaire de données
│   ├── ARCHITECTURE.md      -- Architecture technique
│   ├── PIPELINE.md          -- Documentation pipeline
│   └── DEPLOYMENT.md        -- Guide déploiement VPS
│
└── requirements.txt         -- Dépendances Python
```

---

## 9. RÉSUMÉ POUR L'ARCHITECTE

**Ce projet est un data product complet** qui va de l'ingestion de données brutes jusqu'à l'exposition via des dashboards et un chat IA. L'architecte doit découper en parties qui suivent le flux naturel des données :

1. Infrastructure (Docker Compose + PostgreSQL)
2. Ingestion (DVF → raw PostgreSQL)
3. Transformations (dbt staging → marts)
4. Enrichissements (RPG, Géorisques)
5. Exposition (Metabase dashboards)
6. Chat IA (Streamlit + Text-to-SQL)
7. Orchestration (Kestra)
8. Déploiement (Nginx + SSL)

**Priorité absolue** : les parties 1-6 doivent être fonctionnelles pour la démo SAFER.
Les parties 7-8 sont importantes mais simplifiables.

**Contrainte principale** : VPS 8 GB RAM, tous les services doivent coexister.