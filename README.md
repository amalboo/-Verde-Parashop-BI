# -Verde-Parashop-BI
Dashboard Power BI - Parapharmacie Verde Parashop - Analyse besoins santé et promotions

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
