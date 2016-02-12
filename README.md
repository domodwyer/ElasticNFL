# ElasticNFL
Cram every NFL play of the 2014 season into Elasticsearch - for fun and profit.

##Instructions
Edit the `ElasticNFL.php` script to reflect your endpoint settings (namely `$endpoint`).

Each games goes into the _nfl_ index, under a unique per-game type (the NFL's game ID), with the play number as the document ID - this means the third play of the 2015 Super Bowl is accessed as: `/nfl/2015020100/3`.

##Example Queries
Provided are some rough-ish example queries... I wouldn't go around shouting about these stats like they're gospal, but they're good enough for learning purposes.


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

###Yards per drive, per team, ordered by total yards in the season:
```
{    
  "aggs": {
    "quarter": {
      "terms": { "field": "qtr", "size": 100, "order": {"yards.sum" : "desc"} },
        "aggs": {
          "yards": {
            "stats": {"field": "yards"}
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
						"stats": {"field": "yards"}
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
             "stats": {"field": "yards"}
         }
       }
     }
   }
  }
}
```


##Notes
Possibly not a good idea to cram everything into one index with a type-per-game if you've got the liberty of an entire cluster of nodes to play with.
