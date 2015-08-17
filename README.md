# RNeo4j [![Build Status](https://travis-ci.org/nicolewhite/RNeo4j.svg?branch=master)](https://travis-ci.org/nicolewhite/RNeo4j) [![Coverage Status](https://coveralls.io/repos/nicolewhite/RNeo4j/badge.svg?branch=master&service=github)](https://coveralls.io/github/nicolewhite/RNeo4j?branch=master)

RNeo4j is Neo4j's R driver. It allows you to read and write data from / to Neo4j directly from your R environment.

## Contents

* [Install](#install)
* [Connect](#connect)
* [Nodes](#nodes)
* [Relationships](#relationships)
* [Cypher](#cypher)
* [Shortest Paths](#shortest-paths)
* [Weighted Shortest Paths](#weighted-shortest-paths)
* [Graph Algorithms](#graph-algorithms)
* [Visualizations](#visualizations)
* [Import](#import)
* [Connection Issues](#connection-issues)
* [Contribute](#contribute)

## <a name="#install"></a>Install

### Neo4j

[Download Neo4j](http://neo4j.com/download/) community edition.

#### Windows

If you're on Windows, just download the `.exe` and follow the instructions. You'll get a GUI where you simply press "Start" to start Neo4j.

#### OS X

If you're on OS X, you can download either the `.dmg` or the `.tar.gz`. The `.dmg` is currently offered under the "beta program." This will give you a GUI where you simply press "Start" to start Neo4j. Otherwise, download the `.tar.gz`, unzip, navigate to the directory and execute `./bin/neo4j start`.

#### Linux

If you're on Linux, you have to use the `.tar.gz`. Download the `.tar.gz`, unzip, navigate to the directory and execute `./bin/neo4j start`.

### RNeo4j


```r
install.packages("devtools")
devtools::install_github("nicolewhite/RNeo4j")
library(RNeo4j)
```



## <a name="#connect"></a>Connect


```r
graph = startGraph("http://localhost:7474/db/data/")
```



## <a name="#nodes"></a>Nodes


```r
nicole = createNode(graph, "Person", name="Nicole", age=24)
greta = createNode(graph, "Person", name="Greta", age=24)
kenny = createNode(graph, "Person", name="Kenny", age=27)
shannon = createNode(graph, "Person", name="Shannon", age=23)
```

## <a name="#relationships"></a>Relationships


```r
r1 = createRel(greta, "LIKES", nicole, weight=7)
r2 = createRel(nicole, "LIKES", kenny, weight=1)
r3 = createRel(kenny, "LIKES", shannon, weight=3)
r4 = createRel(nicole, "LIKES", shannon, weight=5)
```

## <a name="#cypher"></a>Cypher

If you're returning tabular results, use `cypher`, which will give you a `data.frame`.


```r
query = "
MATCH (nicole:Person)-[r:LIKES]->(p:Person)
WHERE nicole.name = 'Nicole'
RETURN nicole.name, r.weight, p.name
"

cypher(graph, query)
```

```
##   nicole.name r.weight  p.name
## 1      Nicole        1   Kenny
## 2      Nicole        5 Shannon
```

For anything more complicated, use `cypherToList`, which will give you a `list`.


```r
query = "
MATCH (nicole:Person)-[:LIKES]->(p:Person)
WHERE nicole.name = 'Nicole'
RETURN nicole, COLLECT(p.name) AS friends
"

cypherToList(graph, query)
```

```
## [[1]]
## [[1]]$nicole
## < Node Object > 
## $name
## [1] "Nicole"
## 
## $age
## [1] 24
## 
## 
## [[1]]$friends
## [[1]]$friends[[1]]
## [1] "Kenny"
## 
## [[1]]$friends[[2]]
## [1] "Shannon"
```

Both `cypher` and `cypherToList` accept parameters. These parameters can be passed individually or as a list.


```r
query = "
MATCH (p1:Person)-[r:LIKES]->(p2:Person)
WHERE p1.name = {name1} AND p2.name = {name2}
RETURN p1.name, r.weight, p2.name
"

cypher(graph, query, name1="Nicole", name2="Shannon")
```

```
##   p1.name r.weight p2.name
## 1  Nicole        5 Shannon
```

```r
cypher(graph, query, list(name1="Nicole", name2="Shannon"))
```

```
##   p1.name r.weight p2.name
## 1  Nicole        5 Shannon
```

## <a name="#shortest-paths"></a>Shortest Paths


```r
p = shortestPath(greta, "LIKES", shannon, max_depth=4)
n = nodes(p)
sapply(n, "[[", "name")
```

```
## [1] "Greta"   "Nicole"  "Shannon"
```

## <a name="#weighted-shortest-paths"></a>Weighted Shortest Paths


```r
p = shortestPath(greta, "LIKES", shannon, max_depth=4, cost_property="weight")
n = nodes(p)
sapply(n, "[[", "name")
```

```
## [1] "Greta"   "Nicole"  "Kenny"   "Shannon"
```

```r
p$weight
```

```
## [1] 11
```

## <a name="#cypher"></a>Graph Algorithms


```r
library(igraph)

query = "
MATCH (n)-->(m)
RETURN n.name, m.name
"

edgelist = cypher(graph, query)
ig = graph.data.frame(edgelist, directed=F)

betweenness(ig)
```

```
##  Nicole   Greta   Kenny Shannon 
##       2       0       0       0
```

```r
closeness(ig)
```

```
##    Nicole     Greta     Kenny   Shannon 
## 0.3333333 0.2000000 0.2500000 0.2500000
```

## <a name="#visualizations"></a>Visualizations

### `igraph`


```r
plot(ig)
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 

### `ggnet`


```r
library(network)
library(GGally)

net = network(edgelist)
ggnet(net, label.nodes=TRUE)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png) 

### `visNetwork`

[Read this blog post](http://nicolewhite.github.io/2015/06/18/visualize-your-graph-with-rneo4j-and-visNetwork.html) and [check out this slide deck](http://nicolewhite.github.io/neo4j-presentations/RNeo4j/Visualizations/Visualizations.html#1).

## <a name="#import"></a>Import




```r
library(hflights)
hflights = hflights[sample(nrow(hflights), 1000), ]
row.names(hflights) = NULL

head(hflights)
```

```
##   Year Month DayofMonth DayOfWeek DepTime ArrTime UniqueCarrier FlightNum
## 1 2011     1          4         2    1430    1520            WN       500
## 2 2011     9          5         1    1848    2233            XE      2316
## 3 2011     3          4         5    1121    1253            XE      3152
## 4 2011    11         21         1    2256    2339            OO      5168
## 5 2011    10         31         1    1550    1949            CO      1688
## 6 2011     1         31         1    1302    1357            WN        28
##   TailNum ActualElapsedTime AirTime ArrDelay DepDelay Origin Dest Distance
## 1  N754SW                50      42        5       15    HOU  MSY      303
## 2  N11137               165     146       -1       -2    IAH  RIC     1157
## 3  N14942                92      76      -10       -4    IAH  AMA      519
## 4  N789SK                43      29       -8       -4    IAH  AUS      140
## 5  N11612               179     150       -8       -5    IAH  DTW     1075
## 6  N490WN                55      40       -3        2    HOU  DAL      239
##   TaxiIn TaxiOut Cancelled CancellationCode Diverted
## 1      2       6         0                         0
## 2      7      12         0                         0
## 3      5      11         0                         0
## 4      5       9         0                         0
## 5      9      20         0                         0
## 6      2      13         0                         0
```

```r
addConstraint(graph, "Carrier", "name")
addConstraint(graph, "Airport", "name")

query = "
CREATE (flight:Flight {number: {FlightNum} })
SET flight.year = TOINT({Year}),
    flight.month = TOINT({DayofMonth}),
    flight.day = TOINT({DayOfWeek})

MERGE (carrier:Carrier {name: {UniqueCarrier} })
CREATE (flight)-[:OPERATED_BY]->(carrier)

MERGE (origin:Airport {name: {Origin} })
MERGE (dest:Airport {name: {Dest} })

CREATE (flight)-[o:ORIGIN]->(origin)
CREATE (flight)-[d:DESTINATION]->(dest)

SET o.delay = TOINT({DepDelay}),
    o.taxi_time = TOINT({TaxiOut})

SET d.delay = TOINT({ArrDelay}),
    d.taxi_time = TOINT({TaxiIn})
"

tx = newTransaction(graph)

for(i in 1:nrow(hflights)) {
  row = hflights[i, ]
  
  appendCypher(tx, query,
               FlightNum=row$FlightNum,
               Year=row$Year,
               DayofMonth=row$DayofMonth,
               DayOfWeek=row$DayOfWeek,
               UniqueCarrier=row$UniqueCarrier,
               Origin=row$Origin,
               Dest=row$Dest,
               DepDelay=row$DepDelay,
               TaxiOut=row$TaxiOut,
               ArrDelay=row$ArrDelay,
               TaxiIn=row$TaxiIn)
}

commit(tx)

summary(graph)
```

```
##     This          To    That
## 1 Flight OPERATED_BY Carrier
## 2 Flight      ORIGIN Airport
## 3 Flight DESTINATION Airport
```

## <a name="#connection-issues"></a>Connection Issues

### Couldn't connect to server
```
Error in curl::curl_fetch_memory(url, handle = handle) : 
  Couldn't connect to server
```

Neo4j probably isn't running. Make sure Neo4j is running first. It's also possible you have localhost resolution issues; try connecting to `http://127.0.0.1:7474/db/data/` instead.

### No authorization header supplied

```
Error: client error: (401) Unauthorized
Neo.ClientError.Security.AuthorizationFailed
No authorization header supplied.
```

You have auth enabled on Neo4j and either didn't provide your username and password or they were invalid. You can pass a username and password to `startGraph`.

```
graph = startGraph("http://localhost:7474/db/data/", username="neo4j", password="password")
```

You can also disable auth by editing the following line in `conf/neo4j-server.properties`.

```
# Require (or disable the requirement of) auth to access Neo4j
dbms.security.auth_enabled=false
```

## <a name="#contribute"></a>Contribute

Check out [the contributing doc](https://github.com/nicolewhite/RNeo4j/blob/master/CONTRIBUTING.md) if you'd like to contribute!
