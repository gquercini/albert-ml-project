# Y'a que la taille qui compte?

Lataillequi est un projet réalisé en 2e année de Bachelor Business & Data à AlbertSchool dans le cadre du cours de _Data: Deploying an ML project_ afin de prédire l'équipe de basketteurs idéale pour gagner un match.

### Présentation

```bash
https://www.canva.com/design/DAGnVLwCuhw/JlsmOdTvbZPZ0pkygkcseA/edit?utm_content=DAGnVLwCuhw&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton
```

### Datasets

Téléchargement des données: 
```bash
https://drive.google.com/drive/folders/1K4sXEjIcb7b2yzsQfOt80cdB0G69IDzj?usp=sharing
```
Le projet repose sur un ensemble de fichiers issus de l’API de la NBA ou de sources publiques NBA, portant sur plusieurs saisons. Les données sont structurées autour de trois axes :

- *Statistiques individuelles des joueurs* (all_seasons.csv) : points, passes, rebonds, efficacité au tir, rating d’impact, etc.
- *Résultats de match* (game.csv, line_score.csv) : scores finaux, identifiants d’équipes, issue du match.
- *Référentiels d'identité* (player.csv) : identifiants et noms complets des joueurs.

Ces données couvrent *plusieurs saisons NBA, permettent de relier les performances individuelles à des résultats collectifs, et sont utilisées pour analyser l’impact de compositions d’équipes. Ces fichiers contiennent des **statistiques individuelles et collectives, des **résultats de matchs, des **informations d’identité des joueurs et des équipes*, ainsi que des détails sur les compositions match par match.

# PARTIE 1

L’objectif du projet est de *prédire automatiquement la meilleure composition de 5 joueurs NBA ("dream team") par saison*, c’est-à-dire la combinaison qui maximise les chances de victoire de l'équipe.

### Datasets

- **all_seasons.csv** : statistiques par joueur et par saison (points, rebonds, passes, usage, efficacité au tir, etc.)
- **game.csv** : résultats des matchs avec les équipes concernées et les scores.
- **player.csv** : identifiants et noms des joueurs.
- **common_player.csv** : infos liens joueurs et teams
- **line_score.csv** : scores par équipe pour chaque match, utile pour valider les résultats.
- **team.csv** : infos sur les teams

## 📊 Feature Engineering

1. **Stats normalisées**  
   - `pts_per_game`, `reb_per_game`, `ast_per_game`  
   Moyennes par match pour neutraliser l’effet du nombre de rencontres jouées.

2. **Ratios avancés**  
   - `ast_usg_ratio` = `ast_pct` / `usg_pct`  
   - `reb_pct_sum`  = `oreb_pct` + `dreb_pct`  
   Évaluent l’efficacité collective (création de jeu, impact au rebond).

3. **Net Rating**  
   Bilan offensif – bilan défensif par 100 possessions, indicateur d’impact global.

4. **Poste primaire**  
   Extraction directe de la colonne `position` pour obtenir 3 classes :  
   - **G** (Guard)  
   - **F** (Forward)  
   - **C** (Center)  

---

## 🔍 Sélection des features

1. On calcule la **corrélation** de chaque métrique avec le `win_rate`.  
2. On **retient** les 5 variables les plus corrélées (absolu) pour entraîner le modèle.

---

## 🤖 Pipeline de modélisation

Pour chaque saison **X** :

1. **Entraînement**  
   - On construit un jeu d’entraînement sur toutes les saisons ≠ X.  
   - Chaque observation = une équipe + moyenne des 5 métriques pour ses 5 meilleurs scoreurs (by `pts`).  
   - On apprend un **RandomForestRegressor** à prédire le `win_rate` d’une lineup.

2. **Recherche de la Dream Team**  
   - On définit un **pool réduit** de candidats par poste (top `net_rating`) pour limiter les combinaisons.  
   - On génère **toutes les combinaisons** 2 Guards – 2 Forwards – 1 Center.  
   - On agrège leurs 5 métriques et on utilise le modèle pour estimer leur `win_rate`.  
   - On retient la composition dont la prédiction est la plus élevée.

# PARTIE 2

Le projet permettra aussi de *comparer deux lineups de joueurs* et de *prédire laquelle est la plus performante dans un match simulé*.

## Structure du code

Le système utilise la classe `LineupPredictor` qui permet de comparer deux lineups NBA aléatoires et de prédire laquelle gagnerait. Voici ses fonctionnalités principales:

