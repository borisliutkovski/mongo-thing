#### 2.0. Verify that the number of the documents in the restaurants collection is 25359

  ```
  db.restaurants.count()
  ```

  > 25359

#### 2.1.
```
db.restaurants.find({ borough: "Brooklyn" }).count()
```

> 6086

#### 3.1. How many “Chinese” (cuisine) restaurants are in “Queens” (borough)?
```
db.restaurants.find({ "cuisine": "Chinese", "borough": "Queens" }).count()
```

> 728

#### 3.2. What is the _id of the restaurant which has the grade with the highest ever score?

```
db.restaurants.aggregate([
  { $project:{ max: { $max: '$grades.score' } } },
  { $sort: { max: -1} },
  { $limit: 1 },
  { $project: { _id: 1 } }
])
```

> { "_id" : ObjectId("5c17cdeb98aafdfc259c1a40") }

#### 3.3. Add a grade ```{ grade: "A", score: 7, date: ISODate() }``` to every restaurant in “Manhattan” (borough).

```
db.restaurants.updateMany(
  { 'borough': 'Manhattan'},
  { $push: { 'grades': { 'grade': 'A', 'score': 7, date: ISODate() } } }
)
```

> { "acknowledged" : true, "matchedCount" : 10259, "modifiedCount" : 10259 }

#### 3.4. What are the names of the restaurants which have a grade at index 8 with score less then 7? Use projection to include only names without _id

```
db.restaurants.find(
  { "grades.8.score": { $lt: 7 } },
  { "name": 1, "_id": 0 }
)
```

> { "name" : "Silver Krust West Indian Restaurant" }

> { "name" : "Pure Food" }


#### 3.5. What are _id and borough of “Seafood” (cuisine) restaurants which received at least one “B” grade in period from 2014-02-01 to 2014-03-01? Use projection to include only _id and borough.

```
db.restaurants.find(
  {"cuisine":"Seafood","grades": {$elemMatch:{grade: "B", "date": {$gte: new ISODate("2014-02-01"), $lte: new ISODate("2014-03-01")}}},},{"borough":1}
)
```

> { "_id" : ObjectId("5c17cdeb98aafdfc259c4e4b"), "borough" : "Bronx" }

> { "_id" : ObjectId("5c17cdeb98aafdfc259c50c5"), "borough" : "Manhattan" }

#### 4.1. Create an index which will be used by this query and provide proof (from explain() or Compass UI) that the index is indeed used by the winning plan

```
db.restaurants.createIndex({"name":1})
```
```
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```
```
db.restaurants.find({name: "Glorious Food"}).explain()
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "name" : {
                                "$eq" : "Glorious Food"
                        }
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "name" : 1
                                },
                                "indexName" : "name_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "name" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "name" : [
                                                "[\"Glorious Food\", \"Glorious Food\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "serverInfo" : {
                "host" : "",
                "port" : "",
                "version" : "4.0.4",
                "gitVersion" : ""
        },
        "ok" : 1
}
```

#### 4.2. Drop index from task 4.1
```
db.restaurants.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "test.restaurants"
        },
        {
                "v" : 2,
                "key" : {
                        "name" : 1
                },
                "name" : "name_1",
                "ns" : "test.restaurants"
        }
]
```
```
db.restaurants.dropIndex("name_1")
{ "nIndexesWas" : 2, "ok" : 1 }
```

#### 4.3. Create an index to make this query covered and provide proof(from explain() or Compass UI) that it is indeed covered: ```db.restaurants.find({ restaurant_id: "41098650" }, { _id: 0, borough: 1 })```

