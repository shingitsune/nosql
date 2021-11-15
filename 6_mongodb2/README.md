# Кластерные возможности mongodb

## 1. Построить шардированный кластер

Развернем три ноды:

| ID     | IP         |
| ------ | ---------- |
| mongo1 | 10.138.0.5 |
| mongo2 | 10.138.0.6 |
| mongo3 | 10.138.0.7 |

Запустим на каждой ноде по три инстанса монги:
```shell
sudo mkdir -p /home/mongo/{db1,db2,db3} && sudo chmod 777 /home/mongo/{db1,db2,db3}
mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
```

```text
kitsune@mongo1:~$ ps aux | grep mongo | grep -v "grep"
kitsune     2594  2.9  2.1 1571032 87760 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
kitsune     2642  4.9  2.1 1571032 88232 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
kitsune     2690  8.1  2.1 1571032 87396 ?       Sl   19:52   0:01 mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
```
```text
kitsune@mongo2:~$ ps aux | grep mongo | grep -v "grep"
kitsune     2543  2.9  2.1 1571032 87696 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
kitsune     2591  4.4  2.1 1571032 87636 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
kitsune     2639  7.7  2.1 1571032 88132 ?       Sl   19:52   0:01 mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
```
```text
kitsune@mongo3:~$ ps aux | grep mongo | grep -v "grep"
kitsune     2535  3.4  2.1 1571032 86660 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all
kitsune     2583  4.6  2.1 1571032 86800 ?       Sl   19:51   0:01 mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS2 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid --bind_ip_all
kitsune     2632  6.5  2.1 1571032 88244 ?       Sl   19:52   0:00 mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
```

Инициализируем репликасеты:
```shell
mongo --port 27011
```
```js
> rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "10.138.0.5:27011"},{"_id" : 1, host : "10.138.0.6:27011"},{"_id" : 2, host : "10.138.0.7:27011"}]});
```
```json
{ "ok" : 1 }
```

```shell
mongo --port 27012
```
```js
> rs.initiate({"_id" : "RS2", members : [{"_id" : 0, priority : 3, host : "10.138.0.5:27012"},{"_id" : 1, host : "10.138.0.6:27012"},{"_id" : 2, host : "10.138.0.7:27012"}]});
```
```json
{ "ok" : 1 }
```

```shell
mongo --port 27013
```
```js
> rs.initiate({"_id" : "RS3", members : [{"_id" : 0, priority : 3, host : "10.138.0.5:27013"},{"_id" : 1, host : "10.138.0.6:27013"},{"_id" : 2, host : "10.138.0.7:27013"}]});
```
```json
{ "ok" : 1 }
```

Запустим на каждой ноде по инстансу конфигсервера:
```shell
sudo mkdir -p /home/mongo/dbc && sudo chmod 777 /home/mongo/dbc
mongod --configsvr --dbpath /home/mongo/dbc --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid --bind_ip_all
```

```text
kitsune@mongo1:~$ ps aux | grep mongo | grep configsvr | grep -v "grep"
kitsune     2857  2.5  2.2 1637544 89492 ?       Sl   19:57   0:02 mongod --configsvr --dbpath /home/mongo/dbc --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid --bind_ip_all
```
```text
kitsune@mongo2:~$ ps aux | grep mongo | grep configsvr | grep -v "grep"
kitsune     2764  2.0  2.2 1637628 89528 ?       Sl   19:57   0:01 mongod --configsvr --dbpath /home/mongo/dbc --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid --bind_ip_all
```
```text
kitsune@mongo3:~$ ps aux | grep mongo | grep configsvr | grep -v "grep"
kitsune     2751  2.4  2.2 1637628 89584 ?       Sl   19:57   0:01 mongod --configsvr --dbpath /home/mongo/dbc --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc/dbc.log --pidfilepath /home/mongo/dbc/dbc.pid --bind_ip_all
```

