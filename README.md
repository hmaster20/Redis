# Redis

## Обзор

[Redis](https://redis.io) (от англ. remote dictionary server) — система управления базами данных класса NoSQL с [открытым исходным кодом](https://github.com/redis/redis), работающая со структурами данных типа «ключ — значение».

Используется как для организации баз данных, так и для реализации кэшей, брокеров сообщений.

Redis обеспечивает время отклика на уровне долей миллисекунды и позволяет приложениям, работающим в режиме реального времени, выполнять миллионы запросов в секунду. Фактически, производительность Redis ограничивается только пропускной способностью сети или оперативной памяти.

Redis энергозависим и при перезапуске сервера, при желании, может потерять все данные :)
Но в отличие от Memcached, Redis может сохранять кеш на диск. Эта опция включена по умолчанию.
Кстати, здесь приводиться [сравнение Redis и Memcached](https://stackoverflow.com/questions/10558465/memcached-vs-redis)

Redis, из «коробки», включает поддержку кластеров и поставляется с инструментами высокой доступности Sentinel.

## Консольная утилита

Для подключения к Redis можно использовать консольную утилиту `redis-cli`.

```shell
# Установка для Ubuntu выполняется следующим образом:
sudo apt install -yq redis-tools
# Для вывода справки, выполняется команда:
redis-cli --help
```

> По умолчанию, redis-cli пытается подключиться на экземпляр Redis, запущенный локально (`127.0.0.1`), на стандартном порту `6379`.

---

## Демонстрационный стенд

Стенд представляет из себя [`docker-compose.yml`](docker-compose.yml),
содержащий описание запуска набора контейнеров redis и redis-commander.
В процессе запуска, будут запущены 3 экземпляра Redis и 2 экземпляра redis-commander.
Это сделано для демонстрации различных вариантов настройки.

### Запуск

```shell
git clone https://github.com/hmaster20/Redis.git
cd Redis
docker compose up -d
docker compose ps
docker compose logs -f
```

---

## Настройка подключения

```shell
# Подключение к экземпляру Redis, выполняется в следующем формате
redis-cli -h <host> -p <port>
# или так, если установлен пароль для пользователя по умолчанию (default)
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

## Управление учетными записями

### Создаем административную учетную запись

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

```

# Проверяем
# redis-cli -u redis://<username>:<password>@<host>:<port> <command>
redis-cli -u redis://readonly:newpass@192.168.56.74:6381 INFO Server

# Команда ниже, не позволяет выполнить INFO, т.к. исполняется AUTH
# redis-cli -h 192.168.56.74 -p 6380 AUTH readonly newpass
# Но от имени админа можно выполнить команду:
# redis-cli -h <host> -p <port> <command>
redis-cli -h 192.168.56.74 -p 6380 ACL LIST
```

#### - Создание УЗ для Redis внутри Docker

```shell
# Команды в формате
# docker exec -it <container_name> <command>

# default | Через пользователя по умолчанию (админа)
docker exec -it redis-zero redis-cli ACL SETUSER readonly on allkeys +GET +info +select +@read \>newpass
docker exec -it redis-zero redis-cli ACL LIST
docker exec -it redis-zero redis-cli ACL GETUSER readonly
# Создадим тестовый ключ
docker exec -it redis-zero redis-cli SET test_key test_value

# readonly | Через нового пользователя, проверим доступ
docker exec -ti redis-zero redis-cli --no-auth-warning -u redis://readonly:newpass@192.168.56.74:6380 INFO Server
# Проверим значение ключа
docker exec -ti redis-zero redis-cli --no-auth-warning -u redis://readonly:newpass@192.168.56.74:6380 GET test_key
```

---

## Redis-Commander

За основу взято решение [redis-commander](https://github.com/joeferner/redis-commander/pkgs/container/redis-commander).
Сборка образа описана в [Dockerfile](https://github.com/joeferner/redis-commander/blob/master/Dockerfile) проекта.

Redis-Commander является веб-сервером, написанном на node.js,
и представляет собой консоль веб-управления как одиночными экземплярами Redis, так и кластерами (Sentinel).
Само приложение не является ресурсоёмкое, практически не потребляет ЦП и занимает порядка 24 МБ ОЗУ.

## Образы

Возможно использование публичного образа доступного в репозитория:
DockerHub:
    image: rediscommander/redis-commander:latest
GitHub:
    image: ghcr.io/joeferner/redis-commander:latest

## Развертывание

Для тестового (локального) развертывания можно использовать файл docker-compose.yml
Запуск и вывод статистики приложения выполняется командами:

```shell
docker-compose up -d
docker-compose logs -f
```
