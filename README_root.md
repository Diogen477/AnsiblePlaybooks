# Ansible Playbooks для инфраструктуры CI/CD с 1С:Enterprise

Набор Ansible playbook'ов для автоматизированного развёртывания инфраструктуры непрерывной интеграции и анализа качества кода проектов на платформе 1С:Enterprise.

## Состав репозитория

| Playbook | Назначение |
|----------|-----------|
| [ansible_for_GitLabRunner](ansible_for_GitLabRunner/) | Подготовка сервера GitLab Runner: Java, 1C:EDT, 1C:Platform, SonarScanner, RDP-доступ |
| [ansible_for_SonarQube](ansible_for_SonarQube/) | Установка SonarQube с PostgreSQL и Community Branch Plugin |

Оба playbook'а разворачиваются на одном или нескольких серверах и вместе формируют полный CI/CD-контур для проектов 1С: анализ кода через SonarQube, сборка и тестирование через GitLab Runner с установленными EDT и платформой.

## Схема инфраструктуры

```
┌─────────────────────────────────────────────────────────┐
│                      GitLab Server                      │
│                  (репозитории, CI/CD)                    │
└──────────────┬──────────────────────┬───────────────────┘
               │  .gitlab-ci.yml      │
               ▼                      ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│   GitLab Runner Server   │  │    SonarQube Server      │
│  ┌────────────────────┐  │  │  ┌────────────────────┐  │
│  │ 1C:EDT             │  │  │  │ SonarQube 25.8     │  │
│  │ 1C:Platform 8.3    │  │  │  │ PostgreSQL 16      │  │
│  │ Java (Zulu 17)     │──┼──│  │ Branch Plugin      │  │
│  │ SonarScanner       │  │  │  └────────────────────┘  │
│  │ xRDP               │  │  │     http://:9000         │
│  └────────────────────┘  │  └──────────────────────────┘
│  ansible_for_GitLabRunner│  │  ansible_for_SonarQube   │
└──────────────────────────┘  └──────────────────────────┘
```

## Требования

**Целевые серверы:**
- Ubuntu 20.04 / 22.04 / 24.04 или Debian 11 / 12
- Архитектура x86_64
- Минимум 4 GB RAM (рекомендуется 8 GB)
- Root или sudo-доступ

**Управляющая машина:**
- Ansible >= 2.12
- Python 3.8+

## Быстрый старт

```bash
# 1. Клонируйте репозиторий
git clone https://github.com/Diogen477/AnsiblePlaybooks.git
cd AnsiblePlaybooks

# 2. Установите зависимости Ansible
ansible-galaxy collection install -r ansible_for_GitLabRunner/requirements.yml
ansible-galaxy collection install -r ansible_for_SonarQube/requirements.yml

# 3. Разверните SonarQube
cd ansible_for_SonarQube
nano group_vars/all.yml          # Задайте надёжный db_password
ansible-playbook sonarqube.yml
cd ..

# 4. Подготовьте дистрибутивы 1С и разверните GitLab Runner
cd ansible_for_GitLabRunner
sudo mkdir -p /opt/installers/{edt,platform}
# Скачайте дистрибутивы с releases.1c.ru и положите в /opt/installers/
ansible-playbook site.yml
sudo passwd gitlab-runner
```

Подробные инструкции — в README каждого playbook'а.

## Особенности

**Безопасность:**
- Дистрибутивы 1С устанавливаются из локальных файлов, без хранения учётных данных ИТС в playbook'е
- PostgreSQL слушает только localhost
- Root RDP-доступ отключён по умолчанию, создаётся отдельный пользователь `gitlab-runner`
- GPG-ключи репозиториев хранятся по современному стандарту в `/etc/apt/keyrings/`
- Поддержка Ansible Vault для хранения паролей

**Идемпотентность:**
- Повторные запуски безопасны — уже установленные компоненты не переустанавливаются
- Handlers вместо прямых перезапусков сервисов
- Проверки наличия файлов и сервисов перед каждой операцией

**Гибкость:**
- Теги для выборочной установки компонентов
- Все версии и пути вынесены в переменные (`group_vars/all.yml`)
- Поддержка как локального, так и удалённого развёртывания

## Структура репозитория

```
AnsiblePlaybooks/
├── README.md                          # Этот файл
├── ansible_for_GitLabRunner/
│   ├── README.md                      # Документация playbook'а
│   ├── QUICKSTART.md                  # Быстрый старт
│   ├── site.yml                       # Главный playbook
│   ├── ansible.cfg
│   ├── inventory
│   ├── requirements.yml
│   ├── group_vars/
│   │   └── all.yml                    # Версии, пути, настройки
│   └── roles/
│       ├── common/                    # Базовые пакеты, Xorg, xRDP
│       ├── java/                      # Azul Zulu JDK 17
│       ├── edt/                       # 1C:EDT из локального архива
│       ├── platform/                  # 1C:Platform из локального архива
│       ├── sonar-scanner/             # SonarQube Scanner CLI
│       ├── gitlab-user/               # Пользователь gitlab-runner
│       └── rdp-tools/                 # Настройка xRDP
│
└── ansible_for_SonarQube/
    ├── README.md                      # Документация playbook'а
    ├── QUICKSTART.md                  # Быстрый старт
    ├── sonarqube.yml                  # Главный playbook
    ├── ansible.cfg
    ├── inventory.ini
    ├── requirements.yml
    ├── group_vars/
    │   ├── all.yml                    # Версии, пароли, настройки
    │   └── vault.yml.example          # Пример Ansible Vault
    └── roles/
        ├── common/                    # OpenJDK 17, базовые пакеты
        ├── postgresql/                # PostgreSQL 16
        ├── sonarqube/                 # SonarQube сервер
        └── branch_plugin/            # Community Branch Plugin + webapp
```

## Полезные ссылки

- [Документация Ansible](https://docs.ansible.com/)
- [1C:EDT Downloads](https://releases.1c.ru/project/DevelopmentTools10)
- [1C:Platform Downloads](https://releases.1c.ru/project/Platform83)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Community Branch Plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin)
- [GitLab Runner](https://docs.gitlab.com/runner/)

## Лицензия

MIT

## Автор

[Diogen477](https://github.com/Diogen477)
