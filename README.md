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

## PARTIE 1

L’objectif du projet est de *prédire automatiquement la meilleure composition de 5 joueurs NBA ("dream team") par saison*, c’est-à-dire la combinaison qui maximise les chances de victoire de l'équipe.

### Glossaire datasets PARTIE 1

- **all_seasons.csv** : statistiques par joueur et par saison (points, rebonds, passes, usage, efficacité au tir, etc.)
- **game.csv** : résultats des matchs avec les équipes concernées et les scores.
- **player.csv** : identifiants et noms des joueurs.
- **line_score.csv** : scores par équipe pour chaque match, utile pour valider les résultats.

## PARTIE 2

Le projet permettra aussi de *comparer deux lineups de joueurs* et de *prédire laquelle est la plus performante dans un match simulé*.


### Glossaire datasets PARTIE 2

- play_by_play
- inactive_player
- other_stats
- draft_combine_strat
- common_players
- team, team_history, team_summary, team_details


### Méthodes employées :

Pour atteindre cet objectif, nous allons :

1.⁠ ⁠Nettoyer les datasets pour ne conserver que les informations pertinentes.
2.⁠ ⁠Construire un dataset d'entraînement où chaque ligne représente une *combinaison réelle de 5 joueurs* associée à une équipe, une saison, et un résultat de match (victoire ou défaite).
3.⁠ ⁠Aggréger les statistiques des 5 joueurs pour former un vecteur de caractéristiques représentatif de la composition.
4.⁠ ⁠Entraîner un modèle de machine learning à prédire la probabilité de victoire en fonction des statistiques agrégées d'une lineup.

Les données seront donc utilisées à la fois pour *construire les features d'entrée (X)* et pour définir la *variable cible (y = victoire/défaite)*.

### Méthodes de machine learning utilisées : 

Pour identifier la meilleure composition de joueurs par saison (la “dream team”), nous avons formulé le problème comme une classification binaire : prédire si une équipe gagne (1) ou perd (0) en fonction des caractéristiques de ses 5 joueurs.

Nous avons sélectionné plusieurs modèles complémentaires, chacun ayant ses forces selon le volume de données, la performance attendue, et l’interprétabilité.

1.⁠ ⁠RandomForestClassifier (modèle principal de départ)
→Il fonctionne très bien sur des données tabulaires avec peu de preprocessing.
→Il permet une interprétation directe des variables importantes (features clés comme net_rating, usg_pct…).

✔️ Avantages :
Résistant au surapprentissage.
Pas besoin de scaler les données.
Fonctionne bien même avec des features corrélées.
Rapide à entraîner pour un premier benchmark.

2.⁠ ⁠LogisticRegression (baseline simple et interprétable)
→Pour avoir un modèle de base interprétable.
→Permet de vérifier si les features choisies ont un impact statistique fort.

✔️ Avantages :
Très rapide à entraîner.
Les poids du modèle permettent une lecture directe des influences.
Utile pour valider si le dataset est bien structuré avant des modèles plus complexes.

3.⁠ XGBoostClassifier (modèle avancé pour optimiser les performances)
→C’est un modèle de gradient boosting, reconnu pour sa précision exceptionnelle en machine learning.
→Il excelle dans les compétitions Kaggle (et ton dataset en vient).

✔️ Avantages :
Performances élevées sur petits et moyens datasets.
Résiste bien aux valeurs manquantes.
Permet de gérer très finement les paramètres d’entraînement (early stopping, learning rate…).
Fournit une importance des features et un gain associé à chaque split.

4.⁠ ⁠Multi-Layer Perceptron (MLPClassifier) — Réseau de neurones basique
→Pour tester un modèle non linéaire plus profond, qui pourrait apprendre des synergies complexes entre joueurs (interactions non triviales).

✔️ Avantages :
Capable de modéliser des relations plus subtiles entre les stats (ex : “LeBron + Kyrie” ≠ “LeBron seul” + “Kyrie seul”).
Si on alimente le modèle avec beaucoup de données (plusieurs saisons), il peut généraliser des patterns plus profonds.
