# Быстрый старт SonarQube

## За 5 минут к запуску

### 1. Установите Ansible
```bash
# Ubuntu/Debian
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

### 4. Настройте пароль БД
```bash
# Создайте vault файл
cp group_vars/vault.yml.example group_vars/vault.yml

# Отредактируйте - ОБЯЗАТЕЛЬНО смените пароль!
nano group_vars/vault.yml

# Зашифруйте
ansible-vault encrypt group_vars/vault.yml
# Введите пароль для vault (запомните!)
```

### 5. (Опционально) Настройте версии
```bash
nano group_vars/all.yml

# Измените при необходимости:
# sonarqube_version: "25.8.0.112029"
# postgresql_version: "16"
```

### 6. Запустите установку
```bash
ansible-playbook sonarqube.yml --ask-vault-pass
```

### 7. Откройте в браузере
```
http://localhost:9000
```

**Логин:** admin  
**Пароль:** admin

### 8. ⚠️ ВАЖНО: Смените пароль!
При первом входе SonarQube попросит сменить пароль.

## Готово! 🎉

---

## Полезные команды

### Просмотр vault файла
```bash
ansible-vault view group_vars/vault.yml
```

### Редактирование vault
```bash
ansible-vault edit group_vars/vault.yml
```

### Проверка синтаксиса
```bash
ansible-playbook sonarqube.yml --syntax-check
```

### Dry-run
```bash
ansible-playbook sonarqube.yml --check --ask-vault-pass
```

### Проверка служб
```bash
# SonarQube
sudo systemctl status sonarqube

# PostgreSQL
sudo systemctl status postgresql

# Логи
sudo journalctl -u sonarqube -f
```

---

## Выборочная установка

### Только база данных
```bash
ansible-playbook sonarqube.yml --ask-vault-pass --tags database
```

### Без Community Branch Plugin
```bash
ansible-playbook sonarqube.yml --ask-vault-pass --skip-tags plugin
```

### Только SonarQube (БД уже установлена)
```bash
ansible-playbook sonarqube.yml --ask-vault-pass --tags sonarqube
```

---

## Troubleshooting

### SonarQube не запускается
```bash
# Проверьте логи
sudo journalctl -u sonarqube -n 50

# Проверьте статус
sudo systemctl status sonarqube

# Перезапустите
sudo systemctl restart sonarqube
```

### Забыли пароль vault
```bash
# Расшифруйте
ansible-vault decrypt group_vars/vault.yml

# Отредактируйте
nano group_vars/vault.yml

# Зашифруйте снова
ansible-vault encrypt group_vars/vault.yml
```

### Не подключается к БД
```bash
# Проверьте PostgreSQL
sudo systemctl status postgresql

# Проверьте подключение
sudo -u postgres psql -d sonarqube -c "\dt"

# Проверьте пароль в конфигурации
sudo cat /opt/sonarqube/conf/sonar.properties | grep jdbc
```

### Нужно больше памяти
```bash
# Отредактируйте all.yml
nano group_vars/all.yml

# Увеличьте JVM параметры:
# sonarqube_web_java_opts: "-Xmx2048m -Xms512m"
# sonarqube_ce_java_opts: "-Xmx2048m -Xms512m"

# Перезапустите
ansible-playbook sonarqube.yml --ask-vault-pass --tags sonarqube
```

---

## Следующие шаги

1. **Создайте проект** в SonarQube UI
2. **Сгенерируйте токен** для CI/CD
3. **Настройте анализатор** в вашем проекте:

```bash
# Пример для GitLab CI
sonar-scanner \
  -Dsonar.projectKey=my-project \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://sonarqube.example.com:9000 \
  -Dsonar.login=$SONAR_TOKEN
```

4. **Настройте Quality Gate**
5. **Интегрируйте с CI/CD**

---

## Безопасность

### После установки обязательно:
```bash
# 1. Смените пароль admin в SonarQube

# 2. Настройте firewall
sudo ufw allow 9000/tcp
sudo ufw enable

# 3. (Опционально) Настройте SSL через nginx
```

---

## Дополнительная информация

- **README.md** - Полная документация
- **CHANGES.md** - Список всех изменений
- Создайте Issue на GitHub если нашли проблему

**Успешной работы с SonarQube!** 🚀
