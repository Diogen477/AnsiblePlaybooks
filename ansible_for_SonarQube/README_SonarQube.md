# Ansible Playbook для установки SonarQube

Автоматизированная установка сервера SonarQube с PostgreSQL и [Community Branch Plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin) для анализа качества кода и поддержки работы с ветками.

## Требования

- **ОС:** Ubuntu 20.04 / 22.04 / 24.04 или Debian 11 / 12
- **Архитектура:** x86_64
- **RAM:** минимум 4 GB (рекомендуется 8 GB)
- **Диск:** минимум 10 GB свободного пространства
- **Ansible:** >= 2.12

## Быстрый старт

```bash
# 1. Перейдите в директорию playbook'а
cd ansible_for_SonarQube

# 2. Установите коллекции Ansible
ansible-galaxy collection install -r requirements.yml

# 3. Задайте надёжный пароль для БД
nano group_vars/all.yml        # Измените db_password

# 4. Запустите
ansible-playbook sonarqube.yml

# 5. Откройте в браузере
#    http://localhost:9000
#    Логин: admin / Пароль: admin
#    ⚠️ Смените пароль при первом входе!
```

## Установленные компоненты

| Роль | Что устанавливается | Версия по умолчанию |
|------|---------------------|---------------------|
| `common` | OpenJDK 17, wget, unzip, python3-psycopg2 | 17 |
| `postgresql` | PostgreSQL, создание БД и пользователя | 16 |
| `sonarqube` | SonarQube, systemd-сервис, настройка JDBC | 25.8.0.112029 |
| `branch_plugin` | Community Branch Plugin, замена webapp, javaagent | 25.8.0 |

## Установка Community Branch Plugin

