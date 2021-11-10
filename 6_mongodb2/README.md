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




### 4. Поронять разные инстансы
