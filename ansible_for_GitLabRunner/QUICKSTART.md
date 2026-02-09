# Быстрый старт

## За 5 минут к запуску

### 1. Установите Ansible (если еще не установлен)
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# Проверка версии (нужна >= 2.12)
ansible --version
```

### 2. Клонируйте проект
```bash
git clone https://github.com/Diogen477/AnsiblePlaybooks.git
cd AnsiblePlaybooks/ansible_for_GitLabRunner
```

### 3. Установите зависимости
```bash
ansible-galaxy collection install -r requirements.yml
```

### 4. Настройте credentials
```bash
# Создайте vault файл из примера
cp group_vars/vault.yml.example group_vars/vault.yml

# Отредактируйте vault.yml - вставьте ваши реальные данные от 1С ИТС
nano group_vars/vault.yml

# Зашифруйте файл
ansible-vault encrypt group_vars/vault.yml
# Введите пароль для vault (запомните его!)
```

### 5. (Опционально) Настройте версии
```bash
# Отредактируйте версии компонентов при необходимости
nano group_vars/all.yml

# Измените:
# edt_version: "2024.2.6"
# platform_version: "8.3.25.1633"
# sonar_scanner_version: "5.0.1.3006"
```

### 6. Запустите установку
```bash
# Полная установка
ansible-playbook site.yml --ask-vault-pass

# Введите пароль от vault когда запросит
```

### 7. Установите пароль для пользователя
```bash
# После завершения установки
sudo passwd gitlab-runner
# Введите надежный пароль (понадобится для RDP)
```

### 8. Подключитесь через RDP
```bash
# Linux
xfreerdp /v:localhost:3389 /u:gitlab-runner

# Windows
mstsc /v:localhost:3389
# Пользователь: gitlab-runner
# Пароль: тот что установили в шаге 7
```

## Готово! 🎉

Теперь у вас полностью настроенное окружение с:
- ✅ Java (Azul Zulu 17)
- ✅ 1C:EDT
- ✅ 1C:Platform
- ✅ SonarScanner
- ✅ RDP доступ

---

## Выборочная установка

### Только базовые компоненты (без 1С)
```bash
ansible-playbook site.yml --ask-vault-pass --tags base,tools
```

### Только 1C EDT (без Platform)
```bash
ansible-playbook site.yml --ask-vault-pass --tags base,edt
```

### Без RDP
```bash
ansible-playbook site.yml --ask-vault-pass --skip-tags rdp
```

---

## Troubleshooting

### Ошибка: "Не удалось найти ссылку для скачивания"
**Причина**: Неверные credentials или недоступная версия

**Решение**:
1. Проверьте логин/пароль от 1С ИТС
2. Проверьте версию на https://releases.1c.ru
3. Убедитесь что подписка ИТС активна

### Ошибка: ansible-vault
**Причина**: Забыли vault пароль или файл не зашифрован

**Решение**:
```bash
# Если забыли пароль - расшифруйте заново
ansible-vault decrypt group_vars/vault.yml
# Отредактируйте
nano group_vars/vault.yml
# Зашифруйте снова
ansible-vault encrypt group_vars/vault.yml
```

### RDP не работает
**Причина**: Не установлен пароль или firewall блокирует

**Решение**:
```bash
# Установите пароль
sudo passwd gitlab-runner

# Проверьте статус xrdp
sudo systemctl status xrdp

# Проверьте порт
sudo netstat -tulpn | grep 3389

# Настройте firewall
sudo ufw allow 3389/tcp
```

---

## Полезные команды

### Проверить синтаксис перед запуском
```bash
ansible-playbook site.yml --syntax-check
```

### Dry-run (без реальных изменений)
```bash
ansible-playbook site.yml --check --ask-vault-pass
```

### Просмотр vault файла
```bash
ansible-vault view group_vars/vault.yml
```

### Редактирование vault файла
```bash
ansible-vault edit group_vars/vault.yml
```

### Проверка установленных компонентов
```bash
java -version
ls /opt/1c/edt/
ls /opt/1cv8/
sonar-scanner --version
systemctl status xrdp
```

---

## Дальнейшие шаги

1. **Настройте GitLab Runner**
   ```bash
   # Установите GitLab Runner
   curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
   sudo apt install gitlab-runner
   
   # Зарегистрируйте runner
   sudo gitlab-runner register
   ```

2. **Настройте SonarQube**
   - Укажите URL SonarQube сервера
   - Получите токен авторизации
   - Настройте в проекте

3. **Создайте .gitlab-ci.yml**
   Пример для 1C проекта:
   ```yaml
   stages:
     - build
     - test
     - quality

   build:
     stage: build
     script:
       - echo "Building 1C project"

   sonar:
     stage: quality
     script:
       - sonar-scanner
   ```

---

## Поддержка

Если возникли проблемы:
1. Проверьте README.md для подробной документации
2. Посмотрите CHANGES.md для списка изменений
3. Создайте Issue на GitHub
