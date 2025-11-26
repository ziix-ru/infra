# Памятка: раскат нового VPS

## 1. Обновить inventory

В файле `inventory.ini` прописать IP нового сервера:

```
[vps_bootstrap]
bootstrap_node ansible_host=NEW_IP ansible_user=root hostname=tw01

[vps]
prod_node ansible_host=NEW_IP ansible_user=deploy hostname=tw01
```

## 2. Bootstrap (первичная настройка)

Создает пользователя `deploy`, настраивает SSH и отключает root-доступ.
Запускать **один раз** на чистом сервере:

```
ansible-playbook site-bootstrap.yml
```

## 3. Основной деплой сервисов

Устанавливает Docker, настраивает файрвол, VPN и прокси.

```
ansible-playbook site.yml
```

### Частичное обновление (Теги)
Если нужно обновить только конкретный компонент, используйте теги:

*   `--tags security` — обновить только настройки UFW и ядра (Firewall).
*   `--tags tuning` — обновить тюнинг сети и sysctl.
*   `--tags amneziawg` — обновить/перезапустить контейнер VPN.
*   `--tags traefik` — обновить прокси.

Пример:
```
ansible-playbook site.yml --tags "security,tuning"
```

## 4. Разработка и Линтер

Проект следует стандартам Ansible 2025. Перед коммитом или запуском рекомендуется проверять код линтером.

### Установка зависимостей
На локальной машине должны быть установлены коллекции и линтер:

```
# Установить коллекции (community.docker, ansible.posix и др.)
ansible-galaxy install -r requirements.yml --force

# Установить линтер (через pipx для изоляции)
pipx install ansible-lint

# ИЛИ через apt (проще, но старее)
sudo apt install ansible-lint
```

### Запуск проверки
Линтер проверяет стиль кода и корректность модулей:

```
# Обычный запуск
ansible-lint site.yml

# Если линтер не видит коллекции (ошибка unknown module), указать путь явно:
ANSIBLE_COLLECTIONS_PATH=~/.ansible/collections ansible-lint site.yml
```

## 5. (Опционально) перенести VPN‑данные

Со старого сервера на локальную машину, затем на новый сервер:

```
# 1. Забрать бэкап
scp -r deploy@OLD_IP:/opt/amneziawg/amneziawg-data ./amnezia-backup

# 2. Залить на новый
scp -r ./amnezia-backup deploy@NEW_IP:/opt/amneziawg/

# (Убедитесь, что папка называется amneziawg-data и лежит внутри /opt/amneziawg/)

# 3. Применить права и перезапустить
ansible-playbook site.yml --tags amneziawg
```

## 6. Доступы после переезда

*   **Web UI AmneziaWG:** `http://NEW_IP/`
*   **Traefik Dashboard:** `http://NEW_IP:8080/`
*   **VPN Endpoint:** `NEW_IP:51820` (UDP)
