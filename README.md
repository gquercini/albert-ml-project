# Y'a que la taille qui compte?

Lataillequi est un projet réalisé en 2e année de Bachelor Business & Data à AlbertSchool dans le cadre du cours de _Data: Deploying an ML project_ .

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

### Glossaire datasets PARTIE 1

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


### Glossaire datasets PARTIE 2

- play_by_play
- inactive_player
- other_stats
- draft_combine_strat
- game_info, game_summary


## Méthodes employées
Pour atteindre notre objectif, nous avons suivi les étapes suivantes :

Nettoyage des données : suppression des colonnes inutiles, harmonisation des formats, traitement des valeurs manquantes.

Construction d’un dataset d’entraînement : chaque ligne représente une combinaison réelle de 5 joueurs, associée à une équipe, une saison, et un résultat de match (victoire ou défaite).

Agrégation des statistiques des 5 joueurs pour créer un vecteur de caractéristiques représentatif de la lineup.

Entraînement de modèles de machine learning afin de prédire la probabilité de victoire d’une composition en fonction de ses statistiques agrégées.

Les données sont donc utilisées à la fois pour :

Construire les features d’entrée (X)

Définir la variable cible (y = victoire/défaite)

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
