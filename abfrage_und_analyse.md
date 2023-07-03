# Teil 1

## Q1
Ausgabe aller Publikationen, bei denen "Erhard Rahm" einer der Autoren ist.
```
// Ausgabe aller Publikationen, bei denen "Erhard Rahm" einer der Autoren ist.
MATCH (a:Author {name: "Erhard Rahm"})-[:authorOf]->(p:Publication)
RETURN p
```

> 11


## Q2
Ausgabe aller Koautoren von "Erhard Rahm".
```
// Ausgabe aller Koautoren von "Erhard Rahm".
MATCH (a:Author {name: "Erhard Rahm"})-[:authorOf]->(:Publication)<-[:authorOf]-(coauthor)
RETURN coauthor
```

> 21


## Q3
Ausgabe aller Publikationen (Titel), die Publikationen von dem Autor "Wei Wang" zitieren.
```
// Ausgabe aller Publikationen (Titel), die Publikationen von dem Autor "Wei Wang" zitieren.
MATCH (p1:Publication)<-[:cites]-(:Publication {authors: "Wei Wang"})
RETURN p1.title
```

> (no changes, no records)


## Q4
Ausgabe aller Venues, deren Publikationen mindestens eine im Titel das Wort "graph" und das Wort "temporal" (beide case insensitive) enthält. Neben dem duplikatfreien Venue-Namen in der ersten Spalte soll in einer zweiten Spalte auch eine duplikatfreie Liste der Publikations-Titel ausgegeben werden, bei denen beide gesuchten Worte vorkommen.
```
// Ausgabe aller Venues, deren Publikationen mindestens eine im Titel das Wort "graph" und das Wort "temporal" (beide case insensitive) enthält. Neben dem duplikatfreien Venue-Namen in der ersten Spalte soll in einer zweiten Spalte auch eine duplikatfreie Liste der Publikations-Titel ausgegeben werden, bei denen beide gesuchten Worte vorkommen.
MATCH (v:Venue)<-[:publishedIn]-(p:Publication)
WHERE toLower(p.title) CONTAINS "graph" AND toLower(p.title) CONTAINS "temporal"
RETURN DISTINCT v.name AS Venue, COLLECT(DISTINCT p.title) AS Publications
```

> 17


## Q5
Wie viele Autoren haben pro Jahr bei der Venue mit dem Namen "Lecture Notes in Computer Science" veröffentlicht? Ausgabe aufsteigend sortiert nach Jahr.
```
// Wie viele Autoren haben pro Jahr bei der Venue mit dem Namen "Lecture Notes in Computer Science" veröffentlicht? Ausgabe aufsteigend sortiert nach Jahr.
MATCH (v:Venue {name: "Lecture Notes in Computer Science"})<-[:publishedIn]-(p:Publication)<-[:authorOf]-(a:Author)
RETURN p.year AS Year, COUNT(DISTINCT a) AS AuthorCount
ORDER BY Year
```

> ...


## Q6
Wie viele Verbindungen/Pfade (ohne Einschränkung von Label und Richtung) gibt es zwischen den Autoren 'Ioanna Tsalouchidou' und 'Charu C. Aggarwal', die eine exakte Länge von 4 Kanten haben?
```
// Wie viele Verbindungen/Pfade (ohne Einschränkung von Label und Richtung) gibt es zwischen den Autoren 'Ioanna Tsalouchidou' und 'Charu C. Aggarwal', die eine exakte Länge von 4 Kanten haben?
MATCH path = (a1:Author {name: 'Ioanna Tsalouchidou'})-[*4]-(a2:Author {name: 'Charu C. Aggarwal'})
RETURN COUNT(DISTINCT path) AS PathCount
```

> 5


## Q7
Auf welchen Venues hat der Autor 'Charu C. Aggarwal' schon Publikationen veröffentlicht? Ausgabe einer duplikatfreien Liste von Venue-Namen sowie einer duplikatfreien Liste von Jahren, indem die Publikationen erfolgt sind.
```
// Auf welchen Venues hat der Autor 'Charu C. Aggarwal' schon Publikationen veröffentlicht? Ausgabe einer duplikatfreien Liste von Venue-Namen sowie einer duplikatfreien Liste von Jahren, indem die Publikationen erfolgt sind.
MATCH (a:Author {name: 'Charu C. Aggarwal'})-[:authorOf]->(p:Publication)-[:publishedIn]->(v:Venue)
RETURN DISTINCT v.name AS venue_name, COLLECT(DISTINCT p.year) AS Years
```

> 4


## Q8
Von welchen der in Q7 genannten Venues war 'Charu C. Aggarwal' als Zweitautor einer entsprechenden Publikation involviert?
```
// Von welchen der in Q7 genannten Venues war 'Charu C. Aggarwal' als Zweitautor einer entsprechenden Publikation involviert?
MATCH (a:Author {name: 'Charu C. Aggarwal'})-[:authorOf {position: 2}]->(p:Publication)-[:publishedIn]->(v:Venue)
RETURN DISTINCT v.name AS venue_name
```

