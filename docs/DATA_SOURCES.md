# 📦 Sources de Données — DVF+ La Réunion

> Référence complète de toutes les sources de données utilisées ou envisagées pour le projet.
> Pour le résumé projet, voir le [README.md](../README.md).

---

## 1. Source Principale — DVF

### 1A. DVF Brut (DGFiP)
- **Éditeur** : Direction Générale des Finances Publiques
- **Contenu** : Toutes les transactions immobilières et foncières en France
- **Format** : CSV (~500 Mo compressé France entière)
- **Période** : 2014 à aujourd'hui (MAJ semestrielle)
- **Licence** : Licence Ouverte 2.0
- **URL** : https://www.data.gouv.fr/datasets/demandes-de-valeurs-foncieres
- **Téléchargement** : https://cadastre.data.gouv.fr/dvf
- **Statut projet** : ❌ Non utilisé (préférer DVF+ Cerema)

### 1B. DVF Géolocalisé (Etalab)
- **Contenu** : DVF normalisé + coordonnées GPS
- **Format** : CSV.GZ (~494 Mo)
- **URL** : https://www.data.gouv.fr/datasets/demandes-de-valeurs-foncieres-geolocalisees
- **Statut projet** : ❌ Non utilisé (préférer DVF+ Cerema)

### 1C. DVF+ Open Data (Cerema/DGALN) — ⭐ SOURCE RETENUE
- **Contenu** : DVF restructuré en modèle relationnel (17 tables), géolocalisé
- **Format** : SQL PostgreSQL/PostGIS, GeoPackage, CSV
- **Volume 974** : ~60 000 transactions (2014-2024)
- **MAJ** : 2x/an (avril + octobre). Prochaine : avril 2026
- **URL** : https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data
- **API** : https://apidf-preprod.cerema.fr/swagger/
- **Statut projet** : ✅ Source principale

### 1D. Explorateur DVF (Application web)
- **Usage** : Exploration visuelle préliminaire
- **URL** : https://app.dvf.etalab.gouv.fr/

### 1E. DV3F (Cerema — accès restreint)
- **Contenu** : DVF+ enrichi avec données supplémentaires
- **Accès** : Réservé SAFER et collectivités via convention Cerema
- **Statut projet** : 🔒 Phase ultérieure (si contrat SAFER signé)

---

## 2. Données Cadastrales et Bâtiments

### 2A. Plan Cadastral Informatisé (PCI)
- **Éditeur** : DGFiP / Etalab
- **Contenu** : Contours parcelles, sections cadastrales, emprise bâtiments
- **Format** : GeoJSON, EDIGEO, DXF (par commune ou département)
- **Jointure** : `id_parcelle` (section + numéro)
- **URL** : https://cadastre.data.gouv.fr/datasets/plan-cadastral-informatise
- **Statut projet** : 🟡 Should (géométrie parcelles)

### 2B. BDNB — Base de Données Nationale des Bâtiments
- **Éditeur** : CSTB
- **Contenu** : 32M bâtiments, 400+ attributs (morphologie, année construction, DPE estimé, surface, hauteur, matériaux)
- **Format** : CSV, GeoPackage, PostgreSQL dump — **téléchargeable par département (974)**
- **Jointure** : `batiment_groupe_id` via parcelle cadastrale
- **URL** : https://bdnb.io/download/
- **Data.gouv** : https://www.data.gouv.fr/datasets/base-de-donnees-nationale-des-batiments
- **Statut projet** : 🟡 Should (enrichissement bâti)

### 2C. DPE — Diagnostics de Performance Énergétique
- **Éditeur** : ADEME
- **Contenu** : Classe énergie A-G, classe GES, consommation estimée
- **Jeux de données** :
  - Logements existants (depuis juillet 2021) : https://data.ademe.fr/datasets/dpe03existant
  - Logements neufs (depuis juillet 2021) : https://data.ademe.fr/datasets/dpe02neuf
  - Anciens (avant juillet 2021) : https://data.ademe.fr/datasets/dpe-france
- **Accès** : Téléchargement + API REST
- **Jointure** : Adresse (géocodage BAN) ou parcelle
- **⚠️ Vigilance** : Volume très faible en outre-mer. Plan B = BDNB (DPE estimé)
- **Statut projet** : 🟡 Should (à vérifier volume 974)

---

## 3. Données Géographiques et Risques

### 3A. Base Adresse Nationale (BAN)
- **Éditeur** : IGN / DINUM / ANCT
- **Contenu** : 25M adresses géolocalisées
- **API** : Service de géocodage Géoplateforme (remplace api-adresse.data.gouv.fr)
- **Limite** : 50 appels/seconde/IP
- **URL doc** : https://adresse.data.gouv.fr/outils/api-doc/adresse
- **Statut projet** : 🟡 Should (géocodage complémentaire)

### 3B. Géorisques — Risques Naturels et Technologiques
- **Éditeur** : BRGM / Ministère Transition Écologique
- **Contenu** : Inondation, retrait-gonflement argiles, sismique, cavités, ICPE
- **Pertinence 974** : Cyclones, volcan (Piton de la Fournaise), mouvements de terrain, submersion marine
- **Accès** : API REST v2 (avec jeton) + téléchargement direct
- **URL API** : https://www.georisques.gouv.fr/doc-api
- **URL bases** : https://www.georisques.gouv.fr/donnees/bases-de-donnees
- **Jointure** : Code INSEE commune ou coordonnées GPS
- **Statut projet** : 🔴 Must (enrichissement critique pour la démo)

### 3C. GeoJSON Communes 974
- **Éditeur** : IGN
- **Contenu** : Contours des 24 communes de La Réunion
- **Usage** : Carte choropleth Metabase
- **Statut projet** : 🔴 Must (indispensable pour les dashboards)

