# ElasticNFL
Cram every NFL play of the 2014 season into Elasticsearch - for fun and profit.

Tested on Elasticsearch 2.3.2.

##Installation
I'm on OS X, Elasticsearch and all of it's dependencies can be installed using [brew](http://brew.sh/): `brew install elasticsearch`

I'm using one machine as the Elasticsearch host, and my dev/client on another. Elasticsearch likes RAM, on the server run `ES_HEAP_SIZE=12g elasticsearch -Dnetwork.host=0.0.0.0` to get started.

##Instructions
Edit the `ElasticNFL.php` script to reflect your endpoint settings (namely `$endpoint`).

Each games goes into the _nfl_ index, under a unique per-game type (the NFL's game ID), with the play number as the document ID - this means the third play of the Super Bowl is accessed as: `/nfl/2015020100/3`.

##Example Queries
Provided are some rough-ish example queries... I wouldn't go around shouting about these stats like they're gospel, but they're good enough for learning purposes. They're all requested via `POST http://1.2.3.4:9200/nfl/_search` with the provided JSON as the request body.


###Average yards to go per down:
```
{  
  "aggs": {
    "down": {
      "terms": { "field": "down" },
      "aggs" : {
        "yardstogo": {
          avg: { "field": "ydstogo"}
        }
      }
    }
  }
}
```

###Yards per team, ordered by total yards in the season:
```
{    
  "aggs": {
    "teams": {
      "terms": { "field": "posteam", "size": 100, "order": {"yards.sum" : "desc"} },
        "aggs": {
          "yards": {
            "stats": {"field": "players.yards"}
        }
      }
    }
  }
}
```

###Tom Brady's passing yard statistics, broken down by quarter, to anyone except Amendola:
```
{
  "query": {
 		"bool": {
 			"must": [
 				{"match": {"players.playerName": "T.Brady"}},
 				{"match": {"desc": "pass"}}
 			],
			"must_not": [
				{"match": {"players.playerName": "D.Amendola"}}
			]
 		}
 	},
	"aggs": {
		"quarter": {
			"terms": { "field": "qtr", "size": 5, "order": {"yards.sum" : "desc"} },
				"aggs": {
					"yards": {
						"stats": {"field": "players.yards"}
					}
				}
			}
		}
	}
}
```

###NE top recievers who didn't drop the ball:
```
{
  "query": {
 		"bool": {
 			"must": [
 				{"match": {"players.playerName": "T.Brady"}},
 				{"match": {"desc": "pass"}}
 			],
			"must_not": [
				{"match": {"desc": "incomplete"}}
			]
 		}
  },
  "aggs": {
     "quarter": {
       "terms": { "field": "players.playerName", "size": 5, "order": {"yards.count" : "desc"} },
         "aggs": {
           "yards": {
             "stats": {"field": "players.yards"}
         }
       }
     }
   }
  }
}
```


##Notes
Possibly not a good idea to cram everything into one index with a type-per-game if you've got the liberty of an entire cluster of nodes to play with.
