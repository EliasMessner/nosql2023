#Q1
// Ausgabe aller Publikationen, bei denen "Erhard Rahm" einer der Autoren ist.
MATCH (a:Author {name: "Erhard Rahm"})-[:authorOf]->(p:Publication)
RETURN p

> 11


#Q2
// Ausgabe aller Koautoren von "Erhard Rahm".
MATCH (a:Author {name: "Erhard Rahm"})-[:authorOf]->(:Publication)<-[:authorOf]-(coauthor)
RETURN coauthor

> 21


#Q3
// Ausgabe aller Publikationen (Titel), die Publikationen von dem Autor "Wei Wang" zitieren.
MATCH (p1:Publication)<-[:cites]-(:Publication {authors: "Wei Wang"})
RETURN p1.title

> (no changes, no records)


#Q4
// Ausgabe aller Venues, deren Publikationen mindestens eine im Titel das Wort "graph" und das Wort "temporal" (beide case insensitive) enthält. Neben dem duplikatfreien Venue-Namen in der ersten Spalte soll in einer zweiten Spalte auch eine duplikatfreie Liste der Publikations-Titel ausgegeben werden, bei denen beide gesuchten Worte vorkommen.
MATCH (v:Venue)<-[:publishedIn]-(p:Publication)
WHERE toLower(p.title) CONTAINS "graph" AND toLower(p.title) CONTAINS "temporal"
RETURN DISTINCT v.name AS Venue, COLLECT(DISTINCT p.title) AS Publications

> 17


#Q5
// Wie viele Autoren haben pro Jahr bei der Venue mit dem Namen "Lecture Notes in Computer Science" veröffentlicht? Ausgabe aufsteigend sortiert nach Jahr.
MATCH (v:Venue {name: "Lecture Notes in Computer Science"})<-[:publishedIn]-(p:Publication)<-[:authorOf]-(a:Author)
RETURN p.year AS Year, COUNT(DISTINCT a) AS AuthorCount
ORDER BY Year

> ...


#Q6
// Wie viele Verbindungen/Pfade (ohne Einschränkung von Label und Richtung) gibt es zwischen den Autoren 'Ioanna Tsalouchidou' und 'Charu C. Aggarwal', die eine exakte Länge von 4 Kanten haben?
MATCH path = (a1:Author {name: 'Ioanna Tsalouchidou'})-[*4]-(a2:Author {name: 'Charu C. Aggarwal'})
RETURN COUNT(DISTINCT path) AS PathCount

> 5


#Q7
// Auf welchen Venues hat der Autor 'Charu C. Aggarwal' schon Publikationen veröffentlicht? Ausgabe einer duplikatfreien Liste von Venue-Namen sowie einer duplikatfreien Liste von Jahren, indem die Publikationen erfolgt sind.
MATCH (a:Author {name: 'Charu C. Aggarwal'})-[:authorOf]->(p:Publication)-[:publishedIn]->(v:Venue)
RETURN DISTINCT v.name AS venue_name, COLLECT(DISTINCT p.year) AS Years

> 4


#Q8
// Von welchen der in Q7 genannten Venues war 'Charu C. Aggarwal' als Zweitautor einer entsprechenden Publikation involviert?
MATCH (a:Author {name: 'Charu C. Aggarwal'})-[:authorOf {position: 2}]->(p:Publication)-[:publishedIn]->(v:Venue)
RETURN DISTINCT v.name AS venue_name

> 2


#Q9
// Es gibt Autoren, die bereits zusammen Publikationen veröffentlicht haben (Koautoren). Jeder der Autoren kann nun jeweils auch Publikationen ohne diesen Koautor veröffentlicht haben. Wenn sich diese "unabhängigen" Publikationen zitieren, kann man das als Buddy-Citation bezeichnen. Ermitteln Sie alle Autor-Paare, die mindestens 4 gemeinsame Publikationen veröffentlicht haben und deren unabhängige Publikationen sich mindestens in eine Richtung zitieren (A zitiert B oder B zitiert A).
MATCH (a1:Author)-[:authorOf]->(p1:Publication)<-[:authorOf]-(a2:Author),
      (a1)-[:authorOf]->(p2:Publication)-[:cites]->(p3:Publication)<-[:authorOf]-(a2)
WHERE a1 < a2
WITH a1, a2, COUNT(DISTINCT p1) AS CommonPublications
WHERE CommonPublications >= 4
RETURN a1.name, a2.name, CommonPublications

> 236


#Q10
# TODO use subquery and output all venues
// Ermitteln Sie das am meisten zitierte Papier im Jahr 2016 und geben sie alle Venues der Publikationen aus, die dieses Top-Papier zitieren, sowie die Anzahl der Publikationen pro Venue. Sortieren Sie die Ausgabe absteigend nach der Anzahl. Hinweis: mit dem CALL Operator können Subqueries realisiert werden. 
MATCH (p1:Publication {year: 2016})<-[:cites]-(:Publication)-[:publishedIn]->(v:Venue)
WITH p1, v, COUNT(*) AS CitationCount
ORDER BY CitationCount DESC
LIMIT 1
RETURN p1.title AS TopPaper, v.name AS Venue, CitationCount
