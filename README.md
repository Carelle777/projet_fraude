# Détection de Fraude en Assurance (Projet RNCP) ## 👥 Contexte de Réalisation
Ce projet a été réalisé en binôme dans le cadre de la validation du **Bloc Fullstack** de ma formation Jedha. Il concrétise notre capacité à collaborer sur une codebase commune, à structurer un pipeline de Machine Learning de bout en bout et à le présenter de manière professionnelle.

Bienvenue sur mon projet de certification. Ce dépôt contient un pipeline de Machine Learning complet conçu pour identifier les réclamations frauduleuses dans le secteur de l'assurance.

## Contexte Business
La fraude à l'assurance représente une perte financière majeure. L'objectif de ce projet est de détecter automatiquement un signal très faible (les fraudes représentent seulement 3% de notre jeu de données) afin de bloquer les paiements suspects et d'aider les enquêteurs à prioriser leurs audits manuels.

*Note : Les données utilisées pour ce projet proviennent d'un dataset fictif public issu de Kaggle. N'étant pas de véritables données de production, elles ne contiennent pas de signaux métiers profonds (comme l'historique bancaire réel), ce qui explique le plafond de verre mathématique des scores finaux. L'objectif est ici de valider la méthodologie technique de bout en bout.*

## Stack Technique
*   **Langage :** Python
*   **Data Manipulation :** Pandas, Numpy
*   **Machine Learning :** Scikit-Learn (Régression Logistique, Random Forest, Decision Tree), XGBoost
*   **Visualisation :** Matplotlib, Seaborn

## Démarche et Pipeline
1.  **Data Warehouse :** Fusion de 4 bases de données relationnelles (Clients, Contrats, Départements, Historique) via leurs clés primaires.
2.  **Feature Engineering :** Création de variables continues (âge du client, durée du contrat en jours) et nettoyage strict des données personnelles pour la conformité RGPD.
3.  **Pipeline MLOps :** Création d'un `ColumnTransformer` (Imputation par la médiane, Standardisation, Encodage One-Hot). Ce pipeline garantit une protection absolue contre la fuite de données (Data Leakage) grâce à un split stratifié initial.
4.  **Modélisation (Imbalanced Data) :** Benchmark de plusieurs algorithmes lourds (XGBoost, Random Forest). Le choix final s'est porté sur une Régression Logistique pénalisée (`class_weight='balanced'`) couplée à une réduction de dimensionnalité pour éliminer le bruit et maximiser la détection.

## Résultats
Le modèle final parvient à intercepter **70% des fraudes** dans un environnement asymétrique extrême. Ce résultat démontre une application réussie du *Cost-Sensitive Learning*, où l'algorithme est mathématiquement configuré pour comprendre que rater une fraude coûte beaucoup plus cher à l'entreprise que de générer un faux positif de contrôle.