---

## 4. Données Agricoles (angle SAFER)

### 4A. RPG — Registre Parcellaire Graphique
- **Éditeur** : IGN / ASP
- **Contenu** : Parcelles agricoles déclarées PAC, type de culture (canne, vergers, prairies...)
- **Format** : GeoPackage, Shapefile (par région)
- **Édition 2024** : 8 bases (parcelles, prairies permanentes, surfaces non agricoles, bio, IAE...)
- **URL** : https://geoservices.ign.fr/rpg
- **Data.gouv** : https://www.data.gouv.fr/datasets/rpg
- **Jointure** : Intersection géospatiale PostGIS (`ST_Intersects`)
- **Statut projet** : 🔴 Must (cœur de l'angle SAFER)

### 4B. Parcelles Agriculture Biologique
- **Éditeur** : Agence Bio
- **Contenu** : Parcelles bio et en conversion (PAC 2019-2023)
- **Couverture** : France + DOM dont La Réunion (EPSG:2975)
- **URL** : https://www.data.gouv.fr/datasets/parcelles-en-agriculture-biologique-ab-declarees-a-la-pac
- **Usage** : Analyser la prime de prix pour le foncier bio
- **Statut projet** : 🔵 Could (enrichissement secondaire)

### 4C. Barème des Prix des Terres Agricoles
- **Éditeur** : Ministère de l'Agriculture (arrêté annuel)
- **Contenu** : Valeur vénale moyenne des terres agricoles par région (€/ha)
- **Dernière MAJ Réunion** : Arrêté du 26 août 2025
- **URL** : https://www.safer-reunion.fr/foncier
- **Usage** : Référentiel de comparaison avec les prix DVF réels
- **Statut projet** : 🔴 Must (seed dbt pour comparaison aux barèmes officiels)

---

## 5. Données Socio-Économiques

### 5A. Données Communales INSEE
- **Contenu** : Population, revenus, logements, emploi par commune
- **Sources La Réunion** :
  - Population : https://data.regionreunion.com/explore/dataset/population-francaise-communespublic/
  - Référentiel communes : https://data.regionreunion.com/explore/dataset/communes-millesime-france/
  - Codes postaux : https://data.regionreunion.com/explore/dataset/laposte_hexasmaldatanova/
- **Jointure** : Code INSEE commune
- **Statut projet** : 🔵 Could (enrichissement dim_communes)

### 5B. Portail Open Data La Réunion
- **Éditeur** : Région Réunion
- **URL** : https://data.regionreunion.com/explore/
- **Jeux intéressants** :
  - Potentiel foncier (espaces non urbanisés via SIG) : https://data.regionreunion.com/explore/dataset/potentiel-foncier/
- **Statut projet** : 🔵 Could

### 5C. AGORAH — Agence d'Urbanisme de La Réunion
- **Contenu** : Observation foncière, aménagement, habitat
- **URL** : https://www.agorah.com/index.php/open-data/
- **⚠️ Note** : AGORAH utilise DVF depuis 2016 pour l'OTIF. Se positionner en complémentaire, jamais en concurrent.
- **Statut projet** : 🔵 Could (référence, pas source directe)

### 5D. EDF Réunion Open Data
- **Contenu** : Consommation électrique, mix énergétique, réseaux
- **URL** : https://opendata-reunion.edf.fr/
- **Statut projet** : 🔵 Could (enrichissement secondaire)

---

## 6. Clés de Jointure

| Source A | Source B | Clé de jointure |
|----------|----------|-----------------|
| DVF | Cadastre (PCI) | `id_parcelle` (section + numéro) |
| DVF | BDNB | `batiment_groupe_id` via parcelle cadastrale |
| DVF | DPE | Adresse (géocodage BAN) ou parcelle |
| DVF | RPG | Intersection géospatiale (`ST_Intersects`) |
| DVF | Géorisques | Code INSEE commune ou coordonnées GPS |
| DVF | INSEE | Code INSEE commune |
| DVF | Potentiel foncier | Intersection géospatiale |
| DVF | Parcelles Bio | Intersection géospatiale |

---

## 7. Points d'Attention

### RGPD
Les données DVF contiennent des informations liées à des transactions réelles. Bien que les noms des parties ne soient pas inclus, le croisement adresse + prix + date pourrait permettre une ré-identification. Ne jamais indexer sur les moteurs de recherche, ne jamais exposer les données à l'adresse individuelle sans protection.

### Couverture DOM
Toutes les sources ne couvrent pas systématiquement le 974. Toujours vérifier la disponibilité avant d'intégrer une source :
- ✅ DVF, RPG, Géorisques, BDNB, BAN, INSEE : confirmé 974
- ⚠️ DPE : très faible volume outre-mer
- ✅ Parcelles Bio : inclut La Réunion (EPSG:2975 mentionné)

### Projection Géographique
La Réunion utilise **EPSG:2975** (RGR92 / UTM zone 40S). Ne JAMAIS utiliser Lambert-93 (EPSG:2154) qui est réservé à la métropole. PostGIS : `ST_Transform(geom, 2975)`.

### Volumes Estimés (974)
| Source | Volume estimé |
|--------|--------------|
| DVF+ | ~60 000 mutations (2014-2024) |
| RPG | ~42 000 ha de parcelles |
| BDNB | ~150 000 bâtiments |
| Géorisques | 24 communes à interroger |
| DPE | À vérifier (potentiellement < 1 000) |

### Temporalité
| Source | Disponible depuis |
|--------|-------------------|
| DVF | 2014 |
| RPG | 2007 |
| DPE (nouveau format) | Juillet 2021 |
| BDNB | 2022 |
| Géorisques | Continue |

---

*Document compilé le 06/03/2026 — Référence interne projet DVF+ La Réunion*