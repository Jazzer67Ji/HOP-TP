# QUIZ — MongoDB (10 Questions) : Réponses

---

## 1️⃣ Différence fondamentale entre modèle relationnel et modèle document

- **Relationnel (SQL)** : données stockées dans des **tables** avec des lignes et colonnes, schéma strict, relations via des clés étrangères (JOIN)
- **Document (MongoDB)** : données stockées dans des **documents JSON/BSON** imbriqués, schéma flexible, les données liées peuvent être intégrées dans le même document

---

## 2️⃣ L'opérateur `$group`

`$group` regroupe les documents selon un critère et permet d'appliquer des fonctions d'agrégation (`$sum`, `$avg`, `$count`, etc.).

**Équivalent SQL** : `GROUP BY`

```js
// Mongo
db.taxi.aggregate([
  { $group: { _id: "$zone", total: { $sum: "$montant" } } }
])

-- SQL équivalent
SELECT zone, SUM(montant) FROM taxi GROUP BY zone;
```

---

## 3️⃣ Différence entre `$match` et `find()`

- **`find()`** : requête simple directement sur une collection, utilisé seul
- **`$match`** : filtre utilisé **à l'intérieur d'un pipeline d'agrégation** (`aggregate()`), peut être combiné avec `$group`, `$lookup`, etc.

```js
// find simple
db.taxi.find({ zone: "Paris" })

// $match dans un pipeline
db.taxi.aggregate([
  { $match: { zone: "Paris" } },
  { $group: { _id: "$chauffeur", total: { $sum: "$montant" } } }
])
```

---

## 4️⃣ Pourquoi MongoDB n'impose pas de schéma strict ? Quels risques ?

**Avantage** : flexibilité totale, chaque document peut avoir des champs différents, idéal pour des données évolutives ou hétérogènes.

**Risques** :
- Incohérence des données (un champ `montant` en string dans un doc, en number dans un autre)
- Pas de contraintes d'intégrité natives
- Requêtes qui retournent des résultats inattendus si les champs sont mal nommés
- Maintenance difficile à grande échelle sans convention stricte

---

## 5️⃣ Embedding vs Referencing

- **Embedding** : on intègre les données liées **directement dans le document** parent → lecture rapide, tout en un seul accès
- **Referencing** : on stocke uniquement l'`_id` du document lié et on fait une jointure via `$lookup` → données moins dupliquées, mais lecture plus coûteuse

**Règle générale** :
- Embedding si les données sont toujours lues ensemble et peu volumineuses
- Referencing si les données liées sont partagées entre plusieurs documents ou très volumineuses

---

## 6️⃣ L'opérateur `$lookup`

`$lookup` permet de faire une **jointure entre deux collections** dans un pipeline d'agrégation.

**Équivalent SQL** : `JOIN`

```js
// Mongo
db.commandes.aggregate([
  {
    $lookup: {
      from: "clients",
      localField: "client_id",
      foreignField: "_id",
      as: "infos_client"
    }
  }
])

-- SQL équivalent
SELECT * FROM commandes JOIN clients ON commandes.client_id = clients._id;
```

---

## 7️⃣ Pourquoi Mongo est performant pour les données semi-structurées ?

- Le format **BSON (JSON binaire)** est nativement adapté aux données imbriquées et variables
- Pas besoin de normaliser : un document peut contenir tableaux, objets imbriqués, champs optionnels
- Lecture d'un document entier en **un seul accès disque** (pas de JOIN coûteux)
- Scalabilité horizontale native (**sharding**) pour de grands volumes

---

## 8️⃣ Différence entre `insertOne()` et `updateOne()`

- **`insertOne()`** : crée un **nouveau document** dans la collection
- **`updateOne()`** : **modifie un document existant** qui correspond au filtre (ne crée rien si rien ne correspond, sauf avec l'option `upsert: true`)

```js
// Insertion d'un nouveau document
db.taxi.insertOne({ chauffeur: "Paul", montant: 15.5 })

// Mise à jour d'un document existant
db.taxi.updateOne(
  { chauffeur: "Paul" },
  { $set: { montant: 20.0 } }
)
```

---

## 9️⃣ Pourquoi l'indexation est critique dans Mongo ? Comment créer un index ?

**Sans index**, Mongo fait un **full collection scan** : il parcourt tous les documents un par un → très lent sur de gros volumes.

**Avec index**, Mongo utilise une structure optimisée (B-Tree) pour trouver les documents en O(log n).

```js
// Créer un index sur le champ "zone"
db.taxi.createIndex({ zone: 1 })  // 1 = ordre croissant, -1 = décroissant

// Vérifier les index existants
db.taxi.getIndexes()
```

---

## 🔟 Dans quel cas MongoDB est moins adapté que Neo4j ?

MongoDB est moins adapté quand les données sont **fortement relationnelles et interconnectées**, notamment :
- Systèmes de **recommandation** (qui a aimé quoi, qui connaît qui)
- **Réseaux sociaux** (amis d'amis, graphe de relations)
- **Détection de fraude** (chaînes de transactions liées)
- **Routage et chemins** (le plus court chemin entre deux nœuds)

Neo4j est conçu pour traverser des relations complexes de façon native et très performante, là où Mongo devrait multiplier les `$lookup` coûteux.

---

## 🎓 Bonus — Système de recommandation Taxi : SQL, Mongo ou Neo4j ?

**Réponse : Neo4j**, et voici pourquoi techniquement.

Un système de recommandation Taxi repose sur des relations entre :
- des **passagers** (qui prennent des courses similaires)
- des **zones** (points de départ/arrivée fréquents)
- des **chauffeurs** (préférences, habitudes)
- des **horaires** (patterns temporels)

**Pourquoi pas SQL ?**
Les jointures récursives (ex: "les passagers qui ont pris les mêmes trajets que toi ont aussi aimé...") sont très coûteuses en SQL et complexes à écrire.

**Pourquoi pas MongoDB seul ?**
Mongo est excellent pour stocker les courses et les profils, mais les relations entre entités (graph traversal) nécessitent de nombreux `$lookup` imbriqués → peu performant et difficile à maintenir.

**Pourquoi Neo4j ?**
- Chaque course, passager, zone devient un **nœud**
- Les relations (a_pris, part_de, arrive_a) deviennent des **arêtes**
- La requête "trouver des trajets similaires à partir de mes habitudes" s'écrit nativement en **Cypher** et est exécutée en temps réel
- Neo4j est optimisé pour le **graph traversal** : parcourir des millions de relations sans JOIN

**Architecture idéale en production** :
- **MongoDB** → stockage brut des courses (données opérationnelles)
- **Neo4j** → moteur de recommandation (relations entre passagers/zones)
- **SQL** → reporting et agrégats métier
