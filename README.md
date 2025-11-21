
#  Global Development Dashboard — World Progress 2030

## Presentation d'quipe 
1_Leila Mourid

2_Chaimaa Maache 

3_Ayoub 

4_Hiba Azizi

#Repartition des taches:
<img width="1354" height="571" alt="image" src="https://github.com/user-attachments/assets/34552e4d-33ef-4720-b8f6-8b5ab07b4116" />


---

## 1.  Contexte du Projet
GDW pilote le programme **“World Progress 2030”** pour suivre le développement économique et environnemental mondial à partir de données ouvertes.

**Problème à résoudre :**  
- Identifier les zones de **croissance durable**  
- Détecter les **risques écologiques**  
- Orienter les **priorités de financement**  

**Objectif :**  
Créer un **tableau de bord Power BI** permettant de visualiser PIB, population et émissions de CO₂ par pays, avec des **KPIs pertinents** et un **storytelling clair**.


---

## 2.  Objectifs du projet
- Automatiser l’extraction des données depuis **deux APIs REST**  
- Nettoyer, croiser et modéliser des jeux de données hétérogènes  
- Créer un **modèle de données cohérent** entre indicateurs économiques et géographiques  
- Définir et calculer des **KPIs équilibrés** (performance / impact)  
- Produire un **rapport Power BI interactif** à 4 pages  
- Collaborer efficacement au sein d’une équipe de 4, avec documentation continue  

---
##  Étapes dans Power BI (Power Query)

1. **Importer l’API REST Countries**  
   - Power BI Desktop → `Transform Data`  
   - `Home → Get Data → Web`  
   - Coller l’URL : `https://restcountries.com/v3.1/all`  
   - Sélectionner **JSON** → Convertir en table → Dérouler les colonnes nécessaires  
   - Vérifier les types de colonnes (texte, nombre, booléen)  

2. **Importer l’API World Bank**  
   - Power BI Desktop → `Transform Data → Get Data → Web`  
   - URL exemple pour PIB : `https://api.worldbank.org/v2/country/all/indicator/NY.GDP.MKTP.CD?date=2015:2022&format=json`  
   - Convertir JSON en table → Dérouler colonnes → Filtrer années 2015-2022  
   - Sélectionner les colonnes importantes : Country ISO3, Year, Value
  
---

##  Champs identifiés pour le projet

| Table | Champ | Type | Description |
|-------|-------|------|------------|
| REST Countries | ISO3 | Texte | Code pays ISO 3 lettres, clé pour fusion |
| REST Countries | Name | Texte | Nom du pays |
| REST Countries | Region | Texte | Continent / région principale |
| REST Countries | Subregion | Texte | Sous-région |
| REST Countries | Languages | Texte | Langue(s) principale(s) |
| REST Countries | Area | Nombre | Superficie du pays (km²) |
| REST Countries | Independence | Booléen | Statut d’indépendance |
| World Bank | Country ISO3 | Texte | Code pays ISO3, clé pour fusion |
| World Bank | Year | Nombre | Année de l’indicateur |
| World Bank | Value | Nombre | Valeur de l’indicateur (PIB, CO₂, Population) |



----
##  Observations initiales

- Tous les pays n’ont pas forcément toutes les années disponibles.  
- Certaines valeurs sont nulles ou manquantes → seront traitées à l’étape 2.  
- Les clés de fusion principales : **ISO3 + Year** pour la table FactIndicateurs.  
- Types de données ajustés pour correspondre entre les tables.

----
##  Prochaines étapes

1. Nettoyage des données et gestion des valeurs nulles (Power Query).  
2. Création des colonnes calculées utiles (région simplifiée, population classée, ratio CO₂/PIB…).  
3. Fusion des tables sur la clé **ISO3** pour construire la table de faits.

----

##  Tableau des transformations

