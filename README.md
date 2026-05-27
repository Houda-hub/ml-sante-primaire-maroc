# 🏥 Clustering des Établissements de Soins de Santé Primaire au Maroc (2024)

> **Projet Machine Learning – Apprentissage Non-Supervisé**  
> Dataset : Répartition des Établissements de Soins de Santé Primaire – Maroc 2024

---

## 📌 Table des Matières

1. [Description du Projet](#-description-du-projet)
2. [Dataset](#-dataset)
3. [Structure du Repo](#-structure-du-repo)
4. [Prérequis & Installation](#-prérequis--installation)
5. [Pipeline ML](#-pipeline-ml)
6. [Algorithmes Appliqués](#-algorithmes-appliqués)
7. [Résultats & Évaluation](#-résultats--évaluation)
8. [Visualisations](#-visualisations)
9. [Interprétation](#-interprétation)
10. [Conclusion](#-conclusion)

---

## 🎯 Description du Projet

Ce projet applique les techniques d'**apprentissage non-supervisé** sur un dataset réel des établissements de soins de santé primaire au Maroc pour l'année 2024.

**Objectif :** Identifier des regroupements naturels (clusters) de délégations selon leur profil d'offre sanitaire, et détecter des configurations atypiques.

**Type d'apprentissage :** Non-supervisé (Clustering + Réduction de dimensionnalité)

---

## 📊 Dataset

| Attribut | Valeur |
|---|---|
| Source | Ministère de la Santé – Maroc 2024 |
| Fichier | `etablissements-de-soins-de-sante-primaire-2024.xlsx` |
| Lignes | 3 224 établissements |
| Colonnes | 7 (Région, Délégation, Commune, Nom, Catégorie, Abréviation, Description) |

### Variables

| Colonne | Type | Description |
|---|---|---|
| `Region` | Catégoriel | 12 régions administratives du Maroc |
| `Delegation` | Catégoriel | 82 délégations sanitaires |
| `Commune` | Catégoriel | Commune de l'établissement |
| `Nom_Etablissement` | Texte | Nom de l'établissement |
| `Categorie` | Catégoriel | Type d'établissement (8 types) |
| `Abreviation` | Catégoriel | Code abrégé du type |
| `Description` | Texte | Libellé complet du type |

### Types d'Établissements

| Code | Description | Nombre |
|---|---|---|
| CSR-1 | Centre de Santé Rural niveau 1 | 893 |
| DR | Dispensaire Rural | 859 |
| CSU-1 | Centre de Santé Urbain niveau 1 | 692 |
| CSR-2 | Centre de Santé Rural niveau 2 | 431 |
| CSU-2 | Centre de Santé Urbain niveau 2 | 190 |
| CDTMR | Centre de Diagnostic et Traitement des Maladies Respiratoires | 71 |
| CRSR | Centre de Référence en Santé de Reproduction | 53 |
| LSP | Laboratoire de Santé Publique | 35 |

---

## 📁 Structure du Repo

```
├── ml_sante_primaire.ipynb                              # Notebook principal
├── etablissements-de-soins-de-sante-primaire-2024.xlsx  # Dataset
├── resultats_clustering.csv                             # Résultats exportés
├── README.md                                            # Ce fichier (rapport)
└── figures/
    ├── fig1_distribution.png
    ├── fig2_heatmap_profil.png
    ├── fig3_pca_variance.png
    ├── fig4_pca_2d.png
    ├── fig5_kmeans_elbow.png
    ├── fig6_kmeans_clusters.png
    ├── fig7_kmeans_profils.png
    ├── fig8_dendrogramme.png
    ├── fig9_hierarchique_clusters.png
    ├── fig10_dbscan_kdist.png
    ├── fig11_dbscan_clusters.png
    ├── fig12_comparaison.png
    └── fig13_clusters_par_region.png
```

---

## ⚙️ Prérequis & Installation

### Cloner le repo

```bash
git clone https://github.com/<votre-username>/<nom-du-repo>.git
cd <nom-du-repo>
```

### Installer les dépendances

```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy openpyxl
```

### Lancer le notebook

```bash
jupyter notebook ml_sante_primaire.ipynb
```

---

## 🔄 Pipeline ML

```
Données brutes (Excel)
        │
        ▼
  Chargement & EDA
  (exploration, visualisations)
        │
        ▼
  Prétraitement
  ┌─────────────────────────────────────────┐
  │  Pivot : délégation × catégorie         │
  │  → Profil numérique par délégation      │
  │  StandardScaler → normalisation         │
  └─────────────────────────────────────────┘
        │
        ▼
  Réduction de dimensionnalité
  ┌─────────────┐
  │     PCA     │  → Scree plot, variance cumulée, projection 2D
  └─────────────┘
        │
        ▼
  Clustering
  ┌────────────┐  ┌──────────────────┐  ┌────────────┐
  │  K-Means   │  │  Hiérarchique    │  │   DBSCAN   │
  │ (Elbow+Sil)│  │  (Ward + Dendro) │  │ (k-dist)   │
  └────────────┘  └──────────────────┘  └────────────┘
        │
        ▼
  Évaluation & Comparaison
  (Silhouette, Calinski-Harabasz, Davies-Bouldin)
        │
        ▼
  Interprétation & Export CSV
```

---

## 🤖 Algorithmes Appliqués

### 1. PCA – Analyse en Composantes Principales

**Principe :** Projette les données dans un espace de plus faible dimension en maximisant la variance expliquée.

**Application :**
- Réduction depuis 8 features vers N composantes (couvrant ≥ 80% de variance)
- Visualisation 2D des délégations
- Préparation des features pour les algorithmes de clustering

**Hyperparamètres :** `n_components` choisi via le scree plot et la courbe de variance cumulée.

---

### 2. K-Means

**Principe :** Partitionne les données en K clusters en minimisant la somme des distances au centroïde.

**Application :**
- K optimal déterminé via la **méthode du coude** (Elbow) et le **score de silhouette**
- Initialisation `k-means++` pour de meilleures convergences
- 10 initialisations aléatoires pour robustesse

**Hyperparamètres :**
```python
KMeans(n_clusters=K, init='k-means++', n_init=10, random_state=42)
```

---

### 3. Clustering Hiérarchique Agglomératif

**Principe :** Fusionne itérativement les points/groupes les plus proches pour construire un arbre (dendrogramme). Ne nécessite pas de K prédéfini.

**Application :**
- Méthode de liaison : **Ward** (minimise la variance intra-cluster)
- Dendrogramme complet + coupe à K clusters pour comparaison avec K-Means

**Hyperparamètres :**
```python
AgglomerativeClustering(n_clusters=K, linkage='ward')
```

---

### 4. DBSCAN

**Principe :** Clustering basé sur la densité. Identifie les clusters de forme arbitraire et détecte les **points bruit (anomalies)** (label = -1).

**Application :**
- `eps` estimé via la **courbe des k-distances** (coude = seuil de densité)
- Identification des délégations à profil atypique

**Hyperparamètres :**
```python
DBSCAN(eps=eps_val, min_samples=3)
```

---

## 📈 Résultats & Évaluation

### Métriques d'évaluation interne

| Métrique | Description | Meilleur |
|---|---|---|
| **Silhouette** | Cohésion + séparation [-1, 1] | ↑ proche de 1 |
| **Calinski-Harabasz** | Ratio dispersion inter/intra | ↑ plus élevé |
| **Davies-Bouldin** | Similarité moyenne entre clusters | ↓ proche de 0 |

### Comparaison des algorithmes

| Algorithme | Nb. Clusters | Silhouette ↑ | Calinski-Harabasz ↑ | Davies-Bouldin ↓ |
|---|---|---|---|---|
| **K-Means** | K optimal | *voir notebook* | *voir notebook* | *voir notebook* |
| **Hiérarchique (Ward)** | K optimal | *voir notebook* | *voir notebook* | *voir notebook* |
| **DBSCAN** | Auto-détecté | *voir notebook* | *voir notebook* | *voir notebook* |

> 💡 Les valeurs exactes sont affichées dans la **Section 8** du notebook après exécution.

---

## 🖼️ Visualisations

Le notebook génère 13 figures :

| Figure | Description |
|---|---|
| `fig1_distribution` | Distribution des catégories et des régions |
| `fig2_heatmap_profil` | Heatmap du profil des délégations |
| `fig3_pca_variance` | Scree plot + variance cumulée |
| `fig4_pca_2d` | Projection PCA 2D colorée par région |
| `fig5_kmeans_elbow` | Méthode du coude + score silhouette |
| `fig6_kmeans_clusters` | Clusters K-Means sur PCA 2D |
| `fig7_kmeans_profils` | Heatmap des profils moyens des clusters |
| `fig8_dendrogramme` | Dendrogramme hiérarchique |
| `fig9_hierarchique_clusters` | Clusters hiérarchiques sur PCA 2D |
| `fig10_dbscan_kdist` | Courbe k-distances pour le choix d'eps |
| `fig11_dbscan_clusters` | Clusters DBSCAN + bruit sur PCA 2D |
| `fig12_comparaison` | Comparaison des métriques |
| `fig13_clusters_par_region` | Distribution des clusters par région |

---

## 🔍 Interprétation

Les clusters identifiés correspondent à des **profils d'offre sanitaire** distincts :

- **Profil rural** : délégations à forte proportion de CSR-1 et DR — zones essentiellement rurales avec peu de structures spécialisées
- **Profil urbain** : délégations à dominante CSU-1 / CSU-2 — grandes villes avec offre plus diversifiée
- **Profil intermédiaire** : mixte rural/urbain, présence de structures spécialisées (CDTMR, CRSR)
- **Anomalies (DBSCAN)** : délégations avec un profil atypique pouvant indiquer des inégalités ou des spécificités locales

---

## ✅ Conclusion

| Point | Résumé |
|---|---|
| **Problème** | Pas de variable cible → apprentissage non-supervisé |
| **Stratégie** | Pivot délégation × catégorie → profil numérique |
| **Meilleur algo** | À déterminer selon les métriques (voir notebook) |
| **Apport** | Identification de 3-4 profils sanitaires distincts au Maroc |
| **Limite** | Absence de données démographiques (population couverte) |

---

## 👤 Auteur

> Projet réalisé dans le cadre du cours de **Machine Learning**  
> Filière : *[votre filière]*  
> Année universitaire : 2024–2025
