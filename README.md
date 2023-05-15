# Redis

## Обзор

[Redis](https://redis.io) (от англ. remote dictionary server) — система управления базами данных класса NoSQL с [открытым исходным кодом](https://github.com/redis/redis), работающая со структурами данных типа «ключ — значение».

Используется как для организации баз данных, так и для реализации кэшей, брокеров сообщений.

Redis обеспечивает время отклика на уровне долей миллисекунды и позволяет приложениям, работающим в режиме реального времени, выполнять миллионы запросов в секунду. Фактически, производительность Redis ограничивается только пропускной способностью сети или оперативной памяти.

Redis энергозависим и при перезапуске сервера, при желании, может потерять все данные :)
Но в отличие от Memcached, Redis может сохранять кеш на диск. Эта опция включена по умолчанию.
Кстати, здесь приводиться [сравнение Redis и Memcached](https://stackoverflow.com/questions/10558465/memcached-vs-redis)

Redis, из «коробки», включает поддержку кластеров и поставляется с инструментами высокой доступности Sentinel.

## Консольная утилита | redis-cli

Для подключения к Redis можно использовать консольную утилиту `redis-cli`.

```shell
# Установка для Ubuntu выполняется следующим образом:
sudo apt install -yq redis-tools
# Для вывода справки, выполняется команда:
redis-cli --help
```

По умолчанию, `redis-cli` пытается подключиться на экземпляр Redis, запущенный локально `127.0.0.1` на порту `6379`.

### Настройка подключения

```shell
# Подключение к экземпляру Redis, выполняется в следующем формате
redis-cli -h <host> -p <port>
# Если установлен пароль для пользователя по умолчанию (default)
redis-cli -h <host> -p <port> -a <password>

# Например:
redis-cli -h 192.168.56.74 -p 6380
# или
redis-cli -h 192.168.56.74 -p 6381 -a redispass

# Если требуется выполнить подключение к базе под конкретным пользователем,
# то можно выбрать один из следующих вариантов.

# Стандартное подключение, с последующим вводом логина и пароля:
redis-cli -h <host> -p <port>
AUTH <username> <password>
# Например:
redis-cli -h 192.168.56.74 -p 6381
192.168.56.74:6381> AUTH user somepass

# Другой формат подключения
redis-cli -u redis://<username>:<password>@<host>:<port>
# Например:
redis-cli -u redis://user:somepass@192.168.56.74:6381
```

При выполнении команды выше, может появляться предупреждение:

> Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe

Для скрытия предупреждения можно использовать ключ `--no-auth-warning`

---

## Об управлении учетными записями

Управление пользователями выполняется командой [ACL SETUSER](https://redis.io/commands/acl-setuser/)

По умолчанию, в системе создана административная учетная запись пользователя default без пароля

> (`user default on nopass ~* &* +@all"`)

При запуске Redis, можно задать пароль, выполнив команду `redis-server --requirepass <password>` и указав нужный пароль

> (`user default on #ba073dbe27113d518a376fa0d192 ~* &* +@all`)

Для шифрования паролей используется SHA256, что  указано в разделе ["How passwords are stored internally"](https://redis.io/docs/management/security/acl/). Кстати, для создания пароля, можно воспользоваться командой [ACL GENPASS](https://redis.io/commands/acl-genpass/)

---

Для вывода информации о пользователях и паролях через redis-cli, можно использовать команды:

- [`ACL LIST`](https://redis.io/commands/acl-list)
- [`ACL GETUSER <user>`](https://redis.io/commands/acl-getuser), например `ACL GETUSER default`.

---

## Графическая утилита | Redis-Commander

За основу взято решение [redis-commander](https://github.com/joeferner/redis-commander/pkgs/container/redis-commander).
Сборка образа описана в [Dockerfile](https://github.com/joeferner/redis-commander/blob/master/Dockerfile) проекта.

Redis-Commander является веб-сервером, написанном на node.js,
и представляет собой консоль веб-управления как одиночными экземплярами Redis, так и кластерами (Sentinel).
Само приложение не является ресурсоёмкое, практически не потребляет ЦП и занимает порядка 24 МБ ОЗУ.

### Образы

Возможно использование публичного образа доступного в репозитория:
DockerHub:
    image: rediscommander/redis-commander:latest
GitHub:
    image: ghcr.io/joeferner/redis-commander:latest

### Примечание

Redis-Commander имеет возможность подключаться к экземплярам Redis по логину и паролю.
Кроме этого, в целях обеспечения безопасности, он сам может иметь логин и пароль для доступа к UI.
Используя переменные, можно подключиться только к одному экземпляру Redis или кластеру Sentinel.
При использовании переменных Docker, файл конфигурации создается автоматически внутри контейнера.
Но есть обходной вариант. Предварительное создание и использования файлов конфигурации в формате json.
Затем проброс файла внутрь контейнера.
Примеры файлов можно увидеть [здесь](config.json) и [здесь](config_0.json).
Подробнее про использование переменных [здесь](https://github.com/joeferner/redis-commander/blob/master/docs/connections.md) и [здесь](https://github.com/joeferner/redis-commander/blob/master/docs/configuration.md).

> В связи с тем, что `Redis-Commander` не является ресурсоемким, его можно использовать как сайдкар, при деплое совместно с экземпляром Redis.

---

## Демонстрационный стенд

Стенд представляет из себя [`docker-compose.yml`](docker-compose.yml),
содержащий описание контейнеров redis и redis-commander.
По умолчанию, будут запущены 3 экземпляра Redis и 2 экземпляра redis-commander.
[`redis-commander-custom-single`](https://github.com/hmaster20/Redis/blob/2e54fea890b188b6248eb471ffd0de84a8cbbedf/docker-compose.yml#L89) и [`redis-commander-custom-multi`](https://github.com/hmaster20/Redis/blob/2e54fea890b188b6248eb471ffd0de84a8cbbedf/docker-compose.yml#L114) запускаются вручную.
Это сделано для демонстрации различных вариантов настройки.

### Запуск

```shell
git clone https://github.com/hmaster20/Redis.git
cd Redis
docker compose up -d
docker compose ps
docker compose logs -f
```

### Управление учетными записями

#### Создаем административную учетную запись

Если хотим создать новую запись администратора, то сделать это можно так:

```shell
# Создать учетную запись (УЗ) с параметрами доступа и паролем
ACL SETUSER admin on allkeys allchannels +@all >adminpass
# Вывести список доступов
ACL LIST
```

---

### Создаем учетную запись только для чтения

#### - Интерактивное создание

```shell
# Выполняем подключение, используя пароль администратора
redis-cli -h 192.168.56.74 -p 6381 -a redispassone --no-auth-warning

# Создаем учетную запись (УЗ)
ACL SETUSER readonly
# Если необходимо, сбрасываем все правила
ACL SETUSER readonly reset
# Обновляем УЗ, добавляя параметры доступа
ACL SETUSER readonly on +info +select +@read
ACL SETUSER readonly on +GET allkeys >newpass

# Можно сделать тоже самое одной командой (!)
ACL SETUSER readonly on allkeys +GET +info +select +@read >newpass

# Чтобы отобразить правила доступа для всех пользователей
ACL LIST
# Вывести информацию для пользователя readonly
ACL GETUSER readonly

# Создаем тестовый ключ | Если база чистая
SET key_redis value_one
GET key_redis

CTRL + D -> Выходим
```

Как проверить доступ с новой УЗ ?

```shell
redis-cli -h 192.168.56.74 -p 6381
AUTH readonly newpass
GET key_redis
```

#### - Создание УЗ одной командой

```shell
# Создаем
redis-cli -h <ip> -p <port> -a <pass> --no-auth-warning ACL SETUSER <user> on allkeys +GET +info +select +@read \><pass>
redis-cli -h <ip> -p <port> -a <pass> --no-auth-warning ACL LIST
# Например
redis-cli -h 192.168.56.74 -p 6382 -a redispasstwo --no-auth-warning ACL SETUSER readonly on allkeys +GET +info +select +@read \>passonly
redis-cli -h 192.168.56.74 -p 6382 -a redispasstwo --no-auth-warning ACL LIST
# Создаем тестовый ключ (если база чистая)
redis-cli -h 192.168.56.74 -p 6382 -a redispasstwo --no-auth-warning SET key_redis value_two


# Как проверить доступ с новой УЗ ?

# Тест аутентификации
redis-cli -h 192.168.56.74 -p 6382 AUTH readonly passonly
# А так не работает :)
redis-cli -u redis://readonly:passonly@192.168.56.74:6382 GET key_redis
# Зато так работает
docker exec -ti redis-2 redis-cli --no-auth-warning -u redis://readonly:passonly@192.168.56.74:6382 GET key_redis

# > Запускаем контейнеры
docker compose up -d redis-commander-custom-single
docker compose stop redis-commander-custom-single
docker compose up -d redis-commander-custom-multi
#
# Подключаемся через веб браузер и проверяем доступность экземпляров Redis
http://192.168.56.74:8081
http://192.168.56.74:8082
http://192.168.56.74:8083
http://192.168.56.74:8084
В место указанного IP-адреса, нужно указать ваш.
```

##### - Создание УЗ для Redis внутри Docker

```shell
# Команды в формате
# docker exec -it <container_name> <command>

# default | Через пользователя по умолчанию
docker exec -it redis-2 redis-cli ACL SETUSER reader on allkeys +GET +info +select +@read \>newpass
docker exec -it redis-2 redis-cli ACL LIST
docker exec -it redis-2 redis-cli ACL GETUSER reader
# Создадим тестовый ключ
docker exec -it redis-2 redis-cli SET test_key test_value

# reader | Через нового пользователя, проверим доступ
docker exec -ti redis-2 redis-cli --no-auth-warning -u redis://reader:newpass@192.168.56.74:6382 INFO Server
# Проверим значение ключа
docker exec -ti redis-2 redis-cli --no-auth-warning -u redis://reader:newpass@192.168.56.74:6382 GET test_key
```

Для остановки используемой конфигурации [`docker-compose.yml`](docker-compose.yml),
в частности опции [`donotstart`](https://github.com/hmaster20/Redis/blob/2e54fea890b188b6248eb471ffd0de84a8cbbedf/docker-compose.yml#L126), выполняем команду:

```shell
docker compose down --remove-orphans
```
