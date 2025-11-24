# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```
- Подключитесь к серверу конфигурации и сделайте инициализацию:

```
docker exec -it configSrv-1 mongosh --port 27017


> rs.initiate({
  _id: "config_server",
  configsvr: true,
  members: [
    { _id: 0, host: "configSrv-1:27017" },
    { _id: 1, host: "configSrv-2:27018" },
    { _id: 2, host: "configSrv-3:27019" }
  ]
})

> exit(); 
```

- Инцициализируйте шарды:
```
docker exec -it shard1-1 mongosh --port 27021

> rs.initiate(
    {
      _id : "shard1rs",
      members: [
        { _id : 0, host : "shard1-1:27021" },
        { _id : 1, host : "shard1-2:27022" },
	{ _id : 2, host : "shard1-3:27023" }
      ]
    }
);
> exit();

docker exec -it shard2-1 mongosh --port 27024

> rs.initiate(
    {
      _id : "shard2rs",
      members: [
        { _id : 0, host : "shard2-1:27024" },
        { _id : 1, host : "shard2-2:27025" },
	{ _id : 2, host : "shard2-3:27026" }
      ]
    }
);
> exit();
```

- Инцициализируйте роутер и наполните его тестовыми данными:
```
docker exec -it mongos_router_repl mongosh --port 27020

> sh.addShard("shard1rs/shard1-1:27021,shard1-2:27022,shard1-3:27023")
> sh.addShard("shard2rs/shard2-1:27024,shard2-2:27025,shard2-3:27026")

> sh.enableSharding("somedb");
> sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

> use somedb

> for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

> db.helloDoc.countDocuments() 
> exit();
```

- Сделайте проверку на шардах:
```
 docker exec -it shard1-1 mongosh --port 27021
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```
- Сделайте проверку на втором шарде:
```
docker exec -it shard2-1 mongosh --port 27024
 > use somedb;
 > db.helloDoc.countDocuments();
 > exit();
```

## Как проверить

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8082

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8082

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8082/docs