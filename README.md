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


## Autres


```python
import foobar

# returns 'words'
foobar.pluralize('word')

# returns 'geese'
foobar.pluralize('goose')

# returns 'phenomenon'
foobar.singularize('phenomena')
```
