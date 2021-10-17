# Базовые возможности mongodb

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add - && echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list && sudo apt-get update && sudo apt-get install -y mongodb-org

sudo mkdir /home/mongo &&  sudo mkdir /home/mongo/db1 && sudo chmod 777 /home/mongo/db1

mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid
```
about to fork child process, waiting until server is ready for connections.
forked process: 2676
child process started successfully, parent exiting

```bash
ps aux | grep mongo| grep -Ev "grep" 
```
kitsune     2676  3.7  2.1 1506876 85092 ?       Sl   16:40   0:00 mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid

```bash
mongo --port 27001
```
MongoDB shell version v4.4.10

```text
> use kitsne
```
switched to db kitsne

```js
> db.rest.insert([
{"URL":"http://www.just-eat.co.uk/restaurants-cn-chinese-cardiff/menu","address":"228 City Road","address line 2":"Cardiff","name":".CN Chinese","outcode":"CF24","postcode":"3JH","rating":5,"type_of_food":"Chinese"},
...
])
```
```js
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 2548,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
})
```

```js
> db.rest.find({"type_of_food": "Japanese"})
```
```json lines
{ "_id" : ObjectId("616c5c06b65347ba3ff51bd1"), "URL" : "http://www.just-eat.co.uk/restaurants-aisushi-n1/menu", "address" : "335 Caledonian Road", "address line 2" : "London", "name" : "Ai Sushi", "outcode" : "N1", "postcode" : "1DW", "rating" : 5, "type_of_food" : "Japanese" }
...
```

```js
> db.rest.find({"type_of_food": "Japanese"}).sort({"rating": -1}).limit(2)
```
```json lines
{ "_id" : ObjectId("616c5c06b65347ba3ff51be0"), "URL" : "http://www.just-eat.co.uk/restaurants-aji-tn23/menu", "address" : "23 New Rents", "address line 2" : "Ashford", "name" : "Aji - Collection Only", "outcode" : "TN23", "postcode" : "2JJ", "rating" : 6, "type_of_food" : "Japanese" }
{ "_id" : ObjectId("616c5c06b65347ba3ff51bd1"), "URL" : "http://www.just-eat.co.uk/restaurants-aisushi-n1/menu", "address" : "335 Caledonian Road", "address line 2" : "London", "name" : "Ai Sushi", "outcode" : "N1", "postcode" : "1DW", "rating" : 5, "type_of_food" : "Japanese" }
```

```js
> db.rest.updateMany({"address line 2": "London"}, {$set: {"address line 2": "Paris"}})
```
```json
{ "acknowledged" : true, "matchedCount" : 345, "modifiedCount" : 345 }
```

```js
> db.rest.updateMany({"type_of_food": "Chinese", "name": {$regex: "Fish"}}, {$set: {"type_of_food": "Chinese fish"}})
```
```json
{ "acknowledged" : true, "matchedCount" : 4, "modifiedCount" : 4 }
```

```text
> quit()
```
Замерим время выполнения запроса:
```bash
time mongo kitsune --port 27001 --quiet --eval 'db.rest.find({"type_of_food": "Japanese"}).sort({"rating": -1}).limit(2).shellPrint()'
```
```json lines
{ "_id" : ObjectId("616c820f98a5f388317b050b"), "URL" : "http://www.just-eat.co.uk/restaurants-aji-tn23/menu", "address" : "23 New Rents", "address line 2" : "Ashford", "name" : "Aji - Collection Only", "outcode" : "TN23", "postcode" : "2JJ", "rating" : 6, "type_of_food" : "Japanese" }
{ "_id" : ObjectId("616c820f98a5f388317b04fc"), "URL" : "http://www.just-eat.co.uk/restaurants-aisushi-n1/menu", "address" : "335 Caledonian Road", "address line 2" : "London", "name" : "Ai Sushi", "outcode" : "N1", "postcode" : "1DW", "rating" : 5, "type_of_food" : "Japanese" }

real	0m0.071s
user	0m0.052s
sys	0m0.017s
```

Создаем индекс:
```js
> db.rest.createIndex({"type_of_food" : 1})
```
```json
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```

Еще раз замеряем:
```bash
time mongo kitsune --port 27001 --quiet --eval 'db.rest.find({"type_of_food": "Japanese"}).sort({"rating": -1}).limit(2).shellPrint()'
```
```json lines
{ "_id" : ObjectId("616c820f98a5f388317b050b"), "URL" : "http://www.just-eat.co.uk/restaurants-aji-tn23/menu", "address" : "23 New Rents", "address line 2" : "Ashford", "name" : "Aji - Collection Only", "outcode" : "TN23", "postcode" : "2JJ", "rating" : 6, "type_of_food" : "Japanese" }
{ "_id" : ObjectId("616c820f98a5f388317b04fc"), "URL" : "http://www.just-eat.co.uk/restaurants-aisushi-n1/menu", "address" : "335 Caledonian Road", "address line 2" : "London", "name" : "Ai Sushi", "outcode" : "N1", "postcode" : "1DW", "rating" : 5, "type_of_food" : "Japanese" }

real	0m0.071s
user	0m0.050s
sys	0m0.020s
```
Индекс совсем не повлиял на время. Может неоптимально построен или на таком маленькой количестве документов это незаметно?