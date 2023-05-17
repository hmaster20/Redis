# Redis через Ansible

## Развертывание Redis

1.Загружаем зависимости (!)

    * Для размещения <current_dir>/playbooks/collections:
    ansible-galaxy collection install -r requirements.yml -p ./playbooks/collections/

    * Для размещения <home_dir>/.ansible:
    ansible-galaxy collection install -r requirements.yml

2.Запускаем на исполнение роль

    * Выполняем команду:
    ansible-playbook playbooks/redis.yml -K

    * slaves
    ansible-playbook playbooks/redis-slave.yml -i hosts.yml -k -K

    * pool sentinel
    ansible-playbook playbooks/redis-sentinel.yml -i hosts.yml

## Удаляем Redis

    * Выполняем команду:
    sudo apt-get remove -y redis* && sudo apt autoremove -y

## Переключение в режим репликации

redis-cli info replication
redis-cli config set protected-mode "no"

### Подключение к старому мастеру

redis-cli replicaof 10.224.128.222 6379

### Подключение к новому мастеру, который сейчас реплика старого мастера :)

redis-cli replicaof 10.224.128.111 6379

redis-cli
127.0.0.1:6379> DBSIZE
127.0.0.1:6379> KEYS *

redis-cli DBSIZE
redis-cli keys *

tail -f /var/log/redis/redis-server.log
tail -f /var/log/redis/redis-sentinel.log

nano /etc/redis/redis-sentinel.conf
redis-server /etc/redis/redis-sentinel.conf --sentinel
redis-cli -p 26379 info sentinel

redis-cli replicaof no one

Когда сервисы запущены, выполняем настройку

1. Мастер узел переводим в режим репликации со старым сервером
    redis-cli replicaof 10.224.128.222 6379

2. Остальные узлы переводим в режим репликации с текущим мастером
    redis-cli replicaof 10.224.128.111 6379

3. Состояние репликации проверяем командой redis-cli info replication
    Также отслеживаем состояние репликации на мастере tail -f /var/log/redis/redis-server.log
    Проверяем наличие ключей
    redis-cli DBSIZE
    redis-cli keys *
    Проверяем какие бегут команды
    redis-cli monitor

4. Отклчаем мастера от старого мастера
    redis-cli replicaof no one

5. Проверяем состояние sentinel командой redis-cli -p 26379 info sentinel
    Состояние должно смениться status=ok
    Также отслеживаем журнал на мастере tail -f /var/log/redis/redis-sentinel.log
    На мастере должно быть изменени +sdown > -sdown, на узлах не важно

6. Создаем ключи на мастере
    redis-cli
    127.0.0.1:6379> set key01 001
   Проверяем их наличие на репликах
    redis-cli
    127.0.0.1:6379> get key01

<https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-20-04-ru>
