```
CREATE INDEX FOR (p:Publication) ON (p.id)
```

```
CREATE INDEX FOR (a:Author) ON (a.name)
```

```
CREATE INDEX FOR (v:Venue) ON (v.name)
```

```
//import authors
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "FOREACH(author IN value.authors |
     MERGE (a:Author {name: author})
     MERGE (a)-[:authorOf]->(p:Publication {id: value.id})
  )",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//import publications
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "MERGE (p:Publication {id: value.id})
   SET p.title = value.title,
       p.abstract = value.abstract,
       p.n_citation = value.n_citation,
       p.year = value.year",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//if exists: import referenced publications 
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "FOREACH(reference IN value.references |
     MERGE (r:Publication {id: reference})
  )",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//delete duplicate publications
MATCH (p:Publication)
WITH p 
ORDER BY p.id, size([(p)--(other) | other]) DESC
WITH p.id as id, collect(p) AS nodes 
WHERE size(nodes) >  1
UNWIND nodes[1..] AS n
DETACH DELETE n
```

```
//wenn existent: venue importieren 
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "FOREACH(_ IN CASE WHEN value.venue = '' THEN [] ELSE [value.venue] END |
     MERGE (v:Venue {name: value.venue})
  )",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//position von autoren setzen
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "MATCH (p:Publication) WHERE p.id = value.id
   UNWIND range(1, size(value.authors)) AS position
   MATCH (a:Author {name: value.authors[position - 1]})
   MERGE (a)-[x:authorOf]->(p)
   SET x.position = position",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//publishedIn-relationship zw publikationen und venues:
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "MATCH (p:Publication {id: value.id})
   FOREACH(venue IN CASE WHEN value.venue = '' THEN [] ELSE [value.venue] END |
     MERGE (v:Venue {name: venue})
     MERGE (p)-[:publishedIn]->(v)
   )",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```

```
//cites-relationship zw publikationen
CALL apoc.periodic.iterate(
  "CALL apoc.load.json('file:///dblp-ref-3.json') YIELD value",
  "MATCH (p1:Publication {id: value.id})
   UNWIND value.references AS refId
   MATCH (p2:Publication {id: refId})
   MERGE (p1)-[:cites]->(p2)",
  {batchSize: 1000, iterateList: true, parallel: false}
)
```
