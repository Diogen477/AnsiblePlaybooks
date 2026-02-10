# Быстрый старт SonarQube

## За 5 минут к запуску

### 1. Установите Ansible

```bash
sudo apt update
sudo apt install ansible

# Проверка (нужна >= 2.12)
ansible --version
```

### 2. Клонируйте проект

```bash
git clone https://github.com/Diogen477/AnsiblePlaybooks.git
cd AnsiblePlaybooks/ansible_for_SonarQube
```

### 3. Установите зависимости

```bash
ansible-galaxy collection install -r requirements.yml
```

### 4. Задайте пароль БД

```bash
# Вариант 1: Напрямую (для тестовых сред)
nano group_vars/all.yml    # Измените db_password

# Вариант 2: Через Ansible Vault (для production)
cp group_vars/vault.yml.example group_vars/vault.yml
nano group_vars/vault.yml  # Задайте vault_db_password
ansible-vault encrypt group_vars/vault.yml
# В all.yml замените: db_password: "{{ vault_db_password }}"
```

### 5. (Опционально) Настройте версии

```bash
nano group_vars/all.yml

# При необходимости измените:
# sonarqube_version: "25.8.0.112029"
# plugin_version: "25.8.0"
# postgresql_version: "16"
```

### 6. Запустите установку

```bash
# Без vault
ansible-playbook sonarqube.yml

# С vault
ansible-playbook sonarqube.yml --ask-vault-pass
```

### 7. Откройте в браузере

```
http://localhost:9000
```

**Логин:** admin / **Пароль:** admin

### 8. Смените пароль!

При первом входе SonarQube потребует сменить пароль администратора.

## Готово!

Установлены:
- PostgreSQL 16
- SonarQube 25.8
- OpenJDK 17
- Community Branch Plugin (с заменой webapp)

---

## Выборочная установка

```bash
# Только база данных
ansible-playbook sonarqube.yml --tags database

# Без Community Branch Plugin
ansible-playbook sonarqube.yml --skip-tags plugin

# Только SonarQube (БД уже есть)
ansible-playbook sonarqube.yml --tags sonarqube
```

---

## Решение проблем

### SonarQube не запускается

```bash
sudo journalctl -u sonarqube -n 50
sudo systemctl status sonarqube
sudo systemctl restart sonarqube
```

### Не подключается к БД

```bash
sudo systemctl status postgresql
sudo -u postgres psql -d sonarqube -c "\dt"
sudo cat /opt/sonarqube/conf/sonar.properties | grep jdbc
```

### Проверка служб

```bash
sudo systemctl status sonarqube
sudo systemctl status postgresql
sudo journalctl -u sonarqube -f
```

---

## Полезные команды

```bash
# Проверка синтаксиса
ansible-playbook sonarqube.yml --syntax-check

# Dry-run
ansible-playbook sonarqube.yml --check

# Просмотр vault
ansible-vault view group_vars/vault.yml

# Редактирование vault
ansible-vault edit group_vars/vault.yml
```

---

## Дальнейшие шаги

1. Создайте проект в SonarQube UI
2. Сгенерируйте токен: **My Account → Security → Generate Tokens**
3. Настройте анализ в CI/CD:
   ```bash
   sonar-scanner \
     -Dsonar.projectKey=my-project \
     -Dsonar.sources=. \
     -Dsonar.host.url=http://sonarqube.example.com:9000 \
     -Dsonar.token=$SONAR_TOKEN
   ```
4. Настройте Quality Gate
5. Настройте firewall: `sudo ufw allow 9000/tcp`

Подробная документация — в [README.md](README.md).
