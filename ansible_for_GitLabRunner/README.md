# Ansible Playbook для настройки окружения GitLab Runner с 1C:Enterprise

Автоматизированная установка и настройка окружения для работы с GitLab Runner, включая 1C:EDT, платформу 1С, Java, SonarQube Scanner и инструменты удаленного доступа.

## Требования

- **ОС**: Ubuntu 20.04/22.04/24.04 или Debian 11/12
- **Архитектура**: x86_64
- **RAM**: минимум 4 GB (рекомендуется 8 GB)
- **Диск**: минимум 20 GB свободного пространства
- **Ansible**: >= 2.12

## Подготовка дистрибутивов 1С

Плейбук **не скачивает** дистрибутивы с releases.1c.ru автоматически. Перед запуском необходимо вручную скачать установщики и разместить их в локальных директориях.

### 1. Создайте директории для установщиков

```bash
sudo mkdir -p /opt/installers/edt
sudo mkdir -p /opt/installers/platform
```

### 2. Скачайте дистрибутивы

**1C:EDT** — скачайте с https://releases.1c.ru/project/DevelopmentTools10:
```
Файл: 1c_edt_distr_<версия>_<N>_linux_x86_64.tar.gz
Поместите в: /opt/installers/edt/
```

**1C:Platform** — скачайте с https://releases.1c.ru/project/Platform83:
```
Файл: setup-full-<версия>-x86_64.run  (или архив .tar.gz / .zip)
Поместите в: /opt/installers/platform/
```

### 3. Проверьте

```bash
ls -la /opt/installers/edt/
# Должен быть: 1c_edt_distr_2024.2.6_7_linux_x86_64.tar.gz

ls -la /opt/installers/platform/
# Должен быть: setup-full-8.3.25.1633-x86_64.run (или архив)
```

> **Путь по умолчанию:** `/opt/installers/`. Можно изменить переменную `installers_base_dir` в `group_vars/all.yml`.

## Быстрый старт

```bash
# 1. Клонируйте репозиторий
git clone https://github.com/Diogen477/AnsiblePlaybooks.git
cd AnsiblePlaybooks/ansible_for_GitLabRunner

# 2. Установите зависимости
ansible-galaxy collection install -r requirements.yml

# 3. Разместите дистрибутивы 1С (см. раздел выше)

# 4. При необходимости отредактируйте версии
nano group_vars/all.yml

# 5. Проверьте синтаксис
ansible-playbook site.yml --syntax-check

# 6. Запустите
ansible-playbook site.yml

# 7. Установите пароль для пользователя RDP
sudo passwd gitlab-runner
```

## Установленные компоненты

| Компонент | Описание |
|-----------|----------|
| Системные утилиты | ssh, sshpass, unzip, screen, curl, wget |
| Графическая оболочка | Xorg, OpenBox, Xvfb |
| xRDP | Удалённый доступ по протоколу RDP |
| Java | Azul Zulu 17 JDK |
| 1C:EDT | Enterprise Development Tools |
| 1C:Platform | Платформа 1С:Предприятие 8.3 |
| SonarQube Scanner | Статический анализ кода |

## Структура проекта

```
ansible_for_GitLabRunner/
├── ansible.cfg
├── inventory
├── site.yml
├── requirements.yml
├── group_vars/
│   └── all.yml
└── roles/
    ├── common/          # Базовые пакеты, xRDP, графическая оболочка
    ├── java/            # Azul Zulu 17 JDK
    ├── edt/             # 1C:EDT (из локальной директории)
    ├── platform/        # 1C:Platform (из локальной директории)
    ├── sonar-scanner/   # SonarQube Scanner CLI
    ├── gitlab-user/     # Создание пользователя gitlab-runner
    └── rdp-tools/       # Настройка xRDP
```

## Настройка

### Переменные (`group_vars/all.yml`)

```yaml
# Версии
edt_version: "2024.2.6"
platform_version: "8.3.25.1633"
sonar_scanner_version: "5.0.1.3006"

# Пути к установщикам
installers_base_dir: "/opt/installers"

# RDP
rdp_port: 3389
rdp_allow_root: false   # true — разрешить root через RDP
```

### Выборочная установка

```bash
# Только базовые компоненты (без 1С)
ansible-playbook site.yml --tags base,tools

# Только 1С
ansible-playbook site.yml --tags 1c

# Без RDP
ansible-playbook site.yml --skip-tags rdp

# С обновлением системы
ansible-playbook site.yml --tags all,system_update
```

## Подключение через RDP

```bash
# Linux
xfreerdp /v:server_ip:3389 /u:gitlab-runner

# Windows — mstsc.exe
# Сервер: server_ip:3389
# Пользователь: gitlab-runner
```

## Безопасность

- Root доступ через RDP отключён по умолчанию
- Создаётся отдельный пользователь `gitlab-runner` для работы
- GPG-ключи хранятся в `/etc/apt/keyrings/` (современный метод)
- SonarScanner PATH добавляется через `/etc/profile.d/` (не затирает PATH)

## Возможные проблемы

**«Дистрибутив не найден»** — убедитесь, что файлы размещены в `/opt/installers/edt/` и `/opt/installers/platform/`, и имя файла содержит указанную версию.

**RDP не работает** — проверьте `sudo systemctl status xrdp` и установлен ли пароль: `sudo passwd gitlab-runner`.

**Java apt_key warning** — в данной версии используется современный метод с `signed-by`, предупреждений быть не должно.