> 2


## Q9
Es gibt Autoren, die bereits zusammen Publikationen veröffentlicht haben (Koautoren). Jeder der Autoren kann nun jeweils auch Publikationen ohne diesen Koautor veröffentlicht haben. Wenn sich diese "unabhängigen" Publikationen zitieren, kann man das als Buddy-Citation bezeichnen. Ermitteln Sie alle Autor-Paare, die mindestens 4 gemeinsame Publikationen veröffentlicht haben und deren unabhängige Publikationen sich mindestens in eine Richtung zitieren (A zitiert B oder B zitiert A).
```
// Es gibt Autoren, die bereits zusammen Publikationen veröffentlicht haben (Koautoren). Jeder der Autoren kann nun jeweils auch Publikationen ohne diesen Koautor veröffentlicht haben. Wenn sich diese "unabhängigen" Publikationen zitieren, kann man das als Buddy-Citation bezeichnen. Ermitteln Sie alle Autor-Paare, die mindestens 4 gemeinsame Publikationen veröffentlicht haben und deren unabhängige Publikationen sich mindestens in eine Richtung zitieren (A zitiert B oder B zitiert A).
MATCH (a1:Author)-[:authorOf]->(p1:Publication)<-[:authorOf]-(a2:Author),
      (a1)-[:authorOf]->(p2:Publication)-[:cites]->(p3:Publication)<-[:authorOf]-(a2)
WHERE a1 < a2
WITH a1, a2, COUNT(DISTINCT p1) AS CommonPublications
WHERE CommonPublications >= 4
RETURN a1.name, a2.name, CommonPublications
```

> 236


## Q10
Ermitteln Sie das am meisten zitierte Papier im Jahr 2016 und geben sie alle Venues der Publikationen aus, die dieses Top-Papier zitieren, sowie die Anzahl der Publikationen pro Venue. Sortieren Sie die Ausgabe absteigend nach der Anzahl.
```
// Ermitteln Sie das am meisten zitierte Papier im Jahr 2016 und geben sie alle Venues der Publikationen aus, die dieses Top-Papier zitieren, sowie die Anzahl der Publikationen pro Venue. Sortieren Sie die Ausgabe absteigend nach der Anzahl. 
CALL {
    MATCH (p:Publication {year: 2016}) <-[x:cites]- (:Publication)
    WITH p, count(x) AS citation_count
    RETURN p AS topPublication
    ORDER BY citation_count DESC
    LIMIT 1
}
MATCH (topPublication) <-[:cites]- (citingPublication:Publication) -[:publishedIn]-> (v:Venue) <-[:publishedIn]- (p:Publication)
RETURN v.name AS venue, count(p) as publication_count
ORDER BY publication_count DESC
```

> 7



# Teil 2

## Q11
Welche 5 Publikationen haben die meisten Autoren? Ausgabe der Publikation (Titel) sowie
der Anzahl der Autoren.
```
// Welche 5 Publikationen haben die meisten Autoren? Ausgabe der Publikation (Titel) sowie der Anzahl der Autoren.
MATCH(p:Publication)<-[:authorOf]-(a)
WITH p, Collect(a) AS authors
RETURN p.title, size(authors) AS numberOfAuthors
ORDER BY numberOfAuthors DESC LIMIT 5
```
| p.title                            | numberOfAuthors |
| ---------------------------------- | --------------- |
| Construction and Analysis of Weighte|
| d Brain Networks from SICE for the St|
| udy of Alzheimer's Disease          | 351             |
| ---------------------------------- | --------------- |
| In-Datacenter Performance Analysis o|
| f a Tensor Processing Unit          | 74              |
| ---------------------------------- | --------------- |
| 11.1 A 512Gb 3b/cell flash memory on|
|  64-word-line-layer BiCS technology | 57              |
| ---------------------------------- | --------------- |
| 11.4 A 512Gb 3b/cell 64-stacked WL 3|
| D V-NAND flash memory               | 44              |
| ---------------------------------- | --------------- |
| In Memoriam: Gunter Menz            | 44              |



## Q12
Welche 5 Autoren haben die meisten Selbstzitierungen? D.h. Ein Autor hat min. zwei Publikationen A und B, wo mindestens eine die andere zitiert. Ausgabe Autor (Name) sowie der Anzahl der Selbstzitierungen
```
// Welche 5 Autoren haben die meisten Selbstzitierungen? D.h. Ein Autor hat min. zwei Publikationen A und B, wo mindestens eine die andere zitiert. Ausgabe Autor (Name) sowie der Anzahl der Selbstzitierungen
MATCH(a1)-[:authorOf]->(p1)-[:cites]->(p2)<-[:authorOf]-(a2)
WHERE a1=a2
WITH a1, COUNT(p1) AS anzahlSelbszitierungen
ORDER BY anzahlSelbszitierungen DESC LIMIT 5
RETURN a1.name, anzahlSelbszitierungen
```
| a1.name             | anzahlSelbszitierungen |
| ------------------- | ---------------------- |
| Haiying Shen        | 19                     |
| Jinwei Liu          | 16                     |
| Raed Ibraheem Hamed | 13                     |
| Mazin Abed Mohammed | 13                     |
| Dheyaa Ahmed Ibrahim| 13                     |



