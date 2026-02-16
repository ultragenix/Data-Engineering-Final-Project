# Projet Final DE Zoomcamp — Sources de Données DVF La Réunion

> Document de référence compilé pour le projet final.
> Objectif : Analyse du marché foncier et immobilier de La Réunion.

---

## 1. Source Principale — DVF (Demandes de Valeurs Foncières)

### 1A. DVF Brut (DGFiP)
- **Éditeur** : Direction Générale des Finances Publiques
- **Contenu** : Toutes les transactions immobilières et foncières en France (ventes, échanges, adjudications)
- **Champs clés** : date mutation, nature mutation, valeur foncière, adresse, code postal, commune, type de local, surface réelle bâti, nombre de pièces, surface terrain, section/numéro de parcelle cadastrale
- **Format** : CSV (fichiers annuels, ~500 Mo compressé pour la France entière)
- **Période** : 2014 à aujourd'hui (mise à jour semestrielle, dernière : octobre 2025)
- **Licence** : Licence Ouverte 2.0
- **URL** : https://www.data.gouv.fr/datasets/demandes-de-valeurs-foncieres
- **Téléchargement direct** : https://cadastre.data.gouv.fr/dvf
- **Filtrage La Réunion** : département = 974

### 1B. DVF Géolocalisé (Etalab)
- **Contenu** : Même données DVF, normalisées et enrichies avec coordonnées GPS (lat/lon)
- **Avantage** : Format nettoyé, géolocalisé via cadastre, champs normalisés, identifiant de mutation
- **Format** : CSV.GZ (~494 Mo)
- **URL** : https://www.data.gouv.fr/datasets/demandes-de-valeurs-foncieres-geolocalisees
- **Recommandation** : ⭐ **Utiliser cette version plutôt que le DVF brut** — gain de temps énorme sur le nettoyage

### 1C. DVF+ Open Data (Cerema/DGALN)
- **Contenu** : DVF restructuré en modèle relationnel (table mutation = 1 ligne par vente), géolocalisé
- **Avantage** : Modèle de données propre, disponible en SQL PostgreSQL/PostGIS, GeoPackage, ou CSV
- **URL** : https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data
- **Recommandation** : ⭐ **Version la plus aboutie** pour un projet data engineering (déjà structurée)

### 1D. Explorateur DVF (Application web)
- **Usage** : Visualisation interactive des transactions, utile pour l'exploration préliminaire
- **URL** : https://app.dvf.etalab.gouv.fr/

---

## 2. Enrichissement — Données Cadastrales et Bâtiments

### 2A. Plan Cadastral Informatisé (PCI)
- **Éditeur** : DGFiP / Etalab
- **Contenu** : Contours des parcelles, sections cadastrales, emprise des bâtiments
- **Format** : GeoJSON, EDIGEO, DXF (par commune ou département)
- **Usage projet** : Jointure avec DVF via identifiant parcelle cadastrale
- **URL** : https://cadastre.data.gouv.fr/datasets/plan-cadastral-informatise

### 2B. BDNB — Base de Données Nationale des Bâtiments
- **Éditeur** : CSTB (Centre Scientifique et Technique du Bâtiment)
- **Contenu** : Carte d'identité de 32 millions de bâtiments (400+ attributs) : morphologie, année construction, DPE estimé, surface, hauteur, matériaux, risques
- **Format** : CSV, GeoPackage, PostgreSQL dump — **téléchargeable par département (974)**
- **Usage projet** : Enrichir chaque transaction DVF avec les caractéristiques du bâtiment
- **URL** : https://bdnb.io/download/
- **Data.gouv** : https://www.data.gouv.fr/datasets/base-de-donnees-nationale-des-batiments

### 2C. DPE — Diagnostics de Performance Énergétique
- **Éditeur** : ADEME
- **Contenu** : Classe énergétique (A-G), classe GES, consommation estimée, caractéristiques du logement
- **Jeux de données** :
  - DPE Logements existants (depuis juillet 2021) : https://data.ademe.fr/datasets/dpe03existant
  - DPE Logements neufs (depuis juillet 2021) : https://data.ademe.fr/datasets/dpe02neuf
  - DPE anciens (avant juillet 2021) : https://data.ademe.fr/datasets/dpe-france
- **Accès** : Vue tabulaire, téléchargement PostgreSQL, **API REST**
- **Usage projet** : Croiser la valeur foncière avec la performance énergétique → impact du DPE sur les prix

---

## 3. Enrichissement — Données Géographiques et Risques

### 3A. Base Adresse Nationale (BAN)
- **Éditeur** : IGN / DINUM / ANCT
- **Contenu** : 25 millions d'adresses géolocalisées (coordonnées, code INSEE, voie)
- **Usage projet** : Géocodage des adresses DVF qui n'ont pas de coordonnées
- **API** : Service de géocodage Géoplateforme (remplace api-adresse.data.gouv.fr)
- **Téléchargement CSV** : https://adresse.data.gouv.fr/outils/api-doc/adresse
- **Limite API** : 50 appels/seconde/IP

### 3B. Géorisques — Risques Naturels et Technologiques
- **Éditeur** : BRGM / Ministère de la Transition Écologique
- **Contenu** : Zonages inondation, retrait-gonflement argiles, sismique, cavités, ICPE, sites pollués
- **Pertinence La Réunion** : Risques cycloniques, volcaniques, mouvements de terrain
- **Accès** : API REST (v2 avec jeton) + téléchargement direct
- **URL API** : https://www.georisques.gouv.fr/doc-api
- **URL bases** : https://www.georisques.gouv.fr/donnees/bases-de-donnees
- **Usage projet** : Croiser les transactions avec les zones à risque → impact sur les prix

---

## 4. Enrichissement — Données Agricoles (angle SAFER)

