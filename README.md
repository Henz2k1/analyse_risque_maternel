# Analyse exploratoire du risque maternel — Groupe 7

Projet d'évaluation (Semaine 16) · Akieni Academy · Octobre 2026

Analyse exploratoire des facteurs associés aux différents niveaux de risque maternel chez les femmes enceintes, à partir du jeu de données Kaggle « Fighting Maternal Risk ». Le projet s'arrête à l'analyse et à la visualisation (pas de machine learning).

---

## Sommaire

1. [Présentation](#1-présentation)
2. [Données](#2-données)
3. [Structure du projet](#3-structure-du-projet)
4. [Installation et exécution](#4-installation-et-exécution)
5. [Contenu du notebook](#5-contenu-du-notebook)
6. [Nettoyage des données](#6-nettoyage-des-données)
7. [Analyses réalisées](#7-analyses-réalisées)
8. [Visualisations produites](#8-visualisations-produites)
9. [Principaux résultats](#9-principaux-résultats)
10. [Limites](#10-limites)
11. [Pistes d'amélioration](#11-pistes-damélioration)
12. [Dépannage](#12-dépannage)
13. [Équipe](#13-équipe)

---

## 1. Présentation

### Contexte

Dans le cadre de notre formation en Data Science à Akieni Academy, nous étudions les mesures de santé de femmes enceintes (âge, tension artérielle, glycémie, température, fréquence cardiaque) et leur niveau de risque maternel.

### Problématique

> Dans quelle mesure les caractéristiques physiologiques et sociodémographiques des femmes enceintes sont-elles associées aux différents niveaux de risque maternel observés dans les données ?

### Objectifs

- Contrôler la qualité des données et les nettoyer de façon documentée.
- Décrire la répartition des niveaux de risque et la distribution de chaque mesure.
- Comparer les mesures selon le niveau de risque.
- Mesurer les corrélations entre les mesures et le niveau de risque.
- Repérer les seuils de glycémie et de tension associés à un risque élevé.
- Vérifier que les jeux `train` et `test` sont comparables.

### Outils

Python · Jupyter Notebook · Pandas · Matplotlib · Seaborn (carte de chaleur)

---

## 2. Données

### Source

- Plateforme : Kaggle, compétition « Fighting Maternal Risk » (`mlolympiadbd2025`)
- Lien : <https://www.kaggle.com/competitions/mlolympiadbd2025/data>
- Origine : mesures de santé collectées au Bangladesh, dans des établissements de santé (hôpitaux, centres de santé communautaires, structures de soins maternels), notamment en zones rurales (Ahmed et al., 2020).
- Les données restent soumises aux conditions d'utilisation de la compétition Kaggle.

### Fichiers

| Fichier | Variable pandas | Taille brute | Rôle |
|---|---|---|---|
| `train.csv` | `df_train` | 811 × 9 | **Fichier principal** : mesures et niveau de risque (`RiskLevel`) |
| `test.csv` | `df_test` | 203 × 8 | Mêmes variables, sans niveau de risque ; sert à comparer `train` et `test` |
| `sample_submission.csv` | `df_sample` | 203 × 2 | Format de réponse attendu ; contrôle uniquement |
| `metadata.csv` | `df_metadata` | 8 × 2 | Dictionnaire des variables |

### Variables

| Variable (brute → notebook) | Type | Description |
|---|---|---|
| `Age` | Quantitative | Âge (années) |
| `SystolicBP` | Quantitative | Pression artérielle systolique (mmHg) |
| `DiastolicBP` | Quantitative | Pression artérielle diastolique (mmHg) |
| `Blood glucose` → `BloodGlucose` | Quantitative | Glycémie (mmol/L) |
| `BodyTemp` → `BodyTemp_C` | Quantitative | Température ; valeurs d'origine en °F, converties en °C |
| `HeartRate` | Quantitative | Fréquence cardiaque (battements par minute) |
| `RiskLevel` → `RiskLabel` | Qualitative ordinale | Niveau de risque codé 0, 1, 2 ; libellés Low Risk, Mid Risk, High Risk |
| `Id` | Identifiant | Exclu des analyses |
| `Usage` | Constante | Supprimée au nettoyage |

**Hypothèse de codage :** la correspondance 0 = Low Risk, 1 = Mid Risk, 2 = High Risk n'est pas précisée dans `metadata.csv`. Elle est posée comme hypothèse, cohérente avec les données (la glycémie et la tension moyennes augmentent avec le code), et reste à confirmer sur la page Kaggle du jeu.

### Variables créées dans le notebook

| Variable | Description |
|---|---|
| `BodyTemp_C` | Température en °C : (°F − 32) × 5/9, arrondie à une décimale |
| `RiskLabel` | Libellé du niveau de risque |
| `GlycemieClasse` | Glycémie en quatre classes : 7 ou moins, 7 à 9, 9 à 12, plus de 12 |
| `TensionElevee` | Pression systolique supérieure ou égale à 140 mmHg |

---

## 3. Structure du projet

```
projet/
├── README.md
├── requirements.txt
├── analyse_risque_maternel.ipynb
├── donnees/
│   ├── brutes/
│   │   ├── train.csv
│   │   ├── test.csv
│   │   ├── sample_submission.csv
│   │   └── metadata.csv
│   └── traitees/
│       ├── train_clean.csv
│       └── test_clean.csv
├── figures/
│   └── (8 fichiers .png)
└── livrables/
    ├── cahier_de_charges_arm.pdf
    └── presentation_groupe_7.pptx
```

Les dossiers `donnees/traitees/` et `figures/` sont remplis par le notebook. Les fichiers bruts ne sont jamais modifiés.

---

## 4. Installation et exécution

### Prérequis

- Python 3.10 ou plus récent
- Jupyter Notebook ou JupyterLab

### Installation

```bash
# 1. Récupérer le projet
cd projet

# 2. (Recommandé) créer un environnement virtuel
python -m venv venv
source venv/bin/activate        # Windows : venv\Scripts\activate

# 3. Installer les bibliothèques
pip install -r requirements.txt
```

Contenu de `requirements.txt` :

```
pandas
matplotlib
seaborn
jupyter
```

### Préparation des données

Télécharger les quatre fichiers depuis Kaggle (lien de la section 2) et les placer dans `donnees/brutes/` : `train.csv`, `test.csv`, `sample_submission.csv`, `metadata.csv`.

### Exécution

```bash
jupyter notebook 01_EXPLORATION_NETTOYAGE_corrige.ipynb
```

Dans Jupyter, utiliser **Kernel → Restart & Run All** pour exécuter toutes les cellules de haut en bas. Le notebook doit être lancé depuis la racine du projet, car les chemins sont relatifs (`./donnees/brutes/`, `./donnees/traitees/`, `./figures/`).

---

## 5. Contenu du notebook

| Section | Contenu |
|---|---|
| 1. Introduction | Contexte, fichiers, jeu de données, fichier principal, contrainte et outils, import des bibliothèques et chargement |
| 2. Exploration | Dictionnaire, dimensions, premières lignes, valeurs manquantes, variable cible, identifiants, doublons et valeurs suspectes |
| 3. Nettoyage | Copies, suppression de `Usage`, renommage, valeurs aberrantes, conversion °F → °C, libellés du risque, doublons, vérification finale, sauvegarde des CSV |
| 4. Analyse | Répartition du risque, moyennes, médianes, corrélations, glycémie en classes, tension élevée |
| 5. Visualisation | Huit graphiques enregistrés dans `figures/` |
| 6. Conclusion | Résumé, principaux résultats, limites, recommandations |

---

## 6. Nettoyage des données

| Sujet | Décision |
|---|---|
| Valeurs manquantes | Aucune ; rien à faire |
| Fréquence cardiaque aberrante | Suppression des 2 lignes à 7 bpm (seuil exploratoire : moins de 40 bpm) ; `train` passe de 811 à 809 lignes |
| Colonne `Usage` | Une seule valeur par fichier ; supprimée |
| Colonne `Blood glucose` | Renommée `BloodGlucose` |
| Température | Valeurs en °F ; ajout de `BodyTemp_C` en °C, colonne d'origine conservée |
| Niveau de risque | Ajout de `RiskLabel` (hypothèse de codage 0/1/2) |
| Doublons (hors `Id`) | 396 lignes identiques sur 809 ; conservées, car les `Id` diffèrent (patientes distinctes aux mesures courantes) |
| Sauvegarde | `train_clean.csv` et `test_clean.csv` dans `donnees/traitees/` |

---

## 7. Analyses réalisées

| Question | Méthode | Graphique |
|---|---|---|
| Comment les niveaux de risque sont-ils répartis ? | `value_counts(normalize=True)` | 1 |
| Quel est le profil global des variables ? | `describe()` | 2 |
| Comment les mesures varient-elles selon le risque ? | `groupby("RiskLabel")` : moyenne et médiane | 3 |
| Comment le risque varie-t-il selon la glycémie ? | `pd.cut` et `pd.crosstab` | 4 |
| Quelles mesures sont liées au risque et entre elles ? | `corr()` | 5 |
| Glycémie et tension forment-elles des profils distincts ? | Nuage de points | 6 |
| Âge et glycémie forment-ils des profils distincts ? | Nuage de points | 7 |
| `train` et `test` sont-ils comparables ? | Histogrammes superposés (âge, glycémie) | 8 |
| Comment le risque varie-t-il selon la tension systolique ? | `pd.crosstab` (seuil 140 mmHg) | Tableau (section 4.6) |

---

## 8. Visualisations produites

Palette : Low Risk = vert (`#4CAF50`), Mid Risk = orange (`#FFB300`), High Risk = rouge (`#E53935`). Toutes les figures sont enregistrées à 600 dpi.

| N° | Graphique | Fichier |
|---|---|---|
| 1 | Répartition des niveaux de risque | `figures/repartition_niveaux_risque.png` |
| 2 | Distribution de chaque mesure | `figures/distribution_mesures.png` |
| 3 | Chaque mesure selon le niveau de risque | `figures/mesure_selon_niveau_de_risque.png` |
| 4 | Le risque selon la classe de glycémie | `figures/niveau_risque_selon_glycemie.png` |
| 5 | Carte des corrélations | `figures/matrice_correlation_variables.png` |
| 6 | Glycémie et tension systolique | `figures/glycemie_tension_systolique_risque.png` |
| 7 | Âge et glycémie | `figures/scatter_age_glycemie.png` |
| 8 | Comparaison des distributions `train` / `test` | `figures/comparaison_distributions_test_vs_train.png` |

---

## 9. Principaux résultats

Résultats obtenus sur `df_train_clean` (809 lignes). Ils décrivent des associations observées dans les données, pas des relations de cause à effet.

### Répartition des niveaux de risque

| Niveau | Libellé | Proportion |
|:---:|---|:---:|
| 0 | Low Risk | 39,9 % |
| 1 | Mid Risk | 33,3 % |
| 2 | High Risk | 26,8 % |

### Corrélations avec le niveau de risque

| Variable | Corrélation |
|---|:---:|
| `BloodGlucose` | 0,56 |
| `SystolicBP` | 0,40 |
| `DiastolicBP` | 0,34 |
| `Age` | 0,27 |
| `BodyTemp_C` | 0,17 |
| `HeartRate` | 0,17 |

La corrélation entre pression systolique et diastolique est de 0,79.

### Moyennes selon le niveau de risque

| Mesure | Low Risk | Mid Risk | High Risk |
|---|:---:|:---:|:---:|
| Âge (années) | 27 | 28 | 37 |
| Pression systolique (mmHg) | 106 | 113 | 124 |
| Pression diastolique (mmHg) | 72 | 74 | 85 |
| Glycémie (mmol/L) | 7,2 | 7,7 | 12,0 |

### Glycémie et risque

| Glycémie (mmol/L) | Low Risk | Mid Risk | High Risk |
|---|:---:|:---:|:---:|
| 7 ou moins | 41,7 % | 50,5 % | 7,8 % |
| 7 à 9 | 56,2 % | 25,5 % | 18,2 % |
| 9 à 12 | 7,3 % | 21,8 % | 70,9 % |
| plus de 12 | 0 % | 10,6 % | 89,4 % |

Le risque ne croît pas régulièrement avec la glycémie, mais le seuil de 9 mmol/L apparaît comme un point de bascule : au-delà, la majorité des femmes sont classées à risque élevé.

### Tension systolique et risque

| Tension systolique | Low Risk | Mid Risk | High Risk |
|---|:---:|:---:|:---:|
| moins de 140 mmHg | 45,8 % | 37,4 % | 16,7 % |
| 140 mmHg ou plus | 0 % | 4,8 % | 95,2 % |

### Lecture d'ensemble

- La glycémie est la mesure la plus liée au niveau de risque, suivie de la pression artérielle, puis de l'âge.
- La température et la fréquence cardiaque apportent peu d'information.
- Les médianes sont proches entre Low Risk et Mid Risk ; la rupture se fait au niveau High Risk.
- La glycémie sépare les groupes quel que soit l'âge.
- Les jeux `train` et `test` paraissent comparables sur l'âge et la glycémie.

---

## 10. Limites

1. **Nature observationnelle** : seules des associations peuvent être décrites, pas des relations de cause à effet.
2. **Biais potentiel** : les données proviennent de zones rurales du Bangladesh, ce qui limite la généralisation.
3. **Variables manquantes** : d'autres facteurs (parité, antécédents, IMC) pourraient être pertinents.
4. **Données transversales** : pas de suivi longitudinal des patientes.
5. **Classification du risque** : les critères exacts ne sont pas documentés, et le codage 0/1/2 reste une hypothèse.
6. **Corrélation de Pearson** : le niveau de risque est ordonné (0/1/2), donc la corrélation n'est qu'une indication approximative.
7. **Pas de tests statistiques** : les associations sont décrites, non testées.
8. **Valeurs répétitives** : la température vaut 98 °F pour environ 80 % des lignes, ce qui limite son pouvoir d'analyse.
9. **Âges extrêmes** : l'âge varie de 10 à 70 ans ; ces valeurs n'ont pas été vérifiées.

---

## 11. Pistes d'amélioration

1. Développer un modèle de machine learning pour prédire le niveau de risque.
2. Intégrer des variables supplémentaires (parité, IMC, antécédents médicaux).
3. Réaliser un suivi longitudinal des grossesses.
4. Valider les résultats sur d'autres populations.
5. Faire valider les seuils identifiés par des experts médicaux.

---

## 12. Dépannage

| Problème | Cause probable | Solution |
|---|---|---|
| `FileNotFoundError` au chargement | Fichiers absents de `donnees/brutes/` ou notebook lancé hors de la racine du projet | Placer les 4 fichiers CSV dans `donnees/brutes/` et lancer Jupyter depuis la racine |
| `ModuleNotFoundError` | Bibliothèque non installée | `pip install -r requirements.txt` |
| Boîtes à moustaches vides | `RiskLabel` rempli de `NaN` (libellés non cohérents avec ceux de `ordre`) | Exécuter la cellule 3.4 avant le graphique et vérifier avec `df_train_clean["RiskLabel"].value_counts(dropna=False)` |
| `NameError` sur une variable | Cellules exécutées dans le désordre | Kernel → Restart & Run All |
| Dossier `figures/` introuvable | Dossier non créé | Exécuter les cellules du notebook de haut en bas ; il est créé par le code |

---

## 13. Équipe

**Groupe 7 · Akieni Academy**

| Membre | Rôle principal |
|---|---|
| Theresia Surya NGOUBALI | Coordination et collecte |
| Pejuce Pedrich NDINGA | Ingestion et qualité des données |
| Minervi TULIKUMWE MINDAMUTSA | Analyse descriptive et visualisation |
| Eecha Henri MASUKU | Intégration et reproductibilité |