Соберем репликасет конфигсервера:
```shell
mongo --port 27001
```
Проверим сначала статус:
```js
> rs.status()

{
	"ok" : 0,
	"errmsg" : "no replset config has been received",
	"code" : 94,
	"codeName" : "NotYetInitialized",
	"$gleStats" : {
		"lastOpTime" : Timestamp(0, 0),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"lastCommittedOpTime" : Timestamp(0, 0)
}
```

Еще не инициализирован. Инициализируем:
```js
> rs.initiate({"_id" : "RScfg", configsvr: true, members : [{"_id" : 0, priority : 3, host : "10.138.0.5:27001"},{"_id" : 1, host : "10.138.0.6:27001"},{"_id" : 2, host : "10.138.0.7:27001"}]});

{
	"ok" : 1,
	"$gleStats" : {
		"lastOpTime" : Timestamp(1636315378, 1),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"lastCommittedOpTime" : Timestamp(0, 0)
}
```

Запустим два экземпляра `mongos`:
```shell
mongos --configdb RScfg/10.138.0.5:27001,10.138.0.6:27001,10.138.0.7:27001 --port 27000 --fork --logpath /home/mongo/dbc/dbs1.log --pidfilepath /home/mongo/dbc/dbs1.pid
mongos --configdb RScfg/10.138.0.5:27001,10.138.0.6:27001,10.138.0.7:27001 --port 27100 --fork --logpath /home/mongo/dbc/dbs2.log --pidfilepath /home/mongo/dbc/dbs2.pid
```

Соберем шарды:
```shell
mongo --port 27000
```
```js
> sh.addShard("RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011")
> sh.addShard("RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012")
> sh.addShard("RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013")
> sh.status()
```
```text
--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("618830fd72cd53257616847e")
  }
  shards:
        {  "_id" : "RS1",  "host" : "RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011",  "state" : 1 }
        {  "_id" : "RS2",  "host" : "RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012",  "state" : 1 }
        {  "_id" : "RS3",  "host" : "RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013",  "state" : 1 }
  active mongoses:
        "4.4.10" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  yes
        Collections with active migrations: 
                config.system.sessions started at Sun Nov 07 2021 20:08:04 GMT+0000 (UTC)
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                11 : Success
  databases:
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	1013
                                RS2	8
                                RS3	3
                        too many chunks to print, use verbose if you want to force print
```

## 2. Балансировка, данные, ключ шардирования

Включаем балансировку для базы:
```js
> use billing
> sh.enableSharding("billing")

{
	"ok" : 1,
	"operationTime" : Timestamp(1636320320, 3),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636320320, 3),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}

> use config
> db.settings.save({ _id:"chunksize", value: 1})
> use billing
```

Нагенерим данные. Это будут документы чеков, состоящие из имени и суммы:
```js
for (var i = 0; i < 300000; i++) { var array = ["Olivia","Liam","Emma","Noah","Amelia","Oliver","Ava","Elijah","Sophia","Lucas","Charlotte","Levi","Isabella","Mason","Mia","Asher","Luna","James","Harper","Ethan","Gianna","Mateo","Evelyn","Leo","Aria","Jack","Ella","Benjamin","Ellie","Aiden","Mila","Logan","Layla","Grayson","Avery","Jackson","Camila","Henry","Lily","Wyatt","Scarlett","Sebastian","Sofia","Carter","Nova","Daniel","Aurora","William","Chloe","Alexander","Riley","Ezra","Nora","Owen","Hazel","Michael","Abigail","Muhammad","Rylee","Julian","Penelope","Hudson","Elena","Luke","Zoey","Samuel","Isla","Jacob","Eleanor","Lincoln","Elizabeth","Gabriel","Madison","Jayden","Willow","Luca","Emilia","Maverick","Violet","David","Emily","Josiah","Eliana","Elias","Stella","Jaxon","Maya","Kai","Paisley","Anthony","Everly","Isaiah","Addison","Eli","Ryleigh","John","Ivy","Joseph","Grace","Matthew"]; db.receipts.insert( { name : array[Math.floor(Math.random() * array.length)], amount: Math.random() * 10000 } ) }
```

