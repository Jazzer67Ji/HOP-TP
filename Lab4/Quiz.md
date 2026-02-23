## **Question 1 : Différence entre CREATE et MERGE**

CREATE crée systématiquement un nouveau nœud ou une nouvelle relation à chaque exécution, même si un élément identique existe déjà, ce qui peut générer des doublons. MERGE vérifie d'abord si l'élément existe déjà selon les propriétés spécifiées, et ne le crée que s'il est absent, garantissant l'unicité. Par exemple, CREATE (:Chauffeur {id: 123}) exécuté deux fois créera deux nœuds distincts, tandis que MERGE (:Chauffeur {id: 123}) créera le nœud une seule fois. MERGE est idempotent et essentiel dans les pipelines ETL pour éviter la duplication des données.

---

## **Question 2 : Pourquoi Cypher est un langage déclaratif**

Cypher permet de décrire ce qu'on veut obtenir sans spécifier comment le système doit le faire techniquement. On définit des patterns de graphe à trouver, et Neo4j optimise automatiquement la requête et choisit les index à utiliser. Par exemple, MATCH (a)-[:CONNAIT]->(b) décrit le pattern recherché sans indiquer l'algorithme de traversée. Le développeur se concentre sur la logique métier plutôt que sur l'implémentation bas niveau. C'est similaire à SQL qui est aussi déclaratif, contrairement aux langages procéduraux comme Java où on doit coder explicitement chaque étape.

---

## **Question 3 : Que signifie MATCH (a)-[:REL]->(b)**

Cette syntaxe recherche un pattern dans le graphe où un nœud `a` est connecté à un nœud `b` par une relation de type REL. La flèche `->` indique que la relation est dirigée de `a` vers `b`, donc orientée. Les parenthèses définissent les nœuds et les crochets définissent les relations. Les variables `a` et `b` permettent de référencer ces nœuds dans le reste de la requête pour filtrer, retourner ou modifier leurs propriétés. Sans la flèche, la relation serait bidirectionnelle.

---

## **Question 4 : Différence entre COUNT(*) et COUNT(DISTINCT)**

COUNT(*) compte toutes les lignes retournées par la requête, y compris les doublons potentiels si un même élément apparaît plusieurs fois dans les résultats. COUNT(DISTINCT variable) compte uniquement le nombre de valeurs uniques de cette variable, en éliminant les duplicatas. Par exemple, si un chauffeur a effectué 10 courses, COUNT(*) retournera 10 mais COUNT(DISTINCT chauffeur) retournera 1. COUNT(DISTINCT) est crucial pour compter le nombre réel d'entités distinctes plutôt que le nombre d'occurrences ou de relations.

---

## **Question 5 : Pourquoi WITH est-il nécessaire**

WITH permet de chaîner plusieurs opérations dans une seule requête en créant des étapes intermédiaires. Il sert à passer des résultats d'une partie de la requête à la suivante, comme un pipe en programmation. Il permet de filtrer, agréger ou transformer les données avant de continuer le traitement. Par exemple, WITH permet de calculer d'abord des agrégations puis de filtrer sur ces résultats, ce qui serait impossible directement. Il structure les requêtes complexes en les rendant lisibles et maintenables en séparant la logique en blocs cohérents. Sans WITH, certaines requêtes nécessiteraient plusieurs appels séparés avec stockage temporaire.

---

## **Question 6 : Comment éviter les doublons dans un ETL graphe**

Utiliser systématiquement MERGE au lieu de CREATE pour garantir l'unicité des nœuds et relations. Définir des contraintes d'unicité sur les propriétés clés comme les identifiants avec CREATE CONSTRAINT. Toujours spécifier les propriétés discriminantes dans les MERGE pour identifier correctement les éléments existants. Nettoyer les données sources avant l'import pour éliminer les doublons en amont. Utiliser ON CREATE SET et ON MATCH SET pour gérer différemment les créations et les mises à jour. Tester régulièrement avec des requêtes COUNT pour détecter les duplicatas accidentels. Rendre le pipeline idempotent pour que rejouer le même import ne crée pas de doublons.

---

## **Question 7 : Différence entre relation SQL et relation graphe**

En SQL, une relation est modélisée par une jointure via des clés étrangères stockées dans des tables, nécessitant des JOIN coûteux pour reconstruire les liens. Dans un graphe, les relations sont des entités de première classe stockées physiquement comme des pointeurs directs entre nœuds, rendant la traversée instantanée. Les relations graphes peuvent avoir leurs propres propriétés, comme date, montant, ou type, alors qu'en SQL cela nécessite une table intermédiaire. Les relations graphes sont directionnelles par défaut et peuvent être typées, facilitant la modélisation de réseaux complexes. Les requêtes sur relations multiples sont exponentiellement plus rapides en graphe qu'avec des JOIN SQL imbriqués.

---

## **Question 8 : Pourquoi les graphes sont puissants pour la fraude**

Les graphes révèlent naturellement les patterns de connexions suspects entre entités comme plusieurs comptes utilisant la même carte bancaire ou adresse IP. Ils permettent de détecter des communautés suspectes et des réseaux de complices en analysant les clusters densément connectés. Les algorithmes de centralité identifient les nœuds clés dans des réseaux de fraude organisée. La traversée en profondeur permet de découvrir des liens indirects invisibles dans des tables SQL, comme un fraudeur à 3 degrés de séparation. Les patterns temporels de relations peuvent révéler des comportements coordonnés. L'analyse de chemins entre entités met en évidence les circuits d'argent sale ou les structures de blanchiment.

---

## **Question 9 : Que fait DETACH DELETE**

DETACH DELETE supprime un nœud ainsi que toutes ses relations entrantes et sortantes en une seule opération atomique. Sans DETACH, Neo4j refuse de supprimer un nœud qui a encore des relations pour maintenir l'intégrité du graphe. C'est l'équivalent d'un DELETE CASCADE en SQL qui nettoie automatiquement les dépendances. Par exemple, DETACH DELETE (c:Chauffeur {id: 123}) supprime le chauffeur et toutes ses courses, passagers, et autres connexions. Cette commande doit être utilisée avec précaution car elle peut supprimer plus de données que prévu si le nœud est fortement connecté.

---

## **Question 10 : Pourquoi un graphe est plus naturel pour modéliser des réseaux**

Les graphes reflètent directement la façon dont les humains pensent les relations et connexions entre entités. La structure graphe correspond exactement à des concepts réels comme les réseaux sociaux, les transports, ou les organigrammes d'entreprise. Il n'y a pas de friction cognitive entre le modèle mental et le modèle de données, contrairement aux tables SQL qui nécessitent de tout décomposer. Les requêtes graphe s'écrivent en décrivant visuellement les patterns recherchés, ce qui est intuitif. Le schéma est flexible et évolue naturellement sans migrations complexes quand on ajoute de nouveaux types de relations. La visualisation graphique permet une compréhension immédiate des structures et patterns qui seraient illisibles dans des tables relationnelles.