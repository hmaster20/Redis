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