```
db.restaurants.createIndex({"restaurant_id": 1, "borough": 1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```
```
db.restaurants.find({ restaurant_id: "41098650" }, { _id: 0, borough: 1 }).explain(true)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "restaurant_id" : {
                                "$eq" : "41098650"
                        }
                },
                "winningPlan" : {
                        "stage" : "PROJECTION",
                        "transformBy" : {
                                "_id" : 0,
                                "borough" : 1
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "restaurant_id" : 1,
                                        "borough" : 1
                                },
                                "indexName" : "restaurant_id_1_borough_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "restaurant_id" : [ ],
                                        "borough" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "restaurant_id" : [
                                                "[\"41098650\", \"41098650\"]"
                                        ],
                                        "borough" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 1,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 1,
                "totalDocsExamined" : 0,
                "executionStages" : {
                        "stage" : "PROJECTION",
                        "nReturned" : 1,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 2,
                        "advanced" : 1,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "transformBy" : {
                                "_id" : 0,
                                "borough" : 1
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 1,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 2,
                                "advanced" : 1,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 0,
                                "restoreState" : 0,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "restaurant_id" : 1,
                                        "borough" : 1
                                },
                                "indexName" : "restaurant_id_1_borough_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "restaurant_id" : [ ],
                                        "borough" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "restaurant_id" : [
                                                "[\"41098650\", \"41098650\"]"
                                        ],
                                        "borough" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                },
                                "keysExamined" : 1,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "",
                "port" : "",
                "version" : "4.0.4",
                "gitVersion" : ""
        },
        "ok" : 1
}
```

#### 4.4. Create a partial index on cuisine field which will be used only when filtering on borough equal to “Staten Island”:

```
db.restaurants.createIndex(
  {borough: 1, cuisine: 1},
  {partialFilterExpression: {borough: "Staten Island"}}
)
```
```
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
```

#### 1. db.restaurants.find({ borough: "Staten Island", cuisine: "American" }) – uses index

```
db.restaurants.find({ borough: "Staten Island", cuisine: "American" }).explain(true)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "borough" : {
                                                "$eq" : "Staten Island"
                                        }
                                },
                                {
                                        "cuisine" : {
                                                "$eq" : "American"
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "borough" : 1,
                                        "cuisine" : 1
                                },
                                "indexName" : "borough_1_cuisine_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "borough" : [ ],
                                        "cuisine" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "borough" : [
                                                "[\"Staten Island\", \"Staten Island\"]"
                                        ],
                                        "cuisine" : [
                                                "[\"American\", \"American\"]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 244,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 244,
                "totalDocsExamined" : 244,
                "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 244,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 245,
                        "advanced" : 244,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 1,
                        "restoreState" : 1,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 244,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 244,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 245,
                                "advanced" : 244,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 1,
                                "restoreState" : 1,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "borough" : 1,
                                        "cuisine" : 1
                                },
                                "indexName" : "borough_1_cuisine_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "borough" : [ ],
                                        "cuisine" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "borough" : [
                                                "[\"Staten Island\", \"Staten Island\"]"
                                        ],
                                        "cuisine" : [
                                                "[\"American\", \"American\"]"
                                        ]
                                },
                                "keysExamined" : 244,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "Boris-desktop",
                "port" : 27017,
                "version" : "4.0.4",
                "gitVersion" : "f288a3bdf201007f3693c58e140056adf8b04839"
        },
        "ok" : 1
}
```

#### db.restaurants.find({ borough: "Staten Island", name: "Bagel Land" }) – does not use index
```
db.restaurants.find({ borough: "Staten Island", name: "Bagel Land" }).explain(true)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "borough" : {
                                                "$eq" : "Staten Island"
                                        }
                                },
                                {
                                        "name" : {
                                                "$eq" : "Bagel Land"
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "filter" : {
                                "name" : {
                                        "$eq" : "Bagel Land"
                                }
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "borough" : 1,
                                        "cuisine" : 1
                                },
                                "indexName" : "borough_1_cuisine_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "borough" : [ ],
                                        "cuisine" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "borough" : [
                                                "[\"Staten Island\", \"Staten Island\"]"
                                        ],
                                        "cuisine" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 1,
                "executionTimeMillis" : 1,
                "totalKeysExamined" : 969,
                "totalDocsExamined" : 969,
                "executionStages" : {
                        "stage" : "FETCH",
                        "filter" : {
                                "name" : {
                                        "$eq" : "Bagel Land"
                                }
                        },
                        "nReturned" : 1,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 970,
                        "advanced" : 1,
                        "needTime" : 968,
                        "needYield" : 0,
                        "saveState" : 7,
                        "restoreState" : 7,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 969,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 969,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 970,
                                "advanced" : 969,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 7,
                                "restoreState" : 7,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "borough" : 1,
                                        "cuisine" : 1
                                },
                                "indexName" : "borough_1_cuisine_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "borough" : [ ],
                                        "cuisine" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "borough" : [
                                                "[\"Staten Island\", \"Staten Island\"]"
                                        ],
                                        "cuisine" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                },
                                "keysExamined" : 969,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "Boris-desktop",
                "port" : 27017,
                "version" : "4.0.4",
                "gitVersion" : "f288a3bdf201007f3693c58e140056adf8b04839"
        },
        "ok" : 1
}
```

