# Ansible Playbook для установки SonarQube

Автоматизированная установка SonarQube с PostgreSQL и Community Branch Plugin.

## Требования

- **ОС**: Ubuntu 20.04/22.04/24.04 или Debian 11/12
- **RAM**: минимум 4 GB (рекомендуется 8 GB)
- **Диск**: минимум 10 GB свободного пространства
- **Ansible**: >= 2.12

## Быстрый старт

```bash
# 1. Установите зависимости
ansible-galaxy collection install -r requirements.yml

# 2. Задайте пароль БД
nano group_vars/all.yml   # Измените db_password

# 3. Проверьте синтаксис
ansible-playbook sonarqube.yml --syntax-check

# 4. Запустите
ansible-playbook sonarqube.yml

# 5. Откройте http://localhost:9000
#    Логин: admin / Пароль: admin
#    ⚠️ Смените пароль при первом входе!
```

## Установленные компоненты

| Компонент | Версия (по умолчанию) |
|-----------|----------------------|
| PostgreSQL | 16 |
| SonarQube | 25.8.0.112029 |
| OpenJDK | 17 |
| Community Branch Plugin | 25.8.0 |

## Community Branch Plugin

Плейбук выполняет полную установку плагина согласно [инструкции](https://github.com/mc1arke/sonarqube-community-branch-plugin):

1. **Копирование JAR** в `extensions/plugins/`
2. **Настройка `sonar.web.javaAdditionalOpts`** с `-javaagent:...=web`
3. **Настройка `sonar.ce.javaAdditionalOpts`** с `-javaagent:...=ce`
4. **Замена содержимого `web/`** на `sonarqube-webapp.zip` (с автоматическим бэкапом)
5. **Запуск SonarQube**

## Структура проекта

```
ansible_for_SonarQube/
├── sonarqube.yml              # Главный playbook
├── inventory.ini
├── ansible.cfg
├── requirements.yml
├── group_vars/
│   ├── all.yml               # Переменные
│   └── vault.yml.example     # Пример для Ansible Vault
└── roles/
    ├── common/               # Java, базовые пакеты
    ├── postgresql/           # PostgreSQL (только localhost)
    ├── sonarqube/            # SonarQube сервер
    └── branch_plugin/        # Community Branch Plugin + webapp
```

## Настройка

### Переменные (`group_vars/all.yml`)

```yaml
# Версии
sonarqube_version: "25.8.0.112029"
plugin_version: "25.8.0"
postgresql_version: "16"

# База данных
db_name: "sonarqube"
db_user: "sonar"
db_password: "StrongPassword123"   # Смените!

# Опции
enable_branch_plugin: true
```

### Использование Ansible Vault для пароля БД

```bash
cp group_vars/vault.yml.example group_vars/vault.yml
nano group_vars/vault.yml          # Задайте vault_db_password
ansible-vault encrypt group_vars/vault.yml
```

Затем в `all.yml` замените:
```yaml
db_password: "{{ vault_db_password }}"
```

### Выборочная установка

```bash
# Только БД
ansible-playbook sonarqube.yml --tags database

# Без плагина
ansible-playbook sonarqube.yml --skip-tags plugin

# Только SonarQube
ansible-playbook sonarqube.yml --tags sonarqube
```

## Безопасность

- PostgreSQL слушает **только localhost** (`listen_addresses = 'localhost'`)
- Пароль БД рекомендуется хранить в Ansible Vault
- Смените пароль admin в SonarQube при первом входе
- Рекомендуется настроить firewall и SSL (через nginx reverse proxy)

## Возможные проблемы

**SonarQube не запускается** — проверьте `sudo journalctl -u sonarqube -n 50` и подключение к БД.

**Ошибка плагина после обновления** — убедитесь что версия `plugin_version` совместима с `sonarqube_version`. Бэкап оригинальной директории `web` сохраняется автоматически.

**OutOfMemoryError** — увеличьте JVM параметры в `sonar.properties`.