Роль `branch_plugin` автоматически выполняет все шаги из [официальной инструкции](https://github.com/mc1arke/sonarqube-community-branch-plugin#installation):

1. Скачивает JAR-файл плагина в `extensions/plugins/`
2. Настраивает `sonar.web.javaAdditionalOpts` с параметром `-javaagent:...=web`
3. Настраивает `sonar.ce.javaAdditionalOpts` с параметром `-javaagent:...=ce`
4. Останавливает SonarQube, создаёт бэкап директории `web/`, скачивает `sonarqube-webapp.zip` и заменяет содержимое `web/`
5. Запускает SonarQube и ожидает доступности порта 9000

Бэкап оригинальной директории `web` сохраняется в `web.backup.<дата>` рядом с `web/`.

## Структура проекта

```
ansible_for_SonarQube/
├── sonarqube.yml              # Главный playbook
├── ansible.cfg                # Конфигурация Ansible
├── inventory.ini              # Inventory (localhost)
├── requirements.yml           # Зависимости коллекций
├── .gitignore
├── group_vars/
│   ├── all.yml               # Все переменные: версии, БД, опции
│   └── vault.yml.example     # Пример для Ansible Vault
└── roles/
    ├── common/
    │   ├── tasks/main.yml    # OpenJDK 17, базовые пакеты
    │   └── meta/main.yml
    ├── postgresql/
    │   ├── tasks/main.yml    # PostgreSQL, создание БД, pg_hba.conf
    │   ├── handlers/main.yml # Restart PostgreSQL
    │   └── meta/main.yml
    ├── sonarqube/
    │   ├── tasks/main.yml    # Скачивание, распаковка, systemd, JDBC
    │   ├── handlers/main.yml # Restart SonarQube
    │   └── meta/main.yml
    └── branch_plugin/
        ├── tasks/main.yml    # JAR, javaagent, замена web/, запуск
        ├── handlers/main.yml # Restart SonarQube
        └── meta/main.yml
```

## Настройка

### Переменные (`group_vars/all.yml`)

```yaml
# Версии
sonarqube_version: "25.8.0.112029"
plugin_version: "25.8.0"
postgresql_version: "16"

# SonarQube
sonarqube_dir: "/opt/sonarqube"
sonarqube_user: "sonarqube"

# База данных
db_name: "sonarqube"
db_user: "sonar"
db_password: "StrongPassword123"   # ← Смените!

# Community Branch Plugin — URL архива webapp
webapp_archive_url: "https://github.com/mc1arke/sonarqube-community-branch-plugin/releases/download/{{ plugin_version }}/sonarqube-webapp.zip"
```

### Ansible Vault для пароля БД

Для безопасного хранения пароля:

```bash
# Создайте vault-файл из примера
cp group_vars/vault.yml.example group_vars/vault.yml
nano group_vars/vault.yml          # Задайте vault_db_password

# Зашифруйте
ansible-vault encrypt group_vars/vault.yml

# В all.yml замените строку db_password:
# db_password: "{{ vault_db_password }}"

# Запуск с vault
ansible-playbook sonarqube.yml --ask-vault-pass
```

### Выборочная установка

```bash
# Только базовые пакеты и Java
ansible-playbook sonarqube.yml --tags base

# Только база данных
ansible-playbook sonarqube.yml --tags database

# Только SonarQube (БД уже установлена)
ansible-playbook sonarqube.yml --tags sonarqube

# Без Community Branch Plugin
ansible-playbook sonarqube.yml --skip-tags plugin
```

### Запуск на удалённом хосте

Отредактируйте `inventory.ini`:

```ini
[sonarqube_servers]
sonarqube.example.com ansible_user=ubuntu ansible_become=yes
```

```bash
ansible-playbook sonarqube.yml -i inventory.ini
```

## Безопасность

- **PostgreSQL слушает только localhost** — `listen_addresses = 'localhost'`. Удалённые подключения к БД запрещены.
- **Отдельная запись pg_hba.conf** для пользователя `sonar` — не затрагивает глобальные правила аутентификации PostgreSQL.
- **Пароль БД** рекомендуется хранить в Ansible Vault (пример в `vault.yml.example`).
- **Идемпотентность** — повторные запуски безопасны: SonarQube не перескачивается, JDBC-настройки корректно обновляются.
- **Системный пользователь** — SonarQube работает под `sonarqube`, не под root.

## Проверка установки

```bash
# Статус служб
sudo systemctl status sonarqube
sudo systemctl status postgresql

# Логи SonarQube
sudo journalctl -u sonarqube -f
sudo tail -f /opt/sonarqube/logs/sonar.log

# Проверка подключения к БД
sudo -u postgres psql -d sonarqube -c "\dt"

# Веб-интерфейс
curl -s http://localhost:9000/api/system/status | python3 -m json.tool
```

## Решение проблем

**SonarQube не запускается**
```bash
sudo journalctl -u sonarqube -n 50     # Логи systemd
sudo cat /opt/sonarqube/logs/sonar.log  # Логи приложения
ls -la /opt/sonarqube/                  # Проверка прав
```

**Ошибка подключения к БД**
```bash
sudo systemctl status postgresql
sudo -u postgres psql -c "\du"          # Проверка пользователей
sudo cat /etc/postgresql/16/main/pg_hba.conf | grep sonar
```

**OutOfMemoryError**

Добавьте в `sonar.properties` или измените через playbook:
```properties
sonar.web.javaOpts=-Xmx2048m -Xms512m
sonar.ce.javaOpts=-Xmx2048m -Xms512m
```

**Ошибка плагина после обновления SonarQube**

При обновлении `sonarqube_version` убедитесь, что `plugin_version` совместим. Бэкап директории `web/` сохраняется автоматически — при необходимости можно откатить:
```bash
sudo systemctl stop sonarqube
sudo rm -rf /opt/sonarqube/web
sudo cp -a /opt/sonarqube/web.backup.<дата> /opt/sonarqube/web
sudo systemctl start sonarqube
```

## Интеграция с CI/CD

### GitLab CI

```yaml
sonarqube:
  stage: quality
  script:
    - sonar-scanner
      -Dsonar.projectKey=my-project
      -Dsonar.sources=src
      -Dsonar.host.url=http://sonarqube.example.com:9000
      -Dsonar.token=$SONAR_TOKEN
```

### Создание токена

1. Откройте `http://your-server:9000`
2. Перейдите в **My Account → Security → Generate Tokens**
3. Создайте токен и добавьте его в переменные CI/CD вашего GitLab-проекта как `SONAR_TOKEN`

## Дальнейшие шаги

1. Смените пароль администратора SonarQube при первом входе
2. Создайте проекты в SonarQube UI
3. Сгенерируйте токены для CI/CD
4. Настройте Quality Gates
5. Настройте firewall: `sudo ufw allow 9000/tcp`
6. Для production — настройте SSL через nginx reverse proxy

## Лицензия

MIT

## Автор

[Diogen477](https://github.com/Diogen477)