#### db.restaurants.find({ borough: "Queens", cuisine: "Pizza" }) – does not use index

```
db.restaurants.find({ borough: "Queens", cuisine: "Pizza" }).explain(true)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "borough" : {
                                                "$eq" : "Queens"
                                        }
                                },
                                {
                                        "cuisine" : {
                                                "$eq" : "Pizza"
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "$and" : [
                                        {
                                                "borough" : {
                                                        "$eq" : "Queens"
                                                }
                                        },
                                        {
                                                "cuisine" : {
                                                        "$eq" : "Pizza"
                                                }
                                        }
                                ]
                        },
                        "direction" : "forward"
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 277,
                "executionTimeMillis" : 9,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 25359,
                "executionStages" : {
                        "stage" : "COLLSCAN",
                        "filter" : {
                                "$and" : [
                                        {
                                                "borough" : {
                                                        "$eq" : "Queens"
                                                }
                                        },
                                        {
                                                "cuisine" : {
                                                        "$eq" : "Pizza"
                                                }
                                        }
                                ]
                        },
                        "nReturned" : 277,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 25361,
                        "advanced" : 277,
                        "needTime" : 25083,
                        "needYield" : 0,
                        "saveState" : 198,
                        "restoreState" : 198,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "direction" : "forward",
                        "docsExamined" : 25359
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "Boris-desktop",
                "port" : 27017,
                "version" : "4.0.4",
                "gitVersion" : "f288a3bdf201007f3693c58e140056adf8b04839"
        },
        "ok" : 1
}
```

#### 5. Create an index to make query from task 3.4 covered and provide proof (from explain() or Compass UI) that it is indeed covered

```
db.restaurants.createIndex(
  { "grades.8.score": 1, "name": 1},
  { partialFilterExpression: {"grades.8.score": {$lt: 7}}}
)
```
```
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 2,
        "numIndexesAfter" : 3,
        "ok" : 1
}
```
```
db.restaurants.find({"grades.8.score": {$lt: 7}},{"name": 1, "_id": 0}).explain(true)
{
        "queryPlanner" : {
                "plannerVersion" : 1,
                "namespace" : "test.restaurants",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "grades.8.score" : {
                                "$lt" : 7
                        }
                },
                "winningPlan" : {
                        "stage" : "PROJECTION",
                        "transformBy" : {
                                "name" : 1,
                                "_id" : 0
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "grades.8.score" : 1,
                                        "name" : 1
                                },
                                "indexName" : "grades.8.score_1_name_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "grades.8.score" : [ ],
                                        "name" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "grades.8.score" : [
                                                "[-inf.0, 7.0)"
                                        ],
                                        "name" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [ ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 2,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 2,
                "totalDocsExamined" : 0,
                "executionStages" : {
                        "stage" : "PROJECTION",
                        "nReturned" : 2,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 3,
                        "advanced" : 2,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "transformBy" : {
                                "name" : 1,
                                "_id" : 0
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 2,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 3,
                                "advanced" : 2,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 0,
                                "restoreState" : 0,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "grades.8.score" : 1,
                                        "name" : 1
                                },
                                "indexName" : "grades.8.score_1_name_1",
                                "isMultiKey" : false,
                                "multiKeyPaths" : {
                                        "grades.8.score" : [ ],
                                        "name" : [ ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : true,
                                "indexVersion" : 2,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "grades.8.score" : [
                                                "[-inf.0, 7.0)"
                                        ],
                                        "name" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                },
                                "keysExamined" : 2,
                                "seeks" : 1,
                                "dupsTested" : 0,
                                "dupsDropped" : 0,
                                "seenInvalidated" : 0
                        }
                },
                "allPlansExecution" : [ ]
        },
        "serverInfo" : {
                "host" : "Boris-desktop",
                "port" : 27017,
                "version" : "4.0.4",
                "gitVersion" : "f288a3bdf201007f3693c58e140056adf8b04839"
        },
        "ok" : 1
}
```
