# Быстрый старт

## За 5 минут к запуску

### 1. Установите Ansible (если ещё не установлен)

```bash
sudo apt update
sudo apt install ansible

# Проверка (нужна >= 2.12)
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

### 4. Подготовьте дистрибутивы 1С

```bash
# Создайте директории
sudo mkdir -p /opt/installers/edt
sudo mkdir -p /opt/installers/platform

# Скачайте дистрибутивы с https://releases.1c.ru и положите:
# EDT (.tar.gz)       → /opt/installers/edt/
# Platform (.run/.zip) → /opt/installers/platform/
```

### 5. (Опционально) Настройте версии

```bash
nano group_vars/all.yml

# При необходимости измените:
# edt_version: "2024.2.6"
# platform_version: "8.3.25.1633"
# sonar_scanner_version: "5.0.1.3006"
```

### 6. Запустите установку

```bash
ansible-playbook site.yml
```

### 7. Установите пароль для пользователя

```bash
sudo passwd gitlab-runner
```

### 8. Подключитесь через RDP

```bash
# Linux
xfreerdp /v:localhost:3389 /u:gitlab-runner

# Windows
mstsc /v:localhost:3389
# Пользователь: gitlab-runner
```

## Готово!

Установленные компоненты:
- Java (Azul Zulu 17)
- 1C:EDT
- 1C:Platform
- SonarScanner
- RDP-доступ

---

## Выборочная установка

```bash
# Только базовые компоненты (без 1С)
ansible-playbook site.yml --tags base,tools

# Только 1С
ansible-playbook site.yml --tags 1c

# Без RDP
ansible-playbook site.yml --skip-tags rdp

# С обновлением ОС
ansible-playbook site.yml --tags all,system_update
```

---

## Решение проблем

### Дистрибутив не найден

Playbook выведет сообщение с ожидаемым путём и именем файла. Проверьте:

```bash
ls -la /opt/installers/edt/
ls -la /opt/installers/platform/
```

### RDP не работает

```bash
sudo passwd gitlab-runner             # Установите пароль
sudo systemctl status xrdp            # Проверьте статус
sudo ufw allow 3389/tcp               # Откройте порт
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

## Полезные команды

```bash
# Проверка синтаксиса
ansible-playbook site.yml --syntax-check

# Dry-run (без реальных изменений)
ansible-playbook site.yml --check
```

---

## Дальнейшие шаги

1. **Установите GitLab Runner:**
   ```bash
   curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
   sudo apt install gitlab-runner
   sudo gitlab-runner register
   ```

2. **Разверните SonarQube** — см. `../ansible_for_SonarQube/`

3. **Настройте `.gitlab-ci.yml`** в вашем проекте

Подробная документация — в [README.md](README.md).