### 4A. RPG — Registre Parcellaire Graphique
- **Éditeur** : IGN / ASP (Agence de Services et de Paiement)
- **Contenu** : Contours des parcelles agricoles déclarées PAC, type de culture (canne à sucre, vergers, prairies...)
- **Format** : GeoPackage, Shapefile (par région ou national)
- **Téléchargement** : https://geoservices.ign.fr/rpg
- **Data.gouv** : https://www.data.gouv.fr/datasets/rpg
- **Édition 2024** : 8 bases de données (parcelles, prairies permanentes, surfaces non agricoles, bio, IAE...)
- **Usage projet** : Identifier les transactions portant sur du foncier agricole, analyser par type de culture

### 4B. Parcelles Agriculture Biologique
- **Éditeur** : Agence Bio
- **Contenu** : Parcelles déclarées en agriculture bio et en conversion (PAC 2019-2023)
- **Couverture** : France + DOM dont La Réunion (EPSG 2975)
- **URL** : https://www.data.gouv.fr/datasets/parcelles-en-agriculture-biologique-ab-declarees-a-la-pac
- **Usage projet** : Prime de prix pour le foncier agricole bio ?

### 4C. Barème des Prix des Terres Agricoles
- **Éditeur** : Ministère de l'Agriculture (arrêté annuel)
- **Contenu** : Valeur vénale moyenne des terres agricoles par région (€/hectare)
- **Dernière mise à jour Réunion** : Arrêté du 26 août 2025
- **Accès** : Site SAFER Réunion → https://www.safer-reunion.fr/foncier
- **Usage projet** : Référentiel de comparaison avec les prix DVF réels

---

## 5. Enrichissement — Données Socio-Économiques

### 5A. Données Communales INSEE
- **Contenu** : Population, revenus, logements, emploi par commune
- **Sources** :
  - Population millésimée : https://data.regionreunion.com/explore/dataset/population-francaise-communespublic/
  - Référentiel communes : https://data.regionreunion.com/explore/dataset/communes-millesime-france/
  - Codes postaux : https://data.regionreunion.com/explore/dataset/laposte_hexasmaldatanova/

### 5B. Portail Open Data La Réunion
- **Éditeur** : Région Réunion
- **Contenu** : Multiples jeux de données locaux (transport, économie, environnement, social)
- **URL** : https://data.regionreunion.com/explore/
- **Jeux intéressants** :
  - **Potentiel foncier** : Espaces non urbanisés identifiés par croisement SIG → https://data.regionreunion.com/explore/dataset/potentiel-foncier/

### 5C. AGORAH — Agence d'Urbanisme de La Réunion
- **Contenu** : Données d'observation foncière, aménagement du territoire, habitat
- **URL** : https://www.agorah.com/index.php/open-data/

### 5D. EDF Réunion Open Data
- **Contenu** : Consommation électrique, mix énergétique, réseaux BT/HTA
- **URL** : https://opendata-reunion.edf.fr/
- **Usage** : Enrichissement secondaire (consommation énergie par zone)

---

## 6. Architecture Projet Suggérée

```
SOURCES                    INGESTION              WAREHOUSE           TRANSFORMATION        VISUALISATION
────────                   ─────────              ─────────           ──────────────        ─────────────
DVF+ (CSV/SQL)     ──┐
Cadastre (GeoJSON) ──┤
BDNB (CSV)         ──┤    Docker/Kestra    →    BigQuery      →    dbt               →   Looker Studio
RPG (GeoPackage)   ──┤    (Module 1-2)          (Module 3)         (Module 4)             ou Metabase
INSEE (CSV/API)    ──┤                          Partitioning       Staging → Marts
DPE (API/CSV)      ──┤                          Clustering         Tests + Docs
Géorisques (API)   ──┘
```

### Modèle Dimensionnel Proposé (Kimball)

**Table de faits** : `fct_transactions`
- Grain = 1 transaction foncière
- Mesures : valeur_fonciere, surface_bati, surface_terrain, prix_m2
- Clés étrangères vers dimensions

**Dimensions** :
- `dim_communes` (code INSEE, nom, population, revenus médians)
- `dim_types_biens` (maison, appartement, terrain, local, dépendance)
- `dim_dates` (date mutation → année, trimestre, mois, jour semaine)
- `dim_parcelles` (section cadastrale, surface, zone risque, type culture RPG)
- `dim_dpe` (classe énergie, classe GES, consommation)

---

## 7. Clés de Jointure entre Sources

| Source A | Source B | Clé de jointure |
|----------|----------|-----------------|
| DVF | Cadastre | `id_parcelle` (section + numéro) |
| DVF | BDNB | `batiment_groupe_id` via parcelle cadastrale |
| DVF | DPE | Adresse (géocodage BAN) ou parcelle |
| DVF | RPG | Intersection géospatiale parcelle |
| DVF | Géorisques | Code INSEE commune ou coordonnées |
| DVF | INSEE | Code INSEE commune |
| DVF | Potentiel foncier | Intersection géospatiale |

---

## 8. Points d'Attention

1. **RGPD** : Les données DVF contiennent des données à caractère personnel — pas de ré-identification des personnes, pas d'indexation moteur de recherche
2. **Couverture La Réunion** : Vérifier que chaque source couvre bien le département 974 (la plupart le font mais pas toujours les DOM)
3. **Volume** : DVF France = ~500 Mo. Filtrer sur département 974 réduit drastiquement le volume — idéal pour un projet Zoomcamp
4. **Temporalité** : DVF disponible de 2014 à 2025, RPG depuis 2007, DPE depuis 2013 — bien aligner les périodes
5. **Projection géographique** : La Réunion utilise EPSG:2975 (RGR92 / UTM zone 40S), pas le Lambert-93 métropole