Проверим статус:
```js
> db.receipts.stats()

{
	"sharded" : false,
	"primary" : "RS3",
	"capped" : false,
	//...
}
```

Шардирование не активно. Создадим индекс по имени:
```js
> db.receipts.createIndex({name: 1})

{
	"raw" : {
		"RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013" : {
			"createdCollectionAutomatically" : false,
			"numIndexesBefore" : 1,
			"numIndexesAfter" : 2,
			"commitQuorum" : "votingMembers",
			"ok" : 1
		}
	},
	"ok" : 1,
	"operationTime" : Timestamp(1636322053, 7),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636322053, 7),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}

> db.receipts.stats()

{
	"sharded" : false,
	"primary" : "RS3",
	"capped" : false,
	//...
	"indexBuilds" : [ ],
	"totalIndexSize" : 6696960,
	"totalSize" : 16019456,
	"indexSizes" : {
		"_id_" : 4915200,
		"name_1" : 1781760
	},
	"scaleFactor" : 1,
	"ok" : 1,
	//...
}
```

Появился индекс. Включаем шардирование по имени:
```js
> sh.shardCollection("billing.receipts", {name: 1})

{
	"collectionsharded" : "billing.receipts",
	"collectionUUID" : UUID("85796b24-7e2d-4565-8030-2b381208018b"),
	"ok" : 1,
	"operationTime" : Timestamp(1636322206, 6),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636322206, 6),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}

> sh.status()

--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("618830fd72cd53257616847e")
  }
  shards:
        {  "_id" : "RS1",  "host" : "RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011",  "state" : 1 }
        {  "_id" : "RS2",  "host" : "RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012",  "state" : 1 }
        {  "_id" : "RS3",  "host" : "RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013",  "state" : 1 }
  active mongoses:
        "4.4.10" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  yes
        Collections with active migrations: 
                billing.receipts started at Sun Nov 07 2021 21:57:05 GMT+0000 (UTC)
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                685 : Success
  databases:
        {  "_id" : "billing",  "primary" : "RS3",  "partitioned" : true,  "version" : {  "uuid" : UUID("128fd712-3ac4-4c15-ad79-a1a787bb0590"),  "lastMod" : 1 } }
                billing.receipts
                        shard key: { "name" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	1
                                RS2	2
                                RS3	28
                        too many chunks to print, use verbose if you want to force print
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	342
                                RS2	341
                                RS3	341
                        too many chunks to print, use verbose if you want to force print

> sh.balancerCollectionStatus("billing.receipts")

{
	"balancerCompliant" : false,
	"firstComplianceViolation" : "chunksImbalance",
	"ok" : 1,
	"operationTime" : Timestamp(1636322276, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636322276, 4),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

Видно, что чанки нашей базы чеков еще не растеклись, почти все находятся в примари-шарде `RS3`. 
```js
> use admin
> db.runCommand({shardCollection: "billing.receipts", key: {name: 1}})
> sh.status()

