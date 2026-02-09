# Ansible Playbook для установки SonarQube

Автоматизированная установка SonarQube с PostgreSQL и Community Branch Plugin для анализа кода и контроля качества.

## 📋 Содержание

- [Описание](#описание)
- [Требования](#требования)
- [Установленные компоненты](#установленные-компоненты)
- [Быстрый старт](#быстрый-старт)
- [Настройка](#настройка)
- [Использование](#использование)
- [Безопасность](#безопасность)

## 📖 Описание

Этот Ansible playbook устанавливает полнофункциональный SonarQube сервер с:
- PostgreSQL базой данных
- Community Branch Plugin для работы с ветками
- Оптимизированной конфигурацией для production

**Целевое применение:**
- Анализ качества кода
- Статический анализ безопасности
- Отслеживание технического долга
- CI/CD интеграция

## ⚙️ Требования

### Системные требования

- **ОС**: Ubuntu 20.04/22.04 или Debian 11/12
- **Архитектура**: x86_64
- **RAM**: Минимум 4 GB (рекомендуется 8 GB)
- **Диск**: Минимум 10 GB свободного пространства
- **Права**: root или sudo доступ

### Требования к Ansible

```bash
ansible >= 2.12
```

### Дополнительные зависимости

```bash
# Коллекции (установятся из requirements.yml)
community.postgresql >= 3.0.0
community.general >= 5.0.0
```

## 📦 Установленные компоненты

- **PostgreSQL 16** - База данных
- **SonarQube 25.8** - Платформа анализа кода
- **OpenJDK 17** - Java runtime
- **Community Branch Plugin** - Поддержка веток

## 🚀 Быстрый старт

### 1. Клонирование репозитория

```bash
git clone https://github.com/Diogen477/AnsiblePlaybooks.git
cd AnsiblePlaybooks/ansible_for_SonarQube
```

### 2. Настройка credentials

**⚠️ ВАЖНО**: Не используйте пароль по умолчанию!

```bash
# Создайте vault файл из примера
cp group_vars/vault.yml.example group_vars/vault.yml

# Отредактируйте vault.yml - установите СИЛЬНЫЙ пароль БД
nano group_vars/vault.yml

# Зашифруйте файл
ansible-vault encrypt group_vars/vault.yml
# Введите пароль для vault (запомните его!)
```

### 3. Установка зависимостей

```bash
ansible-galaxy collection install -r requirements.yml
```

### 4. Проверка playbook

```bash
# Проверка синтаксиса
ansible-playbook sonarqube.yml --syntax-check

# Dry-run
ansible-playbook sonarqube.yml --check --ask-vault-pass
```

### 5. Запуск установки

```bash
# Полная установка
ansible-playbook sonarqube.yml --ask-vault-pass

# Введите пароль от vault когда запросит
```

### 6. Доступ к SonarQube

После установки откройте в браузере:
```
http://your-server-ip:9000
```

**Логин по умолчанию:**
- Username: `admin`
- Password: `admin`

**⚠️ ВАЖНО**: Смените пароль администратора при первом входе!

## 🔧 Настройка

### Изменение версий

В файле `group_vars/all.yml`:

```yaml
# Версии компонентов
sonarqube_version: "25.8.0.112029"
plugin_version: "25.8.0"
postgresql_version: "16"
```

### Настройка базы данных

```yaml
# В all.yml (публичные настройки)
db_name: "sonarqube"
db_user: "sonar"
db_host: "localhost"  # БЕЗОПАСНО - только локальный доступ
db_port: 5432

# В vault.yml (приватные настройки)
vault_db_password: "ВАШ_СИЛЬНЫЙ_ПАРОЛЬ"
```

### Настройка JVM параметров

```yaml
# Параметры для веб-сервера
sonarqube_web_java_opts: "-Xmx512m -Xms128m"

# Параметры для Compute Engine
sonarqube_ce_java_opts: "-Xmx512m -Xms128m"

# Для production увеличьте:
sonarqube_web_java_opts: "-Xmx2048m -Xms512m"
sonarqube_ce_java_opts: "-Xmx2048m -Xms512m"
```

### Отключение Branch Plugin

```yaml
# В all.yml
enable_branch_plugin: false
```

## 💻 Использование

### Выборочная установка

```bash
# Только база данных
ansible-playbook sonarqube.yml --ask-vault-pass --tags database

# Без плагина
ansible-playbook sonarqube.yml --ask-vault-pass --skip-tags plugin

# Только базовые пакеты
ansible-playbook sonarqube.yml --ask-vault-pass --tags base
```

### Обновление SonarQube

```bash
# 1. Измените версию в all.yml
sonarqube_version: "NEW_VERSION"

# 2. Запустите playbook
ansible-playbook sonarqube.yml --ask-vault-pass --tags sonarqube
```

### Проверка статуса

```bash
# Статус служб
sudo systemctl status sonarqube
sudo systemctl status postgresql

# Логи SonarQube
sudo journalctl -u sonarqube -f

# Логи в файлах
sudo tail -f /opt/sonarqube/logs/sonar.log
```

## 🔐 Безопасность

### ⚠️ Важные предупреждения

1. **Пароль БД через Vault**
   - ✅ Используется Ansible Vault
   - ✅ Пароль НЕ хранится в открытом виде
   - ⚠️ Не комитьте vault.yml без шифрования!

2. **PostgreSQL только на localhost**
   - ✅ `listen_addresses = 'localhost'`
   - ✅ Доступ только с локальной машины
   - Изменяйте только если знаете что делаете

3. **Смените пароль admin**
   - После первого входа в SonarQube
   - Settings → Security → Users → admin

### Рекомендации по безопасности

```bash
# 1. Настройте firewall
sudo ufw allow 9000/tcp  # SonarQube
sudo ufw enable

# 2. Настройте SSL/TLS через reverse proxy (nginx)
# Пример конфигурации nginx:

server {
    listen 443 ssl;
    server_name sonarqube.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# 3. Регулярно обновляйте SonarQube
# 4. Настройте backup базы данных
# 5. Включите аутентификацию в SonarQube UI
```

### Защита credentials с Ansible Vault

```bash
# Просмотр vault файла
ansible-vault view group_vars/vault.yml

# Редактирование vault файла
ansible-vault edit group_vars/vault.yml

# Смена пароля vault
ansible-vault rekey group_vars/vault.yml

# Использование файла с паролем
echo "vault_password" > .vault_pass
chmod 600 .vault_pass
ansible-playbook sonarqube.yml --vault-password-file .vault_pass
```

## 🐛 Возможные проблемы

### SonarQube не запускается

**Проблема**: Service failed

**Решение**:
```bash
# Проверьте логи
sudo journalctl -u sonarqube -n 50

# Проверьте права на директории
ls -la /opt/sonarqube/

# Проверьте подключение к БД
sudo -u postgres psql -d sonarqube -c "\dt"
```

### PostgreSQL не подключается

**Проблема**: Connection refused

**Решение**:
```bash
# Проверьте статус
sudo systemctl status postgresql

# Проверьте конфигурацию
sudo cat /etc/postgresql/16/main/pg_hba.conf

# Перезапустите
sudo systemctl restart postgresql
```

### Недостаточно памяти

**Проблема**: OutOfMemoryError

**Решение**:
```bash
# Увеличьте JVM параметры в all.yml
sonarqube_web_java_opts: "-Xmx2048m -Xms512m"

# Перезапустите SonarQube
sudo systemctl restart sonarqube
```

## 📊 Структура проекта

```
ansible_for_SonarQube/
├── sonarqube.yml              # Главный playbook
├── inventory.ini              # Inventory файл
├── requirements.yml           # Ansible зависимости
├── ansible.cfg               # Конфигурация Ansible
├── .gitignore                # Исключения Git
├── group_vars/
│   ├── all.yml              # Публичные переменные
│   └── vault.yml.example    # Пример vault файла
└── roles/
    ├── common/              # Базовые пакеты
    ├── postgresql/          # База данных
    ├── sonarqube/           # SonarQube сервер
    └── branch_plugin/       # Community Branch Plugin
```

## 📝 Интеграция с CI/CD

### GitLab CI

```yaml
sonarqube:
  stage: quality
  script:
    - sonar-scanner
      -Dsonar.projectKey=my-project
      -Dsonar.sources=.
      -Dsonar.host.url=http://sonarqube.example.com:9000
      -Dsonar.login=$SONAR_TOKEN
```

### Jenkins

```groovy
stage('SonarQube Analysis') {
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }
    steps {
        withSonarQubeEnv('SonarQube') {
            sh "${scannerHome}/bin/sonar-scanner"
        }
    }
}
```

## 🎯 Следующие шаги

После установки:

1. **Настройте проекты** в SonarQube UI
2. **Создайте токены** для CI/CD интеграции
3. **Настройте Quality Gates** для проектов
4. **Установите плагины** для нужных языков
5. **Настройте правила** анализа кода

## 🤝 Участие в разработке

Если вы нашли баг или хотите предложить улучшение:
1. Создайте Issue на GitHub
2. Предложите Pull Request

## 📄 Лицензия

MIT License

## ✍️ Автор

[Diogen477](https://github.com/Diogen477)

---

**Последнее обновление**: Февраль 2026