| Étape / Colonne | Anomalie détectée | Correction appliquée | Explication / justification métier |
|-----------------|-----------------|--------------------|----------------------------------|
| `Indicator ID` | Aucun problème technique | Aucun changement | Colonne conservée pour identifier chaque indicateur |
| `NY.GDP.MKTP.CD` | Valeurs nulles pour certains pays | Remplacer les valeurs nulls par la moyenne
| `EN.GHG.CO2.MT.CE.AR5` | Valeurs nulles pour certains pays ou continents |Remplacer les valeurs nulles par la moyenne de chaque continent  
| `ISO3Code` | Aucun problème | Conversion en type texte | Clé de fusion principale pour joindre avec autres tables |
| `Country Name` | Noms incohérents ou doublons possibles | Typage texte confirmé | Assure lisibilité dans le tableau de bord |
| `country.cca3` | Valeurs nulles | Suppression après vérification de la correspondance avec ISO3 | Évite colonnes inutiles après fusion avec table de référence `country` |
| `SP.POP.TOTL` | Aucun problème | Renommée en `Population`, type confirmé | Standardisation des noms pour cohérence métier |
| Pivot des indicateurs | Indicateurs sous forme verticale | Pivot des colonnes sur `Indicator ID` | Transforme la table en format “wide” adapté pour Power BI et calcul des KPI |
| Colonnes finales | Structure finale | Colonnes : `ISO3Code`, `Year`, `Country Name`, `PIB`, `Population`, `CO2` | Table prête pour analyses et création de mesures DAX dans Power BI |
--------------------------------
##  Notes générales

- **Fusion avec tables de référence** : `country`, `pib_remplacer`, `CO2_remplacer` pour compléter les valeurs manquantes.  
- **Suppression des lignes inutiles** : null ou vides (`ISO3Code`, `Value`) pour garantir la qualité de la table.  
- **Pivot des indicateurs** : convertit le format “long” API en format “wide” exploitable pour les visualisations et mesures DAX.  
- **Triage des colonnes** : seules les colonnes essentielles sont conservées, le reste est supprimé pour simplifier la table finale.

---
# 📝 Étape 2 – Transformation des données

## Objectif
Assurer la qualité et la cohérence des données extraites, gérer les valeurs manquantes, créer des colonnes calculées utiles et préparer la fusion avec les données contextuelles.

---

##  Tableau des transformations

| Étape / Colonne | Anomalie détectée | Correction appliquée | Explication / justification métier |
|-----------------|-----------------|--------------------|----------------------------------|
| Formats de colonnes | Certaines colonnes (ISO3Code, Year, Value) pouvaient être mal typées | Vérification et correction des types : texte pour ISO3Code, nombre pour Year et Value | Assure compatibilité avec les mesures DAX et les jointures |
| Valeurs nulles PIB | Certains pays avaient un PIB null | Remplacement par la moyenne disponible (`pib_remplacer.moy`) | Garantit que le calcul du PIB total et des KPI fonctionne correctement |
| Valeurs nulles CO₂ | Certaines valeurs nulles par pays ou continent | Remplacement par moyenne continentale (`CO2_remplacer`) | Permet d’avoir un indicateur CO₂ complet pour analyses comparatives |
| Doublons | Doublons possibles après fusion des tables | Suppression des doublons | Garantit l’unicité de chaque ligne par ISO3Code + Year |
| Colonnes calculées | Besoin de colonnes supplémentaires pour analyse | Création de colonnes : Région, Ratio CO₂/PIB, Population classée | Facilite le filtrage et le calcul de KPI dans le tableau de bord |
| Fusion des tables | Données économiques et données contextuelles séparées | Fusion avec table `country` sur ISO3Code | Permet d’avoir les informations pays complètes dans la table de faits |

---

# Notes générales

- Toutes les transformations ont été réalisées dans **Power Query**.  
- Les colonnes temporaires utilisées pour le remplacement de valeurs nulles ont été supprimées après usage.  
- Les colonnes calculées permettent un filtrage facile par région, sous-région, revenu et population.  

---

##  Journal de bord

 | Action  | Commentaire |
|--------|------------|
 | Vérification et correction des types de colonnes  | Assure la cohérence pour DAX et fusion |
 | Suppression doublons | Nettoyage des lignes redondantes |
 | Création colonnes calculées  | Ajout Région, Ratio CO₂/PIB, Population classée |
 | Fusion tables `FactIndicateurs` et `country`  | Jointure sur ISO3Code pour enrichir les données contextuelles |

---

# 📝 Étape 3 – Modélisation des données