--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("618830fd72cd53257616847e")
  }
  shards:
        {  "_id" : "RS1",  "host" : "RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011",  "state" : 1 }
        {  "_id" : "RS2",  "host" : "RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012",  "state" : 1 }
        {  "_id" : "RS3",  "host" : "RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013",  "state" : 1 }
  active mongoses:
        "4.4.10" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                702 : Success
  databases:
        {  "_id" : "billing",  "primary" : "RS3",  "partitioned" : true,  "version" : {  "uuid" : UUID("128fd712-3ac4-4c15-ad79-a1a787bb0590"),  "lastMod" : 1 } }
                billing.receipts
                        shard key: { "name" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	10
                                RS2	10
                                RS3	11
                        too many chunks to print, use verbose if you want to force print
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	342
                                RS2	341
                                RS3	341
                        too many chunks to print, use verbose if you want to force print

> sh.balancerCollectionStatus("billing.receipts")

{
	"balancerCompliant" : true,
	"ok" : 1,
	"operationTime" : Timestamp(1636322675, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636322675, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

Теперь количество чанков в шардах выровнялось, балансировка окончена.

### 3. Настроить аутентификацию и многоролевой доступ

Создадим несколько ролей и пользователей.

```js
> db = db.getSiblingDB("admin")

> db.createRole(
	{
		role: "Admin",
		privileges:[
			{ resource: {anyResource:true}, actions: ["anyAction"]}
		],
		roles:[]
	}
)

> db.createUser({
	user: "DBA",
	pwd: "123456",
	roles: ["Admin"]
})

Successfully added user: { "user" : "DBA", "roles" : [ "Admin" ] }

> db.createRole(
	{
		role: "Manager",
		privileges:[
			{ resource: {db: "", collection: ""}, actions: ["anyAction"]}
		],
		roles:[]
	}
)

> db.createUser({
	user: "Manager1",
	pwd: "123456",
	roles: ["Manager"]
})

Successfully added user: { "user" : "Manager1", "roles" : [ "Manager" ] }

> db.createRole(
	{
		role: "Receipts",
		privileges:[
			{ resource: {db: "billing", collection: "receipts"}, actions: ["anyAction"]}
		],
		roles:[]
	}
)

> db.createUser({
	user: "User",
	pwd: "123456",
	roles: ["Receipts"]
})

Successfully added user: { "user" : "User", "roles" : [ "Receipts" ] }

> db.createRole(
	{
		role: "Backup",
		privileges:[],
		roles:["backup", "restore"]
	}
)

> db.createUser({
	user: "Backuper",
	pwd: "123456",
	roles: ["Backup"]
})

Successfully added user: { "user" : "Backuper", "roles" : [ "Backup" ] }

> use admin

> db.system.roles.find()

{ "_id" : "admin.Admin", "role" : "Admin", "db" : "admin", "privileges" : [ { "resource" : { "anyResource" : true }, "actions" : [ "anyAction" ] } ], "roles" : [ ] }
{ "_id" : "admin.Manager", "role" : "Manager", "db" : "admin", "privileges" : [ { "resource" : { "db" : "", "collection" : "" }, "actions" : [ "anyAction" ] } ], "roles" : [ ] }
{ "_id" : "admin.Receipts", "role" : "Receipts", "db" : "admin", "privileges" : [ { "resource" : { "db" : "billing", "collection" : "receipts" }, "actions" : [ "anyAction" ] } ], "roles" : [ ] }
{ "_id" : "admin.Backup", "role" : "Backup", "db" : "admin", "privileges" : [ ], "roles" : [ { "role" : "backup", "db" : "admin" }, { "role" : "restore", "db" : "admin" } ] }

> db.system.users.find()

{ "_id" : "admin.DBA", "userId" : UUID("4965396e-61bd-439d-9527-753294352cdf"), "user" : "DBA", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "2IEBAqWqT77JRGKXO6U+gg==", "storedKey" : "aMX+GPkBBlrDDkSY86UQxCjXRdM=", "serverKey" : "oXrUwXP3UCLQzC02BG38dgDB1Xk=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "AlHBzoAK0LTEYifsQgGkN9EwPDFQVfNewJTkHw==", "storedKey" : "lblDpceJm9TYDLpOWclbxnHnnaafykEkoj2V6MOrXVk=", "serverKey" : "hDb6juoXkC8rOwMSssshmq0T5IeUDQI82exdY9M4cVg=" } }, "roles" : [ { "role" : "Admin", "db" : "admin" } ] }
{ "_id" : "admin.Manager1", "userId" : UUID("c57e19b2-90a5-4581-8753-2f9f50aac140"), "user" : "Manager1", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "+R4X/IhlX9i34fhBsUXL1A==", "storedKey" : "fkBhre0s9Ndx4S6/M4k/TVzp6cI=", "serverKey" : "dgxsHWN95W67JQRSR9uiX2Serb8=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "fcIiE7gMpupB7CSH/qrFu+DbSo8VMbj3ph5fCg==", "storedKey" : "92UUdjpFnZaBWnFkC0uJwnb6y43NmqMbCc3D7G255kc=", "serverKey" : "irCbLcfyp/L4zRcBlaYlUN2sHoJtlGBfddVOLAGAtMs=" } }, "roles" : [ { "role" : "Manager", "db" : "admin" } ] }
{ "_id" : "admin.User", "userId" : UUID("e116b863-229c-4380-9ff3-b5e174ff9f09"), "user" : "User", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "umt2cVrp3aCgO4nIhoM4Xw==", "storedKey" : "+tDGqrvEWJhrC89tlGOuPFibbwg=", "serverKey" : "9GFmVjDAbhbCshHGKGbKHGANBoc=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "JhatYNJV9sbJQ+S2PT6ASRk8pZrnTXvsjsT+IA==", "storedKey" : "5Ox70XXZHLOo0FAqNbhWaK3ScOrKIHO2I/2K/iUdAr0=", "serverKey" : "kUsN74ej4Rln1rV6zKg6Q8+virCjjjR+t+Dt/Laq5+Y=" } }, "roles" : [ { "role" : "Receipts", "db" : "admin" } ] }
{ "_id" : "admin.Backuper", "userId" : UUID("2f5ac8ae-c48e-4672-9df8-227affd2b278"), "user" : "Backuper", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "93WK6YDpqT80smr4OIUYJw==", "storedKey" : "aUV+W19mewy1K8muX240iXNzNqg=", "serverKey" : "qJ8YuXRuaotJAmK/q/raqKYdVWg=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "ZV8gkxuLCAWPjvUpPs1tUTCjPpMHx77ODqKR5w==", "storedKey" : "OiD2u6NA2vcnrAcsfFW70CCN2kB6L/Lr20gTTKlNTvo=", "serverKey" : "6cr++Xgsp18n0TqWWMKMncPigtOeNy5WyexHSrMqGNI=" } }, "roles" : [ { "role" : "Backup", "db" : "admin" } ] }

> db.shutdownServer()
```

Перезапустим инстанс с включенной аутентификацией.

```shell
mongod --dbpath /home/mongo/db1 --port 27001 --auth --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid
mongo --port 27001
```
```js
> use billing
	
> db.receipts.find()

Error: error: {
	"ok" : 0,
	"errmsg" : "command find requires authentication",
	"code" : 13,
	"codeName" : "Unauthorized"
}
```

Необходима аутентификация.

```shell
mongo --port 27001 -u User -p 123456 --authenticationDatabase "admin"
```
```js
> use billing

> db.receipts.find()

{ "_id" : ObjectId("61917fa693ad63243b0549f9"), "name" : "Benjamin", "amount" : 4649.531705064515 }
{ "_id" : ObjectId("61917fa693ad63243b0549fa"), "name" : "Emilia", "amount" : 9007.51916772755 }
{ "_id" : ObjectId("61917fa693ad63243b0549fb"), "name" : "Eli", "amount" : 3191.4826558199984 }
{ "_id" : ObjectId("61917fa693ad63243b0549fc"), "name" : "Alexander", "amount" : 4994.42386965825 }
{ "_id" : ObjectId("61917fa693ad63243b0549fd"), "name" : "Nova", "amount" : 3014.1906865230417 }
{ "_id" : ObjectId("61917fa693ad63243b0549fe"), "name" : "Zoey", "amount" : 7526.085595938546 }
{ "_id" : ObjectId("61917fa693ad63243b0549ff"), "name" : "Gianna", "amount" : 3783.5716206087923 }
{ "_id" : ObjectId("61917fa693ad63243b054a00"), "name" : "Joseph", "amount" : 5570.12846530342 }
{ "_id" : ObjectId("61917fa693ad63243b054a01"), "name" : "Evelyn", "amount" : 8797.138857276455 }
{ "_id" : ObjectId("61917fa693ad63243b054a02"), "name" : "Sophia", "amount" : 1488.758871715723 }
{ "_id" : ObjectId("61917fa693ad63243b054a03"), "name" : "Ava", "amount" : 4585.211218630209 }
{ "_id" : ObjectId("61917fa693ad63243b054a04"), "name" : "Eli", "amount" : 4903.485552841692 }
{ "_id" : ObjectId("61917fa693ad63243b054a05"), "name" : "Matthew", "amount" : 7015.85532162228 }
{ "_id" : ObjectId("61917fa693ad63243b054a06"), "name" : "Mila", "amount" : 6144.131204570375 }
{ "_id" : ObjectId("61917fa693ad63243b054a07"), "name" : "Riley", "amount" : 775.9185601532814 }
{ "_id" : ObjectId("61917fa693ad63243b054a08"), "name" : "Noah", "amount" : 939.7809010830149 }
{ "_id" : ObjectId("61917fa693ad63243b054a09"), "name" : "Muhammad", "amount" : 8498.62819369472 }
{ "_id" : ObjectId("61917fa693ad63243b054a0a"), "name" : "Stella", "amount" : 358.8194886042828 }
{ "_id" : ObjectId("61917fa693ad63243b054a0b"), "name" : "Lucas", "amount" : 4575.784179231953 }
{ "_id" : ObjectId("61917fa693ad63243b054a0c"), "name" : "Lily", "amount" : 9364.435697882736 }
Type "it" for more

> show databases
billing  0.001GB
```

Аутентификация работает.

### 4. Поронять разные инстансы

Уроним два инстанса примари-шарда `RS3`:

```shell
kill 2690
kill 2639
```

На кластере никак не отразилось, `RS3` по-прежнему примари и ничего не нарушилось.

```js
> sh.status()

--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("618830fd72cd53257616847e")
  }
  shards:
        {  "_id" : "RS1",  "host" : "RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011",  "state" : 1 }
        {  "_id" : "RS2",  "host" : "RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012",  "state" : 1 }
        {  "_id" : "RS3",  "host" : "RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013",  "state" : 1 }
  active mongoses:
        "4.4.10" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  yes
        Failed balancer rounds in last 5 attempts:  5
        Last reported error:  Could not find host matching read preference { mode: "primary" } for set RS3
        Time of Reported error:  Mon Nov 15 2021 06:42:43 GMT+0000 (UTC)
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "billing",  "primary" : "RS3",  "partitioned" : true,  "version" : {  "uuid" : UUID("128fd712-3ac4-4c15-ad79-a1a787bb0590"),  "lastMod" : 1 } }
                billing.receipts
                        shard key: { "name" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	10
                                RS2	10
                                RS3	11
                        too many chunks to print, use verbose if you want to force print
        {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                config.system.sessions
                        shard key: { "_id" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	342
                                RS2	341
                                RS3	341
                        too many chunks to print, use verbose if you want to force print
```

Если при этом посмотреть статус репликасета на оставшемся инсансе, то видим, что у первых двух инстансов изменились значения `health`, `state`, `uptime`, `lastHeartbeatMessage`:

```js
> rs.status()

{
	"set" : "RS3",
	"date" : ISODate("2021-11-15T06:46:05.421Z"),
	"myState" : 2,
	//...
	"members" : [
		{
			"_id" : 0,
			"name" : "10.138.0.5:27013",
			"health" : 0,
			"state" : 8,
			"stateStr" : "(not reachable/healthy)",
			"uptime" : 0,
			//...
			"lastHeartbeat" : ISODate("2021-11-15T06:46:05.039Z"),
			"lastHeartbeatRecv" : ISODate("2021-11-15T06:32:07.370Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "Error connecting to 10.138.0.5:27013 :: caused by :: Connection refused",
			//...
		},
		{
			"_id" : 1,
			"name" : "10.138.0.6:27013",
			"health" : 0,
			"state" : 8,
			"stateStr" : "(not reachable/healthy)",
			"uptime" : 0,
			//...
			"lastHeartbeat" : ISODate("2021-11-15T06:46:05.039Z"),
			"lastHeartbeatRecv" : ISODate("2021-11-15T06:36:44.908Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "Error connecting to 10.138.0.6:27013 :: caused by :: Connection refused",
			//...
		},
		{
			"_id" : 2,
			"name" : "10.138.0.7:27013",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 644038,
			//...
			"lastHeartbeatMessage" : ""
		}
	],
	"ok" : 1,
	//...
}
```

Тем не менее, оставшийся инстанс репликасета находится в состоянии `SECONDARY`, т.е. запись в него не сработает.

Теперь попробуем выключить ноду `mongo3` вместе с последним оставшимся инстансом репликасета `RS3`.

```shell
sudo poweroff
```

Как ни странно, в информации о статусе не вижу изменений:

```js
> sh.status()
--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("618830fd72cd53257616847e")
  }
  shards:
        {  "_id" : "RS1",  "host" : "RS1/10.138.0.5:27011,10.138.0.6:27011,10.138.0.7:27011",  "state" : 1 }
        {  "_id" : "RS2",  "host" : "RS2/10.138.0.5:27012,10.138.0.6:27012,10.138.0.7:27012",  "state" : 1 }
        {  "_id" : "RS3",  "host" : "RS3/10.138.0.5:27013,10.138.0.6:27013,10.138.0.7:27013",  "state" : 1 }
  active mongoses:
        "4.4.10" : 2
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  5
        Last reported error:  Could not find host matching read preference { mode: "primary" } for set RS3
        Time of Reported error:  Mon Nov 15 2021 08:28:44 GMT+0000 (UTC)
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
        {  "_id" : "billing",  "primary" : "RS3",  "partitioned" : true,  "version" : {  "uuid" : UUID("128fd712-3ac4-4c15-ad79-a1a787bb0590"),  "lastMod" : 1 } }
                billing.receipts
                        shard key: { "name" : 1 }
                        unique: false
                        balancing: true
                        chunks:
                                RS1	10
                                RS2	10
                                RS3	11
                        too many chunks to print, use verbose if you want to force print
```

А вот статус коллекции упал в ошибку:

```js
> db.receipts.stats()

{
	"ok" : 0,
	"errmsg" : "Could not find host matching read preference { mode: \"primary\" } for set RS3",
	"code" : 133,
	"codeName" : "FailedToSatisfyReadPreference",
	"operationTime" : Timestamp(1636966408, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1636966431, 3),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

Поднимем инстанс репликасета на ноде `mongo1`:
```shell
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS3 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid --bind_ip_all
```

Статус коллекции прежний. Поднимем инстанс на ноде `mongo2`. 

Успех, репликасет восстановился:

```js
> rs.status()

{
	"set" : "RS3",
	"date" : ISODate("2021-11-15T20:17:42.358Z"),
	"myState" : 1,
	"term" : NumberLong(4),
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"majorityVoteCount" : 2,
	"writeMajorityCount" : 2,
	"votingMembersCount" : 3,
	"writableVotingMembersCount" : 3,
	//...
	"members" : [
		{
			"_id" : 0,
			"name" : "10.138.0.5:27013",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 270,
			//...
		},
		{
			"_id" : 1,
			"name" : "10.138.0.6:27013",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 176,
			//...
		},
		{
			"_id" : 2,
			"name" : "10.138.0.7:27013",
			"health" : 0,
			"state" : 8,
			"stateStr" : "(not reachable/healthy)",
			"uptime" : 0,
			"lastHeartbeatMessage" : "Couldn't get a connection within the time limit of 1000ms",
			//...
		}
	],
	"ok" : 1,
	//...
}
```

Потому что вдвоем они уже смогли выбрать, кто из них будет примари, кто секондари. Причем, что любопытно, сначала инстанс на `mongo2` стал примари после включения, а на `mongo1` — секондари. Но спустя несколько минут они поменялись ролями. Что их сподвигло поменять свои роли? Быть может, шардированный кластер? Проверим статус коллекции.

```js
> db.receipts.stats()

{
	"sharded" : true,
	"capped" : false,
	//...
	"ok" : 1,
	"operationTime" : Timestamp(1637008040, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1637008042, 2),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```

Как видим, состояние коллекции в кластере восстановилось.
