# Create and Start Docker Container

```
sudo docker run -p 7474:7474 -p 7687:7687 \
--name neo4j-nosql-task --user="$(id -u):$(id -g)" \
-e apoc.export.file.enabled=true \
-e apoc.import.file.enabled=true \
-e apoc.import.file.use_neo4j_config=true \
-e NEO4J_PLUGINS=\[\"apoc\"\,\"graph-data-science\"\] \
--volume=$HOME/nosql2023/data:/data \
--volume=$HOME/nosql2023/import:/import \
--volume=$HOME/nosql2023/logs:/logs \
--volume=$HOME/nosql2023/plugins:/plugins \
neo4j:5.6.0
```

# Start Docker Container
```
sudo docker start neo4j-nosql-task
```

# Access Database
```
http://localhost:7474/
```

### Initial Credentials
#### User
```
neo4j
```
#### Password
```
neo4j
```

# Stop Docker Container
```
sudo docker stop neo4j-nosql-task
```
