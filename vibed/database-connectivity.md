# Соединение с базами данных

С vibe.d очень просто производить доступ
к базам данных в бэкенд-сервисах. Поддержка
**MongoDB** и **Redis** встроена в vibe.d,
адаптеры для других баз данных можно найти на
[code.dlang.org](https://code.dlang.org).

### MongoDB

Доступ к MongoDB смоделирован на основе класса
[`MongoClient`](http://vibed.org/api/vibe.db.mongo.client/MongoClient).
Эта реализация не имеет внешних зависимостей,
и построена на основе асинхронных сокетов vibe.d:
блокировки не происходит, если в самом соединении
происходят задержки.

    auto client = connectMongoDB("127.0.0.1");
    auto users = client.getCollection("users");
    users.insert(Bson("peter"));

### Redis

Поддержка Redis так же реализована на основе
сокетов vibe.d и не имеет внешних зависимостей.
Основным классом в данной реализации является
[`RedisDatabase`](http://vibed.org/api/vibe.db.redis.redis/RedisDatabase),
позволяющий передавать команды Redis-серверу.
Дополнительно предоставляются удобные оболочки,
такие как [`RedisList`](http://vibed.org/api/vibe.db.redis.types/RedisList),
предоставляющий прозрачный доступ к списку,
хранящемуся в Redis.

### MySQL

Поддержку MySQL без дополнительных зависимостей
от официальной библиотеки MySQL можно добавить,
используя проект [mysql-native](http://code.dlang.org/packages/mysql-native).
Он так же поддерживает неблокирующие сокеты
vibe.d.

### Postgresql

Полнофункциональный клиент Postgresql реализован
во внешнем модуле [dpq2](http://code.dlang.org/packages/dpq2),
построенном на основе официальной библиотеки
*libpq*. Он использует систему событий vibe.d
для организации асинхронной работы.

Альтернативный вариант поддержки Postgresql -
[ddb](http://code.dlang.org/packages/ddb),
реализующий клиент Postgresql на основе сокетов
vibe.d без дополнительных внешних зависимостей.


