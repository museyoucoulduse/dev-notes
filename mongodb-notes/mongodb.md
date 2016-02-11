MongoDB
=======

## Installation

I have installed MongoDB using docs from mongodb.org
on my local machine and setup in `/etc/mongod.conf`
`bindIP` to `0.0.0.0` what results in access from other machines

Using apt on Debian/Ubuntu

```shell
sudo apt-get install mongodb
```

## Usage

### Python

Install pymongo using `pip` or `conda`

```python
import pymongo
```

or

```python
from pymongo import MongoClient
```

then

```python
client = MongoClient('mongodb://localhost:27017/')
db = client.database # where client property is database of your choice
```

You have full access to creating **collections** where **documents** are stored...

```python
db.myCollection.insert_many(jsonObjectArray)
db.myCollection.insert_one(jsonObject)
```

Querying database

```python
db.forexGlossary.find({"$or":[{"description": {"$regex": "Euro"}},
    {"description": {"$regex": "euro"}}]})
```

In `Python`, `PyMongo` and `Mongoose` in `Node.js`:

```
db.users.find({'name': {'$regex': 'sometext'}})
```

MongoDB Shell, find all greater than 0 `total_pnl`, return only `ticker, tf, algo, benchmark, total_pnl and max_drawdown` fields and sort by `benchmark`.

```javascript
db.oandaTest.find({"total_pnl": {$gt: 0}}, {"ticker": 1, "tf": 1, "algo": 1, "benchmark":1, "total_pnl": 1, "max_drawdown": 1}).sort({"benchmark": -1})
```

Update/replace a document:

```javascript
db.inventory.update(
   { item: "BE10" }, // document to be updated
   {
     item: "BE05",
     stock: [ { size: "S", qty: 20 }, { size: "M", qty: 5 } ],
     category: "apparel"
   }
)
```

Pymongo docs on collections [are here](http://api.mongodb.org/python/current/api/pymongo/collection.html)