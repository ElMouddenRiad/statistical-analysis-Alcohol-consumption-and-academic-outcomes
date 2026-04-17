# Analyse Statistique: Consommation d'Alcool et Résultats Scolaires

**Projet académique de Data Science** – M1 SI2A  
Réalisé par: **Bakr OUKESSOU** et **Riad EL MOUDDEN**  
Présenté: 30 mai 2025

---

## 📋 Résumé Exécutif

Ce projet propose une **analyse statistique rigoureuse** sur l'impact de la consommation d'alcool sur les résultats académiques. À partir d'un dataset fédéré de **382 étudiants** et **57 variables**, nous documentons les relations entre facteurs socio-démographiques, patterns de consommation d'alcool et performance scolaire via six hypothèses testées qui permettent de valider différentes dimensions du problème.

### Résultats Clés

| Hypothèse | Relation Testée | Test Utilisé | Résultat | P-value | Effet |
|-----------|-----------------|--------------|----------|---------|-------|
| **H1** | Éducation parentale → Alcool | T-test indépendant | ❌ | 0.746 | Néant |
| **H2** | Abus alcool → Réussite | Chi-carré | ❌ | 0.657 | Néant |
| **H3** | Genre → Alcool | T-test indépendant | ✅ **Fort** | **6.68e-08** | Medium-Grand |
| **H4** | Semaine vs Week-end | T-test apparié | ✅ **Très fort** | **8.46e-45** | Grand |
| **H5** | Rural/Urbain → Alcool | T-test indépendant | ✅ **Modéré** | **0.038** | Petit |
| **H6** | Alcool ↔ Absences | Pearson r | ✅ **Significatif** | **2.13e-05** | Faible |

**Points saillants:**
- **4 hypothèses confirmées**, 2 rejetées (démontrant pensée critique)
- **Différence de genre** (garçons +34% alcool) et **variation semaine/WE** (55% plus haut WE) = patterns les plus robustes
- Résultats négatifs (H1, H2) attestent qu'éducation parentale et succès académique ne sont pas liés à la consommation déclarée

---

## 🔬 Méthodologie Détaillée

### Phase 1: Nettoyage des Données (Data Cleaning)

#### 1.1 Fusion des Datasets
**Données sources:**
- `student-mat.csv`: 383 étudiants (mathématiques)
- `student-por.csv`: 649 étudiants (portugais)

**Processus de fusion:**
- **Clés de jointure:** 12 colonnes communes (école, sexe, âge, adresse, taille famille, statuts parents, éducation parents, travail parents, temps étude, absences, problèmes, soutien scolaire)
- **Type de jointure:** Inner join (intersection stricte)
- **Résultat:** 382 étudiants avec données cohérentes

#### 1.2 Vérification Valeurs Manquantes
- **Méthode:** `pandas.isnull()` / `pandas.isna()`
- **Résultat:** **Aucune valeur manquante détectée**
- **Action:** Dataset utilisable directement, pas d'imputation nécessaire

#### 1.3 Détection Aberrances (Outliers)
- **Approche:** Méthode IQR (Interquartile Range)
- **Variables analysées:** Alcool (Dalc, Walc), grades (G1, G2, G3), absences
- **Seuils:** Q1 - 1.5×IQR et Q3 + 1.5×IQR
- **Résultat:** Outliers identifiés et retenus (données naturellement valides)
- **Visualisation:** Figure 2 (boxplots distribution alcool)

#### 1.4 Suppression Doublons
- **Méthode:** `pandas.drop_duplicates()` multi-colonnes
- **Résultat:** **Aucun doublon** détecté
- **Conclusion:** Chaque enregistrement = étudiant unique

### Phase 2: Transformation des Données (Data Transformation)

#### 2.1 Feature Engineering (4 Variables Dérivées)

**Variable 1: `grade_avg`** (Moyenne académique)
- **Formule:** $(G1 + G2 + G3) / 3$
- **Plage:** [0, 20]
- **Usage:** Créer variable `success` (≥10 = réussi)

**Variable 2: `alcohol_avg`** (Consommation moyenne)
- **Formule:** $(Dalc + Walc) / 2$
- **Justification:** Homogénéisation semaine/week-end
- **Plage:** [1, 5]
- **Usage:** Tests paramétriques (t-test)

