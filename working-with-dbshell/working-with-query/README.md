Starting from MapR 6.0, the platform has an enhanced syntax for performing queries from DB Shell. This page demonstrates the syntax with examples. All the examples in the page use the table with **Yelp** dataset, introduced [here](https://github.com/aravi5/Working-With-MapRDB-JSON/tree/master/working-with-si#2-import-the-json-file-into-mapr-db).


### 1. "find" help for general syntax

```
maprdb root:> help find
Command:                   find
Description:               Retrieves one or more documents from the table.
Options:
  *, --t, --table          Table path. [required]
  --id                     Document Id.
  --fromid                 Document Id to start from (inclusive)
  --toid                   Document Id to stop at (exclusive)
  --limit                  Maximum number of documents to return.
  --withtags, --withTags   Enables/disables printing with extended Type Tags.
  --pretty                 Enables/disables pretty printing of the document.
  --offset                 Skip first n number of rows in the result.
  --orderby                Sort result by the given fields.
  --c, --where             Condition in JSON format
  --f, --fields            Projections in JSON documents
  --q, --query             Query in JSON documents
Examples:
  find /tables/users
  find /tables/users --fromid user001 --toid user00a --limit 32
```

### 2. Explore data by looking at a few documents - Limit

One of the first things you would want to do is to take a peek at the data in the table, say by looking at 2 documents.

```
find /business_table --limit 2 --pretty true
```

**Sample Output:**
```
{
  "_id" : "-0DET7VdEQOJVJ_v6klEug",
  "address" : "3235 York Regional Road 7",
  "attributes" : {
    "Alcohol" : "beer_and_wine",
    "Ambience" : {
      "casual" : true,
      "classy" : false,
      "hipster" : false,
      "intimate" : false,
      "romantic" : false,
      "touristy" : false,
      "trendy" : false,
      "upscale" : false
    },
    "BikeParking" : true,
    "BusinessParking" : {
      "garage" : false,
      "lot" : false,
      "street" : false,
      "valet" : false,
      "validated" : false
    },
    "Caters" : false,
    "GoodForKids" : true,
    "GoodForMeal" : {
      "breakfast" : false,
      "brunch" : false,
      "dessert" : false,
      "dinner" : true,
      "latenight" : false,
      "lunch" : true
    },
    "HasTV" : false,
    "NoiseLevel" : "average",
    "OutdoorSeating" : false,
    "RestaurantsAttire" : "casual",
    "RestaurantsDelivery" : false,
    "RestaurantsGoodForGroups" : true,
    "RestaurantsPriceRange2" : 2,
    "RestaurantsReservations" : true,
    "RestaurantsTableService" : true,
    "RestaurantsTakeOut" : true,
    "WiFi" : "no"
  },
  "business_id" : "-0DET7VdEQOJVJ_v6klEug",
  "categories" : [ "Asian Fusion", "Restaurants" ],
  "city" : "Markham",
  "hours" : {
    "Friday" : "12:00-23:00",
    "Monday" : "12:00-23:00",
    "Saturday" : "12:00-23:00",
    "Sunday" : "12:00-23:00",
    "Thursday" : "12:00-23:00",
    "Tuesday" : "12:00-23:00",
    "Wednesday" : "12:00-23:00"
  },
  "is_open" : 1,
  "latitude" : 43.8483732,
  "longitude" : -79.348727,
  "name" : "Flaming Kitchen",
  "neighborhood" : "Brown's Corners",
  "postal_code" : "L3R 3P9",
  "review_count" : 25,
  "stars" : 3,
  "state" : "ON"
}
{
  "_id" : "-1H-8MO9uEyS9MGmPz3RQw",
  "address" : "Am Bahnhof 1",
  "attributes" : { },
  "business_id" : "-1H-8MO9uEyS9MGmPz3RQw",
  "categories" : [ "Transportation", "Public Transportation", "Hotels & Travel", "Train Stations", "Metro Stations" ],
  "city" : "Stuttgart-Vaihingen",
  "hours" : { },
  "is_open" : 1,
  "latitude" : 48.7264133835,
  "longitude" : 9.1130644839,
  "name" : "S-Bahnhof Stuttgart-Vaihingen",
  "neighborhood" : "",
  "postal_code" : "70563",
  "review_count" : 4,
  "stars" : 2,
  "state" : "BW"
}
2 document(s) found.
```

### 3. List indexes, if any
```
indexlist --table /business_table
```

**Sample Output:**
```
{"indexedFields":[{"sortOrder":"Asc","fieldPathStr":"stars","onMTime":false,"functional":false,"functionName":null,"fieldPathIdx":{"$numberLong":6}},{"sortOrder":"Asc","fieldPathStr":"state","onMTime":false,"functional":false,"functionName":null,"fieldPathIdx":{"$numberLong":2}}],"includedFields":[{"sortOrder":"None","fieldPathStr":"name","onMTime":false,"functional":false,"functionName":null,"fieldPathIdx":{"$numberLong":4}},{"sortOrder":"None","fieldPathStr":"address","onMTime":false,"functional":false,"functionName":null,"fieldPathIdx":{"$numberLong":5}}],"hashed":false,"fullIndex":true,"numHashPartitions":{"$numberLong":0},"unique":false,"external":false,"disabled":false,"primaryTablePath":"/business_table","indexFid":"2049.7462.436424","indexName":"index1","system":"maprdb","cluster":null,"connectionString":null,"missingAndNullOrdering":"MissingAndNullLast"}
```

### 4. Specify query condition and projection

**SQL Equivalent query**
```
select name, address, review_count from /business_table where stars > 3.0 and state = 'NV'
```

**DB Shell query**
```
find --table /business_table --fields name,address,review_count --where {"$and":[{"$gt":{"stars":3.0}},{"$eq":{"state":"NV"}}]}
```
OR
```
find /business_table --query {"$select":["name","address"],"$where":{"$and":[{"$gt":{"stars":3.0}},{"$eq":{"state":"NV"}}]}}
```

> Note: The latter query uses a newly defined **JSON Grammar** for queries. So one could build an entire query as a JSON and provide that as an input to _--q / --query_.

> Generic syntax: {"$select":["field1","field2",..],"$where":{query condition},"$sort":["field1",..],"$limit":number,"$offset":skip_number}
