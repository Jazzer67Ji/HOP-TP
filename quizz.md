Quiz 2 — Lab 3.2 : Réponses sur le Data Quality Dashboard
1. Pourquoi mesurer le taux de rejet est plus important que le nombre brut de lignes rejetées ?
Le taux de rejet (ex: 5% des lignes) permet de comparer la qualité entre des volumes de données différents. Un nombre brut (ex: 1000 lignes rejetées) ne tient pas compte de la taille du jeu de données : 1000 rejets sur 1 million de lignes (0.1%) est moins critique que 1000 rejets sur 10 000 lignes (10%).

Ici nous avons environ 200 000 lignes sur 6 000 000 ce qui fait environ 3,33%. Ce qui dans notre cas est faible.

2. Si le taux de rejet passe de 3 % à 18 % en une journée, quelles hypothèses peux-tu formuler ?
Problème de source : Changement de format ou de structure dans les données d'entrée (ex: nouveau fournisseur, mise à jour d'API).
Erreur de pipeline : Modification récente dans le traitement (ex: nouvelle règle de validation, bug dans une transformation).
Données corrompues : Fichier source incomplet ou mal généré (ex: export truncqué, encodage incorrect).
Volume anormal : Afflux de données non conformes (ex: spam, données de test).
3. Quelle transformation Hop permet de calculer des agrégations comme AVG, COUNT ou MAX ?
Le composant "Group By" permet de calculer des agrégations (AVG, COUNT, MAX, MIN, SUM, etc.) en regroupant les données selon une ou plusieurs colonnes.

4. Quelle différence entre :
COUNT(*) : Compte toutes les lignes d'un groupe, y compris les valeurs NULL.
COUNT(colonne) : Compte uniquement les lignes où la colonne spécifiée n'est pas NULL.
5. Pourquoi un dashboard de qualité doit-il être historisé ?
Suivi des tendances : Détecter des dégradations ou améliorations dans le temps.
Analyse des causes : Corréler les pics d'erreurs avec des événements (ex: migrations, changements de source).
Preuve de conformité : Montrer l'évolution de la qualité pour des audits ou rapports.
6. Quel est le risque d’utiliser uniquement la moyenne pour analyser total_amount ?
La moyenne masque les extrêmes (ex: une valeur aberrante très élevée peut fausser la moyenne). Il est préférable d'utiliser :

Médiane (moins sensible aux outliers).
Écart-type ou percentiles (pour voir la distribution).
Visualisations (histogrammes, boxplots).
7. À partir de quel seuil une donnée devient-elle un problème ? Qui décide ce seuil ?
Seuil : Dépend du contexte métier (ex: 1% de rejets peut être acceptable pour des logs, mais critique pour des transactions financières).
Décision : C'est une collaboration entre les équipes data et métiers, en fonction des impacts (coût, risque, réglementation).
8. Quelle différence entre :
Métrique descriptive : Décrit un état (ex: "Le taux de rejet est de 5%").
Métrique d’alerte : Déclenche une action si un seuil est dépassé (ex: "Alerte si le taux de rejet > 10%").
9. Pourquoi exporter les métriques dans une table SQL peut être préférable à un CSV ?
Centralisation : Accès simultané pour plusieurs utilisateurs.
Historisation automatique : Possibilité de créer des vues ou requêtes temporelles.
Intégration : Connexion directe avec des outils de visualisation (Tableau, Power BI).
Sécurité : Gestion des droits d'accès et sauvegardes automatiques.
10. Si les données sont propres mais incohérentes d’un point de vue métier, le dashboard le détectera-t-il ? Pourquoi ?
Non, un dashboard de qualité technique ne détecte que des erreurs de format ou de règles prédéfinies (ex: valeurs NULL, types incorrects).
Pour détecter des incohérences métiers, il faut :
Des règles métiers spécifiques (ex: "un client ne peut pas avoir 2 commandes au même timestamp").
Des tests de cohérence (ex: croiser avec d'autres sources de données).
Une validation humaine (revue par les experts métiers).