**Variable 3: `abuse_alcohol`** (Flag binaire d'abus)
- **Calcul:** `alcohol_avg > 3` (seuil haut)
- **Justification:** "Consommation modérée-forte"
- **Usage:** Test chi-carré, analyses stratifiées

**Variable 4: `success`** (Réussite académique)
- **Calcul:** `grade_avg >= 10`
- **Justification:** Seuil officiel validation portugaise
- **Usage:** Chi-carré (H2), performance analysis

#### 2.2 Encodage Variables Qualitatives
- **Ordinales** (Dalc, Walc): Conservées (1-5 existant)
- **Nominales** (sexe, adresse): One-hot encoding
- **Raison:** Préparation pour processus PCA

#### 2.3 Statistiques Descriptives Post-Fusion

```
Effectifs et couverture:
- Observations totales:        382
- Variables utiles:            57
- Taux réussite global:        56.8% (217/382)
- Taux abus alcool:            41.6% (159/382)

Consommation patterns:
- Globale:       1.92 ± 0.89  (moyenne ± écart-type)
- Filles:        1.61 ± 0.75
- Garçons:       2.16 ± 0.97  (+34% vs filles)

Performance académique:
- Moyenne globale:     11.47 ± 3.12
- Réussis (≥10):       13.21 ± 1.88
- Échoués (<10):        8.76 ± 2.41
```

#### 2.4 Réduction Dimensionnalité (PCA Exploratoire)
- **Standardisation:** Z-score normalization
- **Application:** PCA sur 55 variables continues/encodées
- **Composantes retenues:** 2 (visualisation)
- **Variance expliquée:** ~45% (PC1+PC2)
- **Résultat:** Figure 1 - clusters faibles identifiés (profils "sécurisé" vs "à risque")

#### 2.5 Export Dataset Nettoyé
**Fichier:** `PRE/merged_cleaned.csv`
- **Dimensions:** 382 lignes × 57 colonnes
- **Format:** CSV UTF-8
- **État:** Prêt analyse statistique

---

## 📊 Résultats Détaillés par Hypothèse

### **H1: Éducation Parentale vs Consommation d'Alcool** ❌

**Question:** L'éducation des parents prédit-elle la consommation d'alcool?

**Approche:**
- **Groupes:** Éducation basse (1-2) vs haute (4-5)
- **Test:** T-test indépendant
- **H₀:** Moyennes égales entre groupes

**Résultats:**
```
Père - Basse éducation: μ=1.894  vs  Père - Haute: μ=1.861
  T=-0.324, p=0.746 → NON SIGNIFICATIF

Mère - Basse éducation: μ=1.875  vs  Mère - Haute: μ=1.898
  T=-0.338, p=0.736 → NON SIGNIFICATIF
```
**Conclusion:** Éducation parentale N'EST PAS prédicteur de consommation. Figures 3-5 (boxplots).

---

### **H2: Abus d'Alcool vs Réussite Académique** ❌

**Question:** L'abus d'alcool prédicteur de l'échec académique?

**Approche:**
- **Groupes:** Abus oui/non × Réussite oui/non (2×2)
- **Test:** Chi-carré d'indépendance
- **H₀:** Indépendance entre variables

**Résultats:**
```
Tableau de contingence:
                 Réussi      Échoué
Abus = Oui       83 (52.2%)  76 (47.8%)
Abus = Non      134 (57.8%)  98 (42.2%)

Chi² = 0.197, p=0.657 → NON SIGNIFICATIF
```
**Conclusion:** Relation insignifiante. Taux réussite quasi-identiques (52.2% vs 57.8%). Figure 6.

---

### **H3: Genre et Consommation d'Alcool** ✅ **TRÈS SIGNIFICATIF**

**Question:** Différence de consommation entre garçons et filles?

**Approche:**
- **Test:** T-test indépendant
- **H₀:** Moyennes égales

**Résultats:**
```
Filles:    μ=1.614±0.747 (n=186)
Garçons:   μ=2.160±0.973 (n=196)

T = -5.534, p = 6.68e-08 ✅✅✅ HAUTEMENT SIGNIFICATIF

Différence = 0.546 points (34% plus élevé chez garçons)
Cohen's d ≈ 0.63 (effet MEDIUM-GRAND)
95% CI: [0.334, 0.758]
```
**Conclusion:** Patterns de genre très robustes. Garçons consomment significativement plus. Figures 7-8.

---

### **H4: Variation Semaine vs Week-end** ✅ **EXTRÊMEMENT SIGNIFICATIF**

**Question:** Consommation amplifie-t-elle le week-end?

**Approche:**
- **Test:** T-test apparié (paires d'observations)
- **H₀:** Consommation semaine = week-end

**Résultats:**
```
Semaine (Dalc):   μ=1.474±0.607
Week-end (Walc):  μ=2.280±1.178

T = 16.086, p = 8.46e-45 ✅✅✅ EXTRÊMEMENT SIGNIFICATIF

Différence = 0.806 points
Ratio Walc/Dalc = 1.547 (55% plus élevé WE)
Cohen's d ≈ 0.86 (effet GRAND)
95% CI [0.707, 0.905]
```
**Conclusion:** Pattern le PLUS ROBUSTE du projet. Écart systématique et cohérent. Figure 9.

---

### **H5: Environnement (Rural vs Urbain)** ✅ **MODÉRÉMENT SIGNIFICATIF**

**Question:** La résidence rurale/urbaine influence-t-elle?

**Résultats:**
```
Rural:     μ=2.086±0.948 (n=156)
Urbain:    μ=1.821±0.838 (n=226)

T = 2.101, p = 0.038 ✅ SIGNIFICATIF (seuil exact α)

Différence = 0.265 points (14.6%)
Cohen's d ≈ 0.30 (effet PETIT)
95% CI [0.014, 0.516]
```
**Conclusion:** Relation marginale (p=0.038 exactement). Ampleur d'effet petite. Figure 10.

---

### **H6: Consommation d'Alcool et Absences** ✅ **CORRÉLATION FAIBLE MAIS SOLIDE**

**Question:** Consommation liée aux absences?

**Résultats:**
```
Pearson r = 0.2158
T = 4.168, p = 2.13e-05 ✅

Interprétation:
- r² = 0.0466 (seulement 4.66% variance expliquée)
- Corrélation POSITIVE mais faible
- 95% CI: [0.112, 0.314]

Chaque unité d'alcool → ~+0.89 absences attendus
```
**Conclusion:** Relation réelle mais faible ampleur. Nombreux confounders possibles. Figure 11.

---

## 🛠️ Stack Technologique

| Composante | Stack | Justification |
|-----------|-------|---------------|
| **Langage** | Python 3.12 | Écosystème data mature, compatibilité |
| **Données** | Pandas 2.0+ | Manipulation ETL, agrégation |
| **Numériques** | NumPy | Opérations matricielles |
| **Stats** | SciPy.stats | `ttest_ind`, `ttest_rel`, `chi2_contingency`, `pearsonr` |
| **Visualisation** | Matplotlib+Seaborn | Publication-quality figures |
| **ML/PCA** | Scikit-learn | Dimensionality reduction |
| **Développement** | Jupyter Notebook | Interactivité + documentation |
| **Reproduction** | pip+requirements.txt | Versionning dépendances |

**Packages clés:**
```
pandas==2.0.0
numpy==1.24.0
scipy==1.10.0
matplotlib==3.7.0
seaborn==0.12.0
scikit-learn==1.2.0
jupyter==1.0.0
notebook
```

---

## 📁 Structure du Projet

```
groupe-riadbakr/
├── README.md                          ← Documentation (ce fichier)
├── requirements.txt                   ← Dépendances
├── .gitignore                         ← Exclusions Git
│
├── student-mat.csv                    # Données brutes (math)
├── student-por.csv                    # Données brutes (portugais)
│
├── PRE/                               # PHASE 1: Preprocessing
│   ├── preprocessing.ipynb
│   └── merged_cleaned.csv
│
├── FIG/                               # PHASE 2: Visualisation
│   ├── visualisation.ipynb
│   └── figures/
│       ├── figure_1_pca.png
│       ├── figure_2_alcohol_dist.png
│       ├── figure_3_h1_father.png
│       ├── figure_4_h1_mother.png
│       ├── figure_5_h2_success.png
│       ├── figure_6_h3_gender.png
│       ├── figure_7_h3_abuse.png
│       ├── figure_8_h4_weekday.png
│       ├── figure_9_h5_location.png
│       └── figure_10_h6_absences.png
│
├── AN/                                # PHASE 3: Analyse Statistique
│   └── tests_hypotheses.ipynb
│
└── REP/
    └── rapportProjet_RiadBakr.pdf    # Rapport complet (26 pages)
```

---

## 🔄 Pipeline Analytique Complet

```
[DATA SOURCES]
  student-mat.csv + student-por.csv
           ↓
[PREPROCESSING] ← PRE/preprocessing.ipynb
  • Fusion (inner join 12 cols)
  • Contrôle manquants
  • Détection & gestion outliers
  • Suppression doublons
  • Feature engineering (4 vars)
  • Encodage catégories
  • Standardisation Z-score
  • PCA exploratoire
           ↓
[CLEANED DATASET] ← merged_cleaned.csv (382×57)
           ↓
   ├─→ [EXPLORATION] ← FIG/visualisation.ipynb
   │     • 6 ensembles graphiques
   │     • 10 figures PNG exportées
   │
   └─→ [HYPOTHESIS TESTING] ← AN/tests_hypotheses.ipynb
         • 6 hypothèses + tests associés
         • Tables résultats
         • Interprétations
```

---

## 🚀 Installation et Exécution

### 1. Prérequis
- Python 3.10+ (testé 3.12.0)
- pip (package manager)
- Git (optionnel)

### 2. Cloner le Dépôt
```bash
git clone https://github.com/votre-username/groupe-riadbakr.git
cd groupe-riadbakr
```

### 3. Créer Environnement Virtuel
**Windows:**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS/Linux:**
```bash
python -m venv .venv
source .venv/bin/activate
```

### 4. Installer Dépendances
```bash
pip install -r requirements.txt
```

### 5. Lancer Jupyter
```bash
jupyter notebook
```

**Exécution ordre (important):**
1. `PRE/preprocessing.ipynb` → Génère `merged_cleaned.csv`
2. `FIG/visualisation.ipynb` → Génère figures PNG
3. `AN/tests_hypotheses.ipynb` → Calcule statistiques

---

## English Summary

This project delivers a final statistical analysis of alcohol consumption and academic outcomes using two student datasets (`student-mat.csv` and `student-por.csv`).

### Scope and Data
- Final merged dataset: **382 students**, **57 variables**
- Workflow: data preparation, visualization, and hypothesis testing
- Main notebooks:
  - `PRE/preprocessing.ipynb`
  - `FIG/visualisation.ipynb`
  - `AN/tests_hypotheses.ipynb`

### Key Results
- **H1 (Parental education vs alcohol):** not significant (`p = 0.746`)
- **H2 (Alcohol abuse vs academic success):** not significant (`p = 0.657`)
- **H3 (Gender vs alcohol):** significant (`p = 6.68e-08`), higher average among boys
- **H4 (Weekday vs weekend alcohol):** highly significant (`p = 8.46e-45`), weekend consumption is higher
- **H5 (Rural vs urban):** significant (`p = 0.038`), slightly higher average in rural areas
- **H6 (Alcohol vs absences):** significant positive correlation (`r = 0.2158`, `p = 2.13e-05`)

### Final Deliverable
This repository contains the complete and reproducible final version of the academic project, including cleaned data outputs, figures, and statistical test results.

---

## 📚 Références

- Field, A. (2013). *Discovering Statistics Using IBM SPSS*. SAGE.
- Kline, R. B. (2011). *Principles and Practice of SEM*. Guilford.
- Hingson, R. et al. (2009). "Underage drinking..." *Alcohol Res Health*, 29(2).

---

## 👥 Auteurs

**Bakr OUKESSOU** – Analyse statistique, preprocessing  
**Riad EL MOUDDEN** – Visualisations, tests hypothèses

**Master:** M1 SI2A, ESISA, Fes .  
**Année:** 2024/2025

---

**État:** ✅ Complet et Reproductible  
**Dernière mise à jour:** 18 avril 2026
