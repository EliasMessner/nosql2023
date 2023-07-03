# Community Detection

## 1
Erzeugen Sie eine Graph-Projektion mit dem Namen "dblp-community-undirected", welche alle Knoten und Kanten abbildet. Properties müssen nicht enthalten sein. Die Kanten sollten nicht gerichtet (orientation : UNDIRECTED) sein.
```
// Erzeuge Graph-Projektion "dblp-community-undirected"
CALL gds.graph.project(
	'dblp-community-undirected',
	['Author', 'Publication', 'Venue'],
	{
		authorOf: {orientation: 'UNDIRECTED'},
		publishedIn: {orientation: 'UNDIRECTED'},
		cites: {orientation: 'UNDIRECTED'}
	}
);
```
|graphName | nodeCount | relationShipCount |
|----------|-----------|-------------------|
|"dblp-community-undirected"|	593384	|1875822|


## 2 Führen Sie die Louvain Community-Detection aus und lassen sich zunächst im Modus "stats" die Anzahl der gefundenen Communities zurück liefern.
```
// Louvain stats
CALL gds.louvain.stats('dblp-community-undirected')
YIELD communityCount
```
> 3230


## 3 Führen Sie die Louvain Community-Detection aus und ergänzen alle Knoten in der GraphDatenbank mit einer neuen Property "louvain_community", welche die berechnete Community-ID enthalten soll.
```
// Louvain write
CALL gds.louvain.write(
	'dblp-community-undirected',
	{writeProperty: 'louvain_community'}
) YIELD communityCount, modularity, modularities
```
| communityCount |	modularity |	modularities |
|----------------|-----------------|-----------------|
| 3274 |	0.8287694579443555 |	[0.5959932922065571, 0.7707225014665584, 0.8223049242313322, 0.8286136093054173, 0.8287694579443555] |


## 4 Erzeugen Sie eine Cypher-Query, die Ihnen die 10 größten Communities (gemessen nach der Anzahl der Knoten) ausgibt. Die Abfrage erfolgt direkt auf den Graphen, da jeder Knoten jetzt die neue Property "louvain_community" besitzt. Ausgabe von Community-Id und Anzahl der enthaltenen Knoten. 
```
// 10 Größte Communities
MATCH (n)
RETURN n.louvain_community AS communityID, count(n) AS nodeCount
ORDER BY nodeCount DESC LIMIT 10
```
| "communityID" | "nodeCount" |
| ------------- | ----------- |
| 186492        | 46811       |
| 89439         | 37366       |
| 224912        | 29481       |
| 98123         | 27871       |
| 134180        | 24707       |
| 47898         | 18382       |
| 191266        | 14838       |
| 103199        | 14402       |
| 158156        | 13672       |
| 168303        | 13305       |



## 5 Das Ergebnis von der vorherigen Cypher-Query aus 4. kann nun erneut genutzt werden, um herauszufinden, welche Community-Größen am häufigsten vorkommen. Erzeugen Sie die zugehörige Cypher-Query. Ausgabe einer Tabelle mit Community-Größe und Anzahl der Communities mit dieser Größe.
```
// Häufigste Community Größen
CALL {
	MATCH (n)
	RETURN n.louvain_community AS communityID, count(n) AS nodeCount
}
WITH nodeCount, count(communityID) as communityCount
ORDER BY communityCount DESC
RETURN nodeCount, communityCount
```
> 132 rows:
| "nodeCount" | "communityCount" |
| ----------- | ---------------- |
| 3           | 772              |
| 4           | 691              |
| 5           | 436              |
| 2           | 374              |
| 6           | 290              |
| 7           | 195              |
| 8           | 129              |
| 9           | 82               |
| 10          | 45               |
| 11          | 32               |
| 12          | 28               |
| 13          | 18               |
| 14          | 12               |
| ... | ... |
| 1065        | 1                |
| 39          | 1                |
| 298         | 1                |
| 155         | 1                |
| 33          | 1                |
| 61          | 1                |
| 63          | 1                |
| 57          | 1                |
| 32          | 1                |
| 135         | 1                |
| 42          | 1                |
| 67          | 1                |
| 41          | 1                |
| 36          | 1                |


# Betweenness Centrality

## 1 Erzeugen Sie eine Graph-Projektion mit dem Namen "dblp-community-cites", welche nur Publikationen und "cites" Beziehungen abbildet. Properties müssen nicht enthalten sein. Die Kanten sollen gerichtet bleiben.
```
// Erzeuge Graph-Projektion "dblp-community-cites"
CALL gds.graph.project(
	'dblp-community-cites',
	['Publication'],
	['cites']
)
```
|graphName	|nodeCount	|relationshipCount|
|----------|-----------|-------------------|
|"dblp-community-cites"|	413175	|602406|


## 2 Führen Sie den Algorithmus Betweenness Centrality auf den soeben projizierten Graphen im Modus "stream" aus und lassen sich die Titel der Top 10 Publikationen mit dem errechneten Score ausgeben. Je höher der BC-Score, desto mehr kürzeste Pfade gehen durch den Knoten. Kopieren Sie sich den Titel der Publikation mit dem höchsten Score.
> ! takes very long !
```
// Betweenness Centrality - Titel der Top 10 Publikationen
CALL gds.betweenness.stream('dblp-community-cites')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).title as title, score
ORDER BY score DESC LIMIT 10
```
> TODO


## 3 Erstellen Sie nun eine Cypher-Query, die alle benachbarten Publikationen der Top-Publikation ermittelt, die in einer anderen Community existieren (unterschiedliche Community-ID (louvain_community)). Lassen Sie sich die Anzahl der Communities aller Nachbarn ausgeben (COUNT(DISTINCT ...)) um zu sehen, wie viele Papiere aus unterschiedlichen Communities die gefundene Top-Publikation verbindet.
> ! Not tested yet !
```
// Nachbarn der Top-Publikation
MATCH (topPublication {title: ...})-[:cites]-(neighbor)
WHERE topPublication.louvain_community <> neighbor.louvain_community
WITH neighbor
RETURN COUNT(DISTINCT(neighbor.louvain_community)) as numberOfNeighborCommunities
```
> TODO

