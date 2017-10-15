# Secondary Indexes
MapR-DB JSON supports **_secondary indexes_** (beginning with MapR 6.0) which significantly enhances query performance. 

Secondary indexes are created on fields that are most frequently queried. Indexes are essentially tables where data is ordered on indexed fields. Thus providing efficient access to data by *reducing* **i/o cost** (by reducing amount of data scanned) and **cpu cost** (by avoiding sort).

**Data in MapR-DB JSON Table**

id | city | state | review_count |
--- | --- | --- | --- |
1 | San Jose | CA | 4 |
2 | Portland | OR | 3.5 |
3 | Raleigh | NC | 4.5 |
4 | Buffalo | NY | 3.5 |
5 | Chicago | IL | 4 |

**Indexed Fields**
Consider an index on **review_count** and **state**, where the order of each field is _ascending_. In this case, the data is first ordered by review_count and if more than one row have the same value for review_count, the rows are then ordered by state.

review_count | state | id |
--- | --- | --- |
3.5 | NY | 4 |
3.5 | OR | 2 |
4 | CA | 1 |
4 | IL | 5 |
4.5 | NC | 3 |

> Note: "_ _id_" field is internally stored to identify the document.

**Included Fields**
MapR-DB JSON tables support the concept of _included fields_ in secondary index table. Consider the following query where the predicates are on indexed fields (review_count, state). 

```
select city from /table where review_count > 3 and state < 'NJ' 
```

_city_ is in select but not available in index table. Hence, for each row retrieved from index table, one would have to go to the primary table to retrieve the values for city. This would significantly increase random i/o. Thus index table supports the concept of _included fields_. These are fields that are stored in index table, alongside indexed fields.

With city added as included field, the above index table would look like this.

review_count | state | id | city |
--- | --- | --- | --- |
3.5 | NY | 4 | Buffalo |
3.5 | OR | 2 | Portland |
4 | CA | 1 | San Jose |
4 | IL | 5 | Chicago |
4.5 | NC | 3 | Raleigh |

> Note: Included fields are only included in the index table for easy access. 
> Data in index table is not sorted by included fields.

Now that we are familiar with concepts around to secondary indexes, let's dig in to  _creation_, _management_ and _deletion_ of secondary indexes through _command-line interface (CLI)_. I will walk you through these with help of an example - **[Yelp Dataset](https://www.yelp.com/dataset)**.


**Sample Data:**
--
```
{
	"business_id": "8w00UH_0OeRvOpR0M3Uj5Q",
	"name": "Panera Bread",
	"neighborhood": "",
	"address": "1707 W Warner Rd",
	"city": "Tempe",
	"state": "AZ",
	"postal_code": "85284",
	"latitude": 33.3338866567,
	"longitude": -111.967482327,
	"stars": 5.0,
	"review_count": 4,
	"is_open": 1,
	"attributes": {},
	"categories": ["Food Delivery Services", "Soup", "Food", "Salad", "Restaurants", "Sandwiches"],
	"hours": {
		"Monday": "6:00-22:00",
		"Tuesday": "6:00-22:00",
		"Friday": "6:00-22:00",
		"Wednesday": "6:00-22:00",
		"Thursday": "6:00-22:00",
		"Sunday": "6:00-22:00",
		"Saturday": "6:00-22:00"
	}
}
```

### 1. "Put" sample json file into [MapR-XD](https://mapr.com/products/mapr-xd/)
```
hadoop fs -put business.json /tmp/business.json
```

### 2. "Import" the JSON file into MapR-DB

```
mapr importJSON -src /tmp/business.json -dst /business_table -idfield business_id
```
> For more information regarding _importJSON_ utility, refer to [Loading Documents into JSON Tables](https://maprdocs.mapr.com/52/MapR-DB/JSON_DB/loading_documents_into_json_tables.html?hl=import%2Cjson)

### 3. Create secondary indexes to for faster queries

[_maprcli_](https://maprdocs.mapr.com/52/ReferenceGuide/maprcli-REST-API-Syntax.html) stands for _MapR Command-line Interface_. 

```
maprcli table index add -path /business_table -index index_review_count_state -indexedfields review_count:asc,state:asc -includedfields city
```

> Usage: maprcli table index add -path <primary_table_path> -index <index_name> -indexedfields <index_field1>[:sort_order],<index_field2>[:sort_order]... [-includedfield <included_field1>,...]

> sort_order can be _1 / asc / ASC_ for ascending and _-1 / desc / DESC_ for descending.
