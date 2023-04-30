# Redis

## Обзор

[Redis](https://redis.io) (от англ. remote dictionary server) — система управления базами данных класса NoSQL с [открытым исходным кодом](https://github.com/redis/redis), работающая со структурами данных типа «ключ — значение».

Используется как для организации баз данных, так и для реализации кэшей, брокеров сообщений.

Redis является очень производительной базой и часто ограничен только пропускной способностью сети или памяти.

Redis энергозависим и при перезапуске сервера, при желании, может потерять все данные :)
Но в отличие от Memcached, Redis может сохранять кеш на диск. Эта опция включена по умолчанию.

Redis, из «коробки», включает поддержку кластеров и поставляется с инструментами высокой доступности Sentinel.
