# Détection de fraude à l'assurance

Projet de fin du bootcamp **Jedha Fullstack** (certification RNCP38777) — **Carelle Nouko**

> Les données sont **fictives** : elles viennent d'un jeu de données public de Kaggle. Le but de ce projet est de **montrer ma méthode** de bout en bout (fusion de tables, RGPD, pipeline, choix du modèle, seuil), pas d'obtenir un modèle prêt pour la vraie vie.

## 1. Le problème

Un assureur reçoit des déclarations de sinistres (par exemple un accident de voiture). Une petite partie sont des **fraudes** : des gens qui trichent pour être remboursés. Les enquêteurs ne peuvent pas tout vérifier à la main.

**Mon objectif :** construire un modèle qui repère les dossiers suspects, pour dire aux enquêteurs **quels dossiers vérifier en premier**.

Difficulté : dans mes données, seulement **3 %** des dossiers sont des fraudes. Il y en a très peu, donc c'est difficile à repérer.

## 2. Les données

Cinq fichiers CSV, dans le dossier `data/` :

| Fichier | Contenu |
|---|---|
| `customers.csv` | Les clients (âge, catégorie, groupe…) |
| `contracts.csv` | Les contrats d'assurance (dates de début et de fin…) |
| `departements.csv` | Les départements (avec latitude et longitude) |
| `historical.csv` | Les anciens sinistres, avec l'information « fraude : oui / non » |
| `newclaims.csv` | De nouveaux sinistres, **sans** réponse, à noter par le modèle |

Ce sont des données **fictives** (jeu de données public de Kaggle). Elles ne sont **pas** sur GitHub : il faut les placer dans le dossier `data/`.

## 3. Ce que fait le notebook (`notebooks/01_modele_fraude.ipynb`)

| Étape | Ce que je fais |
|---|---|
| 1 à 2 | Je charge les 5 fichiers et je les explore |
| 3 | Je crée 2 nouvelles colonnes : l'âge du client et la durée du contrat en jours |
| 4 | **RGPD** : je supprime les données personnelles (nom, prénom, date de naissance) avant de fusionner. On ne garde que ce qui est utile |
| 5 | Je fusionne les 4 tables en un seul tableau : **1 497 dossiers** |
| 6 | Je sépare 80 % des dossiers pour apprendre et 20 % pour tester (**300 dossiers dont 10 fraudes**), et je prépare les données avec un pipeline |
| 7 | Je regarde les données avec des graphiques |
| 8 | Je compare 3 modèles : Decision Tree, Random Forest, XGBoost |
| 9 | Je construis mon modèle final, plus simple, et je le compare à une règle de bon sens |
| 10 à 11 | Je regarde où le modèle se trompe, et j'essaie plusieurs seuils |
| 12 | J'explique ce que le modèle a appris (les coefficients) |
| 13 | Je note les nouveaux sinistres de `newclaims.csv` |
| 14 | Conclusion et limites |

**Le pipeline** : une chaîne qui prépare les données toujours de la même façon (remplir les cases vides, mettre les nombres à la même échelle, transformer les catégories en 0 et 1). Je le règle **seulement sur les données d'apprentissage**, pour ne pas « tricher » avec les données de test.

## 4. Mes résultats

**Le benchmark** (note F1 : une note qui mélange les fraudes trouvées et les alertes justes) :

| Modèle | F1 |
|---|---|
| Decision Tree | 0,29 |
| Random Forest | 0,00 |
| XGBoost | 0,33 |

**Mon modèle final** est une **régression logistique** avec 6 variables choisies à la main (montant du sinistre, âge, durée du contrat, nombre de contrats sur 3 ans, catégorie et groupe du client). Je l'ai gardée car on peut **expliquer** ses résultats, et parce qu'elle trouve bien les fraudes. Elle est réglée pour donner plus d'importance aux fraudes (`class_weight='balanced'`).

Le **seuil**, c'est le niveau de certitude à partir duquel le modèle dit « fraude ». Plus on le monte, moins il fait de fausses alertes, mais plus il rate de fraudes :

| Seuil | Fraudes trouvées (sur 10) | Fausses alertes | Alertes qui sont vraiment des fraudes |
|---|---|---|---|
| 0,3 | 8 | 142 | 5,3 % |
| 0,5 (par défaut) | 6 | 103 | 5,5 % |
| 0,7 | 5 | 44 | 10,2 % |
| 0,9 | 0 | 0 | — |

- Avec le seuil de 0,7, le modèle signale 49 dossiers et trouve 5 fraudes sur 10. En choisissant 49 dossiers au hasard, on en trouverait environ 1 ou 2 : le modèle fait **mieux que le hasard**.
- La règle simple « un gros montant est suspect » ne trouve **aucune** fraude (F1 = 0).

**Ce que le modèle a appris** (les « odds ratio » : au-dessus de 1, le risque monte ; en dessous de 1, il baisse) : les clients de la catégorie « Workers » (3,1) et du groupe B (1,8) ont un risque plus élevé, ainsi que les clients plus âgés (1,6). Le montant du sinistre a un odds ratio de 0,02 : les **petits** montants paraissent plus suspects. Ça n'aurait pas de sens dans la vraie vie : c'est une particularité de ces données fictives.

## 5. Mes limites

- Il y a seulement **10 fraudes dans le test** : c'est trop peu pour être sûr. Une fraude de plus ou de moins change le résultat de 10 points.
- J'ai choisi le seuil **en regardant le jeu de test** : le résultat est un peu trop beau.
- Il y a beaucoup de **fausses alertes** : chacune devrait être vérifiée par une personne.
- Les données sont **fictives** : je valide ma méthode, pas un modèle prêt pour la vraie vie.
- Je n'ai fait **qu'une seule séparation** entre apprentissage et test.
- Des messages d'avertissement (`RuntimeWarning`) apparaissent pendant l'entraînement : ce sont des calculs numériques un peu instables. Les résultats sont les mêmes à chaque lancement.

## 6. Lancer le projet

```bash
git clone https://github.com/Carelle777/projet_fraude.git
cd projet_fraude
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# placer les 5 fichiers CSV dans le dossier data/
# puis ouvrir notebooks/01_modele_fraude.ipynb et cliquer sur « Run All »
```

## 7. Structure du projet

```
projet_fraude/
├── README.md
├── requirements.txt
├── .gitignore
├── data/                 (les fichiers CSV sont à placer ici, ils ne sont pas sur GitHub)
└── notebooks/
    └── 01_modele_fraude.ipynb
```

## 8. Petit lexique

- **Fraude :** un sinistre déclaré de façon malhonnête.
- **Seuil :** le niveau de certitude à partir duquel le modèle dit « fraude ».
- **Fausse alerte (faux positif) :** un dossier normal signalé à tort.
- **Fraude ratée (faux négatif) :** une vraie fraude que le modèle n'a pas vue.
- **F1 :** une note de 0 à 1 qui mélange les fraudes trouvées et les alertes justes.
- **Pipeline :** une chaîne qui prépare les données puis prédit, toujours dans le même ordre.
- **RGPD :** la loi qui protège les données personnelles. J'ai supprimé les données inutiles avant de fusionner.
- **Odds ratio :** au-dessus de 1, le facteur augmente le risque de fraude ; en dessous de 1, il le diminue.

## 9. Outils

Python · pandas · numpy · scikit-learn · XGBoost · matplotlib · seaborn · Jupyter