```python
class LineupPredictor:
    def __init__(self)
    def load_data()
    def _clean_and_prepare_data()
    def _calculate_team_stats()
    def _train_model()
    def get_lineup_coherence(lineup)
    def select_random_lineup()
    def calculate_lineup_score(lineup)
    def predict_winner(lineup1, lineup2)
```

## Datasets
- `df_v1.csv`: Statistiques individuelles des joueurs par saison (dataset issu de la partie 1)
- `df_draft_combine_cleaned.csv`: Données physiques des joueurs (combine)
- `df_game_summary_cleaned.csv`: Résultats des matchs

Les données sont filtrées pour ne conserver que les saisons après 2000.

## Feature engineering

Le système calcule plusieurs métriques avancées pour chaque joueur:

1. **Statistiques par match**:
   - `pts_per_game`, `reb_per_game`, `ast_per_game`

2. **Métriques d'efficacité**:
   - `efficiency` = `(pts + reb + ast) / gp`
   - `scoring_efficiency` = `pts_per_game / usg_pct`
   - `playmaking` = `ast_per_game * ast_pct`
   - `ast_usg_ratio` = `ast_pct / usg_pct`
   - `reb_pct_sum` = `oreb_pct + dreb_pct`

3. **Classification des positions**: Simplification en 3 catégories:
   - **G**: Guard
   - **F**: Forward
   - **C**: Center

## Modèle de prédiction

Un modèle RandomForest est entraîné pour prédire le taux de victoire (`win_rate`) d'un lineup basé sur les statistiques agrégées des joueurs:

1. Sélection des 8 features les plus corrélées avec `win_rate`
2. Agrégation des statistiques des 5 meilleurs scoreurs de chaque équipe
3. Entraînement d'un `RandomForestRegressor` (200 arbres, profondeur max 10)

## Système de sélection de lineup

1. Sélection de 5 joueurs
2. Extraction des statistiques individuelles et des données du combine
3. Conservation des positions (G, F, C) pour analyse de cohérence

## Système de bonus de cohérence

Un bonus est appliqué au taux de victoire prédit selon la composition:
- +15% pour lineup parfaite (2G, 2F, 1C)
- +10% pour lineup avec les 3 positions
- +5% pour lineup avec 2 positions
- -5% pour lineup avec une seule position

## Procédure de prédiction

Pour comparer deux lineups aléatoires:
1. Sélection de deux lineups avec `select_random_lineup()`
2. Calcul des statistiques agrégées pour chaque lineup
3. Prédiction du taux de victoire avec le modèle RandomForest
4. Application du bonus de cohérence
5. Comparaison des scores finaux
6. Affichage détaillé des joueurs et statistiques du lineup gagnant

Le système fournit une comparaison objective basée sur les statistiques avancées tout en valorisant la complémentarité des positions au sein d'une équipe.



## Modèles de machine learning utilisés
Nous avons formulé le problème comme une classification binaire : prédire si une équipe gagne (1) ou perd (0) selon les statistiques de ses 5 joueurs.

Nous avons sélectionné plusieurs modèles complémentaires, en tenant compte de la performance, de l’interprétabilité et du volume de données :

1. RandomForestClassifier (modèle principal de départ)
Un excellent choix pour les données tabulaires, même peu pré-traitées.

✔️ Avantages :

Robuste face au surapprentissage

Pas besoin de normaliser les données

Tolérant aux features corrélées

Rapide à entraîner

Permet d’interpréter les features importantes (e.g. net_rating, usg_pct, etc.)

2. LogisticRegression (baseline simple et interprétable)
Modèle linéaire pour valider la qualité du dataset.

✔️ Avantages :

Très rapide à entraîner

Interprétation facile des poids (influence directe des variables)

Bon point de départ pour valider la pertinence des features

3. XGBoostClassifier (modèle avancé orienté performance)
Un modèle de gradient boosting réputé pour sa précision.

✔️ Avantages :

Très performant sur petits et moyens datasets

Gère bien les valeurs manquantes

Paramétrage fin (early stopping, learning rate, etc.)

Importance des features et gain à chaque split fournis

4. MLPClassifier (réseau de neurones multi-couches)
Un modèle non-linéaire qui peut apprendre des synergies complexes entre joueurs.

✔️ Avantages :

Capable de modéliser des interactions subtiles (e.g. “LeBron + Kyrie” ≠ “LeBron seul” + “Kyrie seul”)

Potentiel élevé avec des données issues de plusieurs saisons

Peut généraliser des patterns profonds non triviaux