## Objectif
Construire un modèle en étoile cohérent pour Power BI, avec tables de faits et de dimensions, afin de faciliter les calculs DAX et la création du tableau de bord.

---

##  Structure du modèle

| Table | Type | Clé principale | Clés secondaires / Relations |
|-------|------|----------------|----------------------------|
| FactIndicateurs | Fait | ISO3Code + Year | Liée à DimPays (ISO3Code), DimDate (Year) |
| DimPays | Dimension | ISO3Code | Liée à FactIndicateurs |
| DimDate | Dimension | Year | Liée à FactIndicateurs |
| DimRégion | Dimension | Région | Liée à DimPays (Région) |

---

##  Notes générales

- Le modèle suit une **structure en étoile**, idéale pour Power BI et DAX.  
- Les cardinalités ont été vérifiées : **un-à-plusieurs** entre dimensions et table de faits.  
- Toutes les relations sont activées et utilisées pour filtrer correctement les KPI sur les visualisations.  

---

##  Journal de bord

 | Action  | Commentaire |
 |--------|------------|
 | Création table DimPays  | Contient informations géographiques et contextuelles |
 | Création table DimDate  | Permet calculs temporels (MoM, YoY) |
 | Création table DimRégion  | Facilite le filtrage et comparaisons régionales |
 | Définition des relations  | Vérification des cardinalités et cohérence du modèle |

## les relations entre la table fait et les tables de dimentions

<img width="762" height="499" alt="image" src="https://github.com/user-attachments/assets/8eeaa8bd-67e2-4747-b615-a03443e8275e" />

-----------
# 📝 Étape 4 – Création des mesures (DAX)

## Objectif
Traduire les formules mathématiques des KPI en mesures DAX dans Power BI, en vérifiant la cohérence des unités et des formats.

---

##  Tableau des mesures DAX

| Thème | KPI | Description |
|-------|-----|-------------|
| Économie | PIB total (USD) | Valeur totale du PIB par année et par pays. |
|  | Croissance du PIB (%) | Variation du PIB d’une année sur l’autre. |
|  | PIB par habitant (USD) | PIB total / population. |
|  | Part du PIB régional (%) | Part du PIB d’une région dans le total mondial. |
|  | Évolution du PIB depuis 2015 (%) | Croissance cumulée du PIB depuis 2015. |
| Population & Démographie | Population totale | Somme des habitants. |
|  | Croissance démographique (%) | Variation annuelle de la population. |
|  | Densité de population | Population / superficie du pays. |
| Environnement | Émissions totales de CO₂ (kt) | Quantité totale annuelle de CO₂ émise. |
|  | Émissions par habitant (t/hab) | CO₂ total / population. |
|  | Intensité carbone (CO₂/PIB) | Émissions de CO₂ / PIB total. |
|  | Évolution des émissions CO₂ depuis 2015 (%) | Taux de variation cumulée depuis 2015. |
| Développement durable | PIB / CO₂ ratio | Efficience économique (production par tonne de CO₂). |
|  | PIB / Population / Superficie | Indicateur composite de productivité par km². |
|  | Espérance de vie moyenne* | (si ajoutée via World Bank) Indicateur social complémentaire. |
| Comparatifs régionaux | Classement mondial par PIB | Rang du pays en PIB global. |
|  | Classement par intensité carbone | Rang du pays selon pollution par unité économique. |




# 📝 Étape 5 – Construction du tableau de bord Power BI

## Objectif
Créer un tableau de bord interactif de 4 pages, permettant de visualiser les KPI et d’explorer les données selon différents segments.

---

##  Pages de visualisation

| Page | Objectif | Visualisations clés |
|------|----------|-------------------|
| Monde | Vue globale des indicateurs | Carte du monde, graphiques d’évolution du PIB, population et CO₂, indicateurs clés |
| Région | Comparaison par continents et sous-régions | Histogrammes, cartes choroplèthes, KPI résumés par région |
| Pays | Fiche pays détaillée | Tableaux et cartes par pays, évolution du PIB, CO₂, population |
| Corrélation & Durabilité | Analyse de l’impact économique vs environnemental | Graphiques de dispersion PIB vs CO₂, ratio CO₂/PIB, tendances par région |
 

