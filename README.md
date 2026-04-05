# 🌿 Verde Parashop — Business Intelligence Dashboard

> Tableau de bord analytique complet pour une parapharmacie tunisienne
> conçu pour augmenter la marge, comprendre les profils clients
> et optimiser l'impact des promotions sur les besoins santé.

---

## 📋 Contexte métier

Verde Parashop est une parapharmacie dont les objectifs principaux sont :
- Augmenter les revenus et la marge brute
- Améliorer la relation client via l'analyse des besoins santé
- Mesurer l'impact des promotions sur les ventes
- Identifier les besoins santé les plus fréquents
- Comprendre les profils clients selon le sexe et la tranche d'âge

---

## 🏗️ Architecture Data Warehouse

- **SGBD** : SQL Server 2019 (etoileDW)
- **Schéma** : Étoile (Star Schema)
- **Déploiement** : Power BI Service avec passerelle On-premises personnelle

### Tables

| Table | Type | Description |
|-------|------|-------------|
| factBesoin | Fact | 26K+ transactions clients |
| Dim_BesoinMarcheInter | Dimension | Clients · Produits · Besoins · Catégories |
| Dim_Temps | Dimension | Calendrier avec hiérarchie Année → Mois → Jour |
| TypePromo | Dimension | Types de promotions |

### Relations
- factBesoin[BesoinKey] → Dim_BesoinMarcheInter[BesoinKey] (Many-to-One)
- factBesoin[Date_FK] → Dim_Temps[Date_PK] (Many-to-One)

---

## 📊 Pages du rapport

### Page 1 — TendanceMarche
**Objectif** : Vue globale des performances commerciales

| Visuel | Description |
|--------|-------------|
| 3 Cards KPI | Quantite Vendue · CA_YTD · CA_YTD_LY |
| Gauge | Taux de Marge avec seuil cible 30% |
| KPI SDG3 | Aligné ODD3 — part CA produits santé cible 25% |
| Decomposition Tree | CA par Sexe → Tranche Age → Besoin Specifique |
| Bar Chart | CA par Besoin Specifique |
| Q&A Assistant IA | Questions en langage naturel |

### Page 2 — SourceMotivationMarche
**Objectif** : Comprendre les profils et motivations clients

| Visuel | Description |
|--------|-------------|
| Table RANKX | Classement clients par CA avec rang Dense |
| Matrix Heatmap | Besoin Specifique × Tranche Age |
| Donut Chart | CA par Client |
| Bar Chart | CA par Produit Associe |

### Page 3 — ImpcatePromotions
**Objectif** : Mesurer l'impact des promotions sur les ventes

| Visuel | Description |
|--------|-------------|
| Card rouge | % Ventes Sous Promo |
| Card | Qte Vendue Avec Promo |
| Card | Qte Vendue Sans Promo |
| Scatter Chart | PayedAmount × DiscountPourcentage par Besoin |
| Clustered Bar | Qte Vendue Avec/Sans Promo par Besoin Specifique |
| Pie Chart TopN | Top N produits sous promo — dynamique via slicer |

### Page 4 — MarcheInterne
**Objectif** : Analyser le stock et détecter les anomalies

| Visuel | Description |
|--------|-------------|
| 4 Cards | Clients Acheteur · CA_YTD · Quantite Vendue · Stock Marche |
| Line Chart | Évolution stock avec Anomaly Detection IA |
| Waterfall | Stock par Besoin Specifique |
| Line Chart | CA par Mois |
| Q&A Assistant IA | Analyse interactive en langage naturel |

---

## 📐 Mesures DAX

### Agrégations de base
```dax
CA_Total = SUM(factBesoin[PayedAmount])

Quantite Vendue = COUNTROWS(factBesoin)

Clients Acheteur = DISTINCTCOUNT(Dim_BesoinMarcheInter[Client])

Marge_Brute = 
SUMX(
    FILTER(factBesoin, factBesoin[PayedAmount] > 0 && factBesoin[PrixTotalHt] > 0),
    factBesoin[PayedAmount] - factBesoin[PrixTotalHt]
)

Taux_Marge = DIVIDE([Marge_Brute], SUM(factBesoin[PayedAmount]), 0)
```