## Q13
Wie viele Publikationen zitieren sich (A zitiert B oder B zitiert A), haben jedoch vollständig unterschiedliche Autoren? Ausgabe der Anzahl von solchen Publikations-Paaren
```
// Wie viele Publikationen zitieren sich (A zitiert B oder B zitiert A), haben jedoch vollständig unterschiedliche Autoren? Ausgabe der Anzahl von solchen Publikations-Paaren
MATCH (a1)-[:authorOf]->(p1)-[:cites]->(p2)<-[:authorOf]-(a2)
WITH p1, collect(a1) AS authors1, collect(a2) AS authors2
WITH p1, [overlapA1 in authors1 WHERE NOT overlapA1 in authors2]+[overlapA2 in authors2 WHERE NOT overlapA2 in authors1] AS overlapAuthors
WHERE isEmpty(overlapAuthors)
RETURN count(p1) AS AnzahlPaare
```
> 321


## Q14.1
Löschen sie die property "n_citation" von allen Publikationen.
```
// Löschen sie die property "n_citation" von allen Publikationen.
MATCH (p:Publication)
REMOVE p.n_citation
```


## Q14.2
Alle Publikationen, die von anderen Publikationen zitiert werden, sollen ihre tatsächliche Anzahl der Zitierungen erhalten. Erstellen Sie eine Query, die für jede Publikation die zitierenden Publikationen zählt und eine neue Property "cite_count" anlegt, die den aggregierten Wert zugewiesen bekommt.
```
// Alle Publikationen, die von anderen Publikationen zitiert werden, sollen ihre tatsächliche Anzahl der Zitierungen erhalten. Erstellen Sie eine Query, die für jede Publikation die zitierenden Publikationen zählt und eine neue Property "cite_count" anlegt, die den aggregierten Wert zugewiesen bekommt.
MATCH (p_cited)<-[:cites]-(p_citing)
WITH p_cited, count(p_citing) as cite_count
SET p_cited.cite_count = cite_count
```
> Set 413175 properties, completed after 2920 ms.


## Q14.3
Lassen Sie sich die Top 10 Publikationen (d.h., die mit den meisten Zitierungen) (Publikations-Id, Titel, cite_count) ausgeben. Was fällt Ihnen auf?
```
// Lassen Sie sich die Top 10 Publikationen (d.h., die mit den meisten Zitierungen) (Publikations-Id, Titel, cite_count) ausgeben. Was fällt Ihnen auf?
MATCH (p:Publication)
WITH p
WHERE p.cite_count <> null
ORDER BY p.cite_count DESC
LIMIT 10
RETURN p.id, p.title, p.cite_count
```
| p.id                                 | p.title | p.cite_count |
| ------------------------------------ | ------- | ------------ |
| e2f7a74a-8430-4463-94ce-fe85dfd309f9 | null    | 587          |
| 153c5014-dc7a-44a8-a93f-5cd27f1193df | null    | 450          |
| c1b6b493-01ef-420f-be44-7bacfe34e846 | null    | 402          |
| f6bd8b64-684d-429a-aab5-8ff3a2c23cd6 | null    | 388          |
| b944f77f-113b-4a02-ae5e-d4a124b8fd5b | null    | 379          |
| dd83785a-dd19-41e3-9b25-ebabbd48d336 | null    | 345          |
| 546cc930-3d5a-4208-a77b-a506f146ab97 | null    | 324          |
| bff1945c-7b01-4b42-b6c4-1e3601c18a6b | null    | 302          |
| c93eac1a-7d9a-48ab-9fb4-389c85bea00e | null    | 281          |
| 051956bb-f64b-4fdb-87f8-3e2868b8b5d8 | null    | 242          |


> Alle Titel sind null


## Q15
Autoren, die zusammen eine Publikation verfasst haben, werden als Koautoren bezeichnet. Erstellen sie zwischen jeden Koautor-Paar eine neue Kante mit dem Label "coAuthor" und der Property mit dem Namen "since", welche das Jahr speichert, an dem beide Autoren die erste/früheste Publikation zusammen verfasst haben. Die Richtung der neuen Kante spielt hierbei keine Rolle. 
```
// Autoren, die zusammen eine Publikation verfasst haben, werden als Koautoren bezeichnet. Erstellen sie zwischen jeden Koautor-Paar eine neue Kante mit dem Label "coAuthor" und der Property mit dem Namen "since", welche das Jahr speichert, an dem beide Autoren die erste/früheste Publikation zusammen verfasst haben. Die Richtung der neuen Kante spielt hierbei keine Rolle. 
MATCH (a1)-[:authorOf]->(p)<-[:authorOf]-(a2)
WITH a1, a2, p
ORDER BY p.year
WITH a1, a2, collect(p)[0] as firstCoPublication
MERGE (a1)-[:coAuthor {since: firstCoPublication.year}]-(a2)
```

