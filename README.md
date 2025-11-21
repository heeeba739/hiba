
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
## 2️⃣ Étapes dans Power BI (Power Query)

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

## 3️⃣ Champs identifiés pour le projet

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
## 4️⃣ Observations initiales

- Tous les pays n’ont pas forcément toutes les années disponibles.  
- Certaines valeurs sont nulles ou manquantes → seront traitées à l’étape 2.  
- Les clés de fusion principales : **ISO3 + Year** pour la table FactIndicateurs.  
- Types de données ajustés pour correspondre entre les tables.

----
## 5️⃣ Prochaines étapes

1. Nettoyage des données et gestion des valeurs nulles (Power Query).  
2. Création des colonnes calculées utiles (région simplifiée, population classée, ratio CO₂/PIB…).  
3. Fusion des tables sur la clé **ISO3** pour construire la table de faits.

----
##Partie Nettoyage
## 1️⃣ Tableau des transformations

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

