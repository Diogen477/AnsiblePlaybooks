# Ansible Playbook для настройки GitLab Runner с 1С:Enterprise

Автоматизированная подготовка сервера GitLab Runner для CI/CD проектов на платформе 1С:Enterprise. Playbook устанавливает все необходимые компоненты: Java, 1C:EDT, 1C:Platform, SonarScanner и настраивает удалённый доступ через RDP.

## Требования

- **ОС:** Ubuntu 20.04 / 22.04 / 24.04 или Debian 11 / 12
- **Архитектура:** x86_64
- **RAM:** минимум 4 GB (рекомендуется 8 GB)
- **Диск:** минимум 20 GB свободного пространства
- **Ansible:** >= 2.12

## Подготовка дистрибутивов 1С

Playbook **не скачивает** дистрибутивы с releases.1c.ru автоматически. Установщики нужно скачать вручную и разместить в локальных директориях на целевой машине.

### Шаг 1. Создайте директории

```bash
sudo mkdir -p /opt/installers/edt
sudo mkdir -p /opt/installers/platform
```

### Шаг 2. Скачайте дистрибутивы

| Продукт | Откуда скачать | Что положить |
|---------|---------------|-------------|
| 1C:EDT | [releases.1c.ru/project/DevelopmentTools10](https://releases.1c.ru/project/DevelopmentTools10) | `1c_edt_distr_<версия>_<N>_linux_x86_64.tar.gz` → `/opt/installers/edt/` |
| 1C:Platform | [releases.1c.ru/project/Platform83](https://releases.1c.ru/project/Platform83) | `setup-full-<версия>*.run` или архив `.tar.gz` / `.zip` → `/opt/installers/platform/` |

### Шаг 3. Проверьте

```bash
ls /opt/installers/edt/
# 1c_edt_distr_2024.2.6_7_linux_x86_64.tar.gz

ls /opt/installers/platform/
# setup-full-8.3.25.1633-x86_64.run  (или архив с установщиком)
```

> Путь по умолчанию — `/opt/installers/`. Можно изменить через переменную `installers_base_dir` в `group_vars/all.yml`.

## Быстрый старт

```bash
# 1. Перейдите в директорию playbook'а
cd ansible_for_GitLabRunner

# 2. Установите коллекции Ansible
ansible-galaxy collection install -r requirements.yml

# 3. Разместите дистрибутивы 1С (см. раздел выше)

# 4. При необходимости скорректируйте версии
nano group_vars/all.yml

# 5. Запустите
ansible-playbook site.yml

# 6. Задайте пароль пользователя для RDP
sudo passwd gitlab-runner

# 7. Подключитесь
xfreerdp /v:localhost:3389 /u:gitlab-runner
```

## Установленные компоненты

| Роль | Что устанавливается |
|------|---------------------|
| `common` | Обновление пакетов, ssh, curl, wget, Xorg, OpenBox, xRDP |
| `java` | Azul Zulu JDK 17 (репозиторий с `signed-by`) |
| `edt` | 1C:Enterprise Development Tools из локального архива |
| `platform` | 1C:Platform 8.3 из локального архива / установщика |
| `sonar-scanner` | SonarQube Scanner CLI (скачивается с binaries.sonarsource.com) |
| `gitlab-user` | Системный пользователь `gitlab-runner` с домашней директорией и sudo |
| `rdp-tools` | Настройка xRDP: порт, шифрование, ограничение root-доступа |

## Структура проекта

```
ansible_for_GitLabRunner/
├── site.yml                  # Главный playbook с pre-tasks проверками
├── ansible.cfg               # Конфигурация Ansible
├── inventory                 # Inventory (localhost)
├── requirements.yml          # Зависимости коллекций
├── .gitignore
├── group_vars/
│   └── all.yml              # Все переменные: версии, пути, настройки
└── roles/
    ├── common/
    │   ├── tasks/main.yml   # Базовые пакеты, Xorg, xRDP
    │   ├── handlers/main.yml
    │   └── meta/main.yml
    ├── java/
    │   ├── tasks/main.yml   # Azul Zulu 17 (современный signed-by)
    │   ├── handlers/main.yml
    │   └── meta/main.yml
    ├── edt/
    │   ├── tasks/main.yml   # Поиск и установка EDT из /opt/installers/edt/
    │   └── meta/main.yml
    ├── platform/
    │   ├── tasks/main.yml   # Поиск и установка Platform из /opt/installers/platform/
    │   └── meta/main.yml
    ├── sonar-scanner/
    │   ├── tasks/main.yml   # Загрузка, распаковка, PATH через profile.d
    │   ├── handlers/main.yml
    │   └── meta/main.yml
    ├── gitlab-user/
    │   ├── tasks/main.yml   # Создание пользователя и окружения
    │   ├── defaults/main.yml
    │   └── meta/main.yml
    └── rdp-tools/
        ├── tasks/main.yml   # Настройка xRDP
        ├── handlers/main.yml
        └── meta/main.yml
```

## Настройка

### Переменные (`group_vars/all.yml`)

```yaml
# Версии продуктов
edt_version: "2024.2.6"
platform_version: "8.3.25.1633"
sonar_scanner_version: "5.0.1.3006"

# Пути к локальным установщикам
installers_base_dir: "/opt/installers"
edt_installers_dir: "{{ installers_base_dir }}/edt"
platform_installers_dir: "{{ installers_base_dir }}/platform"

# Пути установки
edt_install_path: "/opt/1c/edt"
platform_install_path: "/opt/1cv8"
sonar_scanner_path: "/opt/sonar-scanner"
java_home: "/usr/lib/jvm/zulu17"

# Пользователь
gitlab_runner_user: "gitlab-runner"
gitlab_runner_home: "/home/gitlab-runner"

# RDP
rdp_port: 3389
rdp_allow_root: false
```

### Теги для выборочной установки

| Тег | Роли |
|-----|------|
| `base` | common, java, gitlab-user |
| `1c` | edt, platform |
| `tools` | sonar-scanner |
| `rdp` | rdp-tools |
| `edt` | только edt |
| `platform` | только platform |
| `system_update` | обновление ОС (отключено по умолчанию) |

Примеры:

```bash
# Только базовые компоненты без 1С
ansible-playbook site.yml --tags base,tools

# Только продукты 1С
ansible-playbook site.yml --tags 1c

# Всё кроме RDP
ansible-playbook site.yml --skip-tags rdp

# Запуск с обновлением ОС
ansible-playbook site.yml --tags all,system_update
```

### Запуск на удалённом хосте

Отредактируйте `inventory`:

```ini
[local]
runner.example.com ansible_user=ubuntu ansible_become=yes
```

Уберите `connection: local` из `site.yml` и запустите:

```bash
ansible-playbook -i inventory site.yml
```

## Безопасность

- **Root RDP отключён** по умолчанию (`rdp_allow_root: false`). Для работы используется пользователь `gitlab-runner`.
- **GPG-ключи** хранятся в `/etc/apt/keyrings/` с `signed-by` — deprecated модуль `apt_key` не используется.
- **PATH** — SonarScanner добавляется через `/etc/profile.d/sonar-scanner.sh`, системный PATH не перезаписывается.
- **Обновление ОС** отключено по умолчанию (`safe` upgrade с тегом `never`), запускается явно через `--tags system_update`.
- **Дистрибутивы 1С** берутся из локальных файлов — учётные данные ИТС не хранятся в playbook'е.
- **Идемпотентность** — повторные запуски безопасны: проверяется наличие уже установленных компонентов, временные файлы очищаются через `always`.

## Проверка установки

```bash
java -version                         # Java
ls /opt/1c/edt/                       # EDT
ls /opt/1cv8/                         # Platform
sonar-scanner --version               # SonarScanner
systemctl status xrdp                 # RDP
id gitlab-runner                      # Пользователь
```

## Решение проблем

**«Дистрибутив EDT / Platform не найден»**
Playbook выдаёт подробное сообщение с указанием ожидаемого пути и имени файла. Убедитесь, что файл размещён в правильной директории и его имя содержит указанную в `all.yml` версию.

**RDP не работает**
```bash
sudo systemctl status xrdp           # Статус службы
sudo passwd gitlab-runner            # Убедитесь что пароль установлен
sudo ufw allow 3389/tcp              # Откройте порт в firewall
```

**Java: предупреждение о deprecated apt_key**
В данной версии используется `get_url` + `signed-by`. Если предупреждение всё равно появляется — скорее всего на сервере сохранился старый ключ от предыдущих установок. Удалите его: `sudo rm -f /etc/apt/trusted.gpg.d/azul*`.

**SonarScanner: команда не найдена**
Переменные окружения обновляются через `/etc/profile.d/`. Переподключитесь к серверу или выполните `source /etc/profile.d/sonar-scanner.sh`. Также доступен симлинк `/usr/local/bin/sonar-scanner`.

## Дальнейшие шаги

После успешного запуска playbook'а:

1. **Установите GitLab Runner:**
   ```bash
   curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
   sudo apt install gitlab-runner
   sudo gitlab-runner register
   ```

2. **Настройте SonarQube** — используйте [ansible_for_SonarQube](../ansible_for_SonarQube/) из этого же репозитория.

3. **Создайте `.gitlab-ci.yml`** для проекта 1С:
   ```yaml
   stages:
     - build
     - quality

   sonar:
     stage: quality
     script:
       - sonar-scanner
         -Dsonar.projectKey=my-1c-project
         -Dsonar.sources=src
         -Dsonar.host.url=http://sonarqube.example.com:9000
         -Dsonar.token=$SONAR_TOKEN
   ```

## Лицензия

MIT

## Автор

[Diogen477](https://github.com/Diogen477)
