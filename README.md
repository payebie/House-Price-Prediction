# 🏠 House Price Prediction — Snowflake ML Pipeline

> **Workshop : Data Engineering et Machine Learning avec Snowflake**  
> **Étudiant :** AYEBIE Pla Dorgelès Armand
              **Monica Branly MBOULA
> **Promotion :** MBDIA — Classe 2  
> **Livrable :** `MBAESG_MBDIA_CLASSE2_EVALUATION_DATAENGINEER_MLOPS`

---

## 📌 Contexte

Ce projet implémente un pipeline ML complet **entièrement dans Snowflake** pour prédire le prix de vente de maisons à partir de leurs caractéristiques physiques et équipements.

Les données sont chargées depuis un bucket S3 (`s3://logbrain-datalake/datasets/house_price/`) via un **Stage externe Snowflake**, transformées en Python avec **Snowpark**, entraînées avec **scikit-learn / XGBoost**, stockées dans le **Snowflake Model Registry**, et exposées via une application **Streamlit**.

---

## 📂 Structure du dépôt

```
.
├── house_price_ml_snowflake.ipynb   # Notebook Snowflake — pipeline ML complet
└── README.md                        # Ce fichier
```

---

## 🗄️ Dataset

| Colonne | Type | Description |
|---|---|---|
| `price` | Float | Prix de vente — variable cible |
| `area` | Float | Surface totale (m²) |
| `bedrooms` | Int | Nombre de chambres |
| `bathrooms` | Int | Nombre de salles de bain |
| `stories` | Int | Nombre d'étages |
| `mainroad` | yes/no | Accès route principale |
| `guestroom` | yes/no | Chambre d'amis |
| `basement` | yes/no | Sous-sol |
| `hotwaterheating` | yes/no | Chauffage eau chaude |
| `airconditioning` | yes/no | Climatisation |
| `parking` | Int | Places de stationnement |
| `prefarea` | yes/no | Zone privilégiée |
| `furnishingstatus` | catégorie | Meublée / Semi-meublée / Non meublée |

---

## ⚙️ Pipeline ML — Étapes

| Étape | Description | Technologie |
|---|---|---|
| 1 | Configuration & session Snowflake | Python |
| 2 | Ingestion depuis S3 via Stage + COPY INTO | SQL |
| 3 | Exploration des données (EDA, stats, corrélations, visualisations) | Python |
| 4 | Feature engineering — encodage, normalisation, split 80/20 | Python |
| 5 | Entraînement de 4 modèles | Python / scikit-learn / XGBoost |
| 6 | Évaluation comparative MAE / RMSE / R² | Python |
| 7 | Optimisation GridSearchCV + RandomizedSearchCV | Python |
| 8 | Stockage dans le Snowflake Model Registry | Python + SQL |
| 9 | Inférence + sauvegarde des prédictions | Python + SQL |
| 10 | Application Streamlit interactive | Python / Streamlit |

---

## 📊 Analyse des Performances des Modèles

### Modèles de base (sans optimisation)

| Modèle | MAE | RMSE | R² |
|---|---|---|---|
| Linear Regression | 40 253 | 53 985 | 0.6732 |
| Ridge Regression | 40 253 | 53 988 | 0.6731 |
| Random Forest | 19 500 | 32 483 | 0.8817 |
| **XGBoost** | **12 757** | **27 517** | **0.9151** |

Les modèles linéaires atteignent un R² de 0.67, ce qui indique que la relation entre les features et le prix n'est pas purement linéaire. Random Forest et XGBoost capturent bien mieux les interactions non-linéaires entre les variables.

---

### Après optimisation des hyperparamètres

| Modèle | Méthode | Paramètres optimaux | R² (CV) | MAE (test) | RMSE (test) | R² (test) |
|---|---|---|---|---|---|---|
| XGBoost | GridSearchCV (8 configs × 3 folds) | `learning_rate=0.1, max_depth=5, n_estimators=200` | 0.8286 | 18 054 | 29 531 | 0.9022 |
| **Random Forest** | **RandomizedSearchCV (10 iter × 3 folds)** | **`max_depth=None, max_features=log2, min_samples_split=2, n_estimators=98`** | **0.8365** | **18 422** | **28 103** | **0.9114** |

---

### 🏆 Meilleur modèle retenu

**Random Forest (RandomizedSearchCV)** — enregistré dans le Snowflake Model Registry sous `HOUSE_PRICE_MODEL / v1`

| Métrique | Valeur |
|---|---|
| MAE | 18 422 |
| RMSE | 28 103 |
| **R²** | **0.9114** |

Le Random Forest optimisé surpasse le XGBoost optimisé sur le jeu de test (R² 0.9114 vs 0.9022), bien que le XGBoost de base soit légèrement meilleur (0.9151). La grille de recherche XGBoost était volontairement réduite pour respecter les contraintes mémoire de l'environnement Snowflake (`n_jobs=1`).

---

### Variables les plus influentes

D'après l'importance des features du Random Forest :

1. **AREA** — Surface (variable la plus prédictive)
2. **BATHROOMS** — Nombre de salles de bain
3. **STORIES** — Nombre d'étages
4. **PARKING** — Nombre de places
5. **BEDROOMS** — Nombre de chambres

---

## 🚀 Instructions de déploiement

### Prérequis Snowflake
- Rôle avec accès `CREATE STAGE`, `CREATE TABLE`, `CREATE MODEL`
- Warehouse actif
- Accès réseau au bucket S3 `s3://logbrain-datalake/datasets/house_price/`

### Exécuter le Notebook
Importer `house_price_ml_snowflake.ipynb` dans **Snowflake > Notebooks** et exécuter les cellules dans l'ordre.

### Déployer l'application Streamlit
Copier le code affiché par la dernière cellule du notebook dans **Snowflake > Streamlit > + New Streamlit App**.

---

*Réalisé dans le cadre du Workshop Snowflake — MBA ESG MBDIA Classe 2*