### Time Intelligence
```dax
CA_YTD = 
CALCULATE(
    [Chiffre d affaire],
    DATESYTD(factBesoin[DocumentDate])
)

CA_YTD_LY = 
CALCULATE(
    [Chiffre d affaire],
    FILTER(
        ALL(factBesoin[DocumentDate]),
        YEAR(factBesoin[DocumentDate]) = YEAR(TODAY()) - 1 &&
        MONTH(factBesoin[DocumentDate]) <= MONTH(TODAY())
    )
)

Variation_YTD = DIVIDE([CA_YTD] - [CA_YTD_LY], [CA_YTD_LY], 0)
```

### Ranking & Top N
```dax
Rang_Client = 
RANKX(
    ALLSELECTED(Dim_BesoinMarcheInter[Client]),
    [Chiffre d affaire],
    ,
    DESC,
    Dense
)

TopN_Produit_Promo = 
IF(
    RANKX(
        ALLSELECTED(Dim_BesoinMarcheInter[Produit_Associe]),
        [Remise_Totale],
        ,
        DESC,
        Dense
    ) <= [N_Selectionne],
    [Remise_Totale]
)
```

### Promotions
```dax
Qte Vendue Avec Promo = 
CALCULATE(COUNTROWS(factBesoin), factBesoin[DiscountPourcentage] > 0)

Qte Vendue Sans Promo = 
CALCULATE(COUNTROWS(factBesoin), factBesoin[DiscountPourcentage] = 0)

% Ventes Sous Promo = 
DIVIDE([Qte Vendue Avec Promo], COUNTROWS(factBesoin), 0)

Remise_Totale = 
SUMX(factBesoin, factBesoin[PrixTotalHt] * (factBesoin[DiscountPourcentage] / 100))
```

### SDG3 — Santé & Bien-être
```dax
CA_Sante = 
CALCULATE(
    [Chiffre d affaire],
    Dim_BesoinMarcheInter[Categorie_Parent] IN {
        "Carences & Compléments",
        "Nutrition & Alimentation",
        "Santé Bébé & Enfant",
        "Dispositifs Médicaux"
    }
)

Pct_SDG3 = DIVIDE([CA_Sante], [Chiffre d affaire], 0)

Cible_SDG3 = 0.25
```

### VAR Chaining
```dax
Analyse_Client = 
VAR CA_Client = [Chiffre d affaire]
VAR Moy_Generale = 
    CALCULATE(
        [Chiffre d affaire],
        ALL(Dim_BesoinMarcheInter[Client])
    )
VAR Ratio = DIVIDE(CA_Client, Moy_Generale, 0)
RETURN
IF(Ratio > 1, "Au dessus moyenne", "En dessous moyenne")
```

### Stock Marché
```dax
Stock_Marche = 
SUMX(
    VALUES(factBesoin[BesoinKey]),
    CALCULATE(MAX(factBesoin[QuantiteMarchenterne]))
)
```

---

## 🔒 Sécurité — Row Level Security (RLS)

| Rôle | Accès | Filtre DAX |
|------|-------|------------|
| Pharmacien | Complet | Aucun |
| Commercial | Limité | `[Categorie_Parent] = "Hygiène & Toilette"` |

---

## 🎯 Alignement SDG

**ODD 3 — Bonne Santé et Bien-être**
- KPI dédié mesurant la part du CA sur les produits santé
- Seuil cible : 25% du CA total
- Catégories concernées : Carences & Compléments · Nutrition & Alimentation · Santé Bébé & Enfant · Dispositifs Médicaux

---

## 🤖 Visuels IA

| Visuel | Usage |
|--------|-------|
| Anomaly Detection | Détection automatique des pics anormaux de stock |
| Q&A Assistant | Analyse interactive en langage naturel |
| Decomposition Tree | Drill automatique CA par segment |

---

## ⚡ Power Automate

Flow configuré pour envoyer un email automatique avec :
- Trigger : bouton Power BI cliqué
- Données dynamiques : Qte Vendue Avec/Sans Promo
- Destinataire : Responsable Marketing Verde Parashop

---

## 🚀 Déploiement

- **Power BI Desktop** : développement et modélisation
- **Power BI Service** : publication et partage
- **Passerelle** : On-premises Personal Mode (Hamza.Rekik)
- **Source** : SQL Server etoileDW via connexion directe

---

