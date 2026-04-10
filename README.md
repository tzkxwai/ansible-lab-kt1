# Ansible playbooks (учебное задание)

Структура проекта:

- `ansible.cfg` — каталог инвентаря и общие параметры.
- `inventory/hosts` — целевые хосты (группа `NGINX` и родитель `servers`).
- `group_vars/NGINX.yml` — переменные для Nginx.
- `roles/nginx/` — роль установки Nginx (сценарий 1).
- `01-nginx.yml` … `11-prometheus.yml` — отдельные сценарии.

Коллекции для `06-firewall.yml` и `10-sysctl.yml`:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

На контроллере для хэширования пароля в `02-username.yml` нужен **passlib**:

```bash
pip install passlib
```

## Запуск и проверка

Выполняйте из корня проекта (`kt1`), где лежит `ansible.cfg`.

| Файл | Запуск | Как проверить |
|------|--------|----------------|
| `01-nginx.yml` | `ansible-playbook 01-nginx.yml` | `curl -I http://<host>/` или браузер; `systemctl status nginx` на хосте. |
| `02-username.yml` | `ansible-playbook 02-username.yml` | `ssh newadmin@<host>` (пароль из плейбука: `123`). |
| `03-system-update.yml` | `ansible-playbook 03-system-update.yml` | Лог apt/yum без ошибок; при наличии `/var/run/reboot-required` — хост перезагрузится. |
| `04-config-backup.yml` | `ansible-playbook 04-config-backup.yml` | На сервере: `ls -la /root/ansible_config_backups/`. |
| `05-services.yml` | `ansible-playbook 05-services.yml` | `systemctl status ssh` (или выбранный в vars сервис). |
| `06-firewall.yml` | `ansible-playbook 06-firewall.yml` | Debian: `ufw status`; RHEL: `firewall-cmd --list-all`. |
| `07-postgresql.yml` | `ansible-playbook 07-postgresql.yml` | `systemctl status postgresql`; `sudo -u postgres psql -c '\l'`. |
| `08-git-deploy.yml` | `ansible-playbook 08-git-deploy.yml -e git_repo=URL -e git_dest=/opt/app` | Каталог `git_dest` существует и содержит клон `.git`. |
| `09-lineinfile.yml` | `ansible-playbook 09-lineinfile.yml` | `grep title /var/www/html/index.html` — строка с `<title>Managed by Ansible</title>`. |
| `10-sysctl.yml` | `ansible-playbook 10-sysctl.yml` | `sysctl vm.swappiness` → `10`. |
| `11-prometheus.yml` | `ansible-playbook 11-prometheus.yml` | `systemctl status prometheus`; HTTP `http://<host>:9090` (если порт открыт). |

Проверка синтаксиса любого плейбука:

```bash
ansible-playbook --syntax-check 01-nginx.yml
```

Режим без изменений:

```bash
ansible-playbook --check 01-nginx.yml
```

Перед запуском отредактируйте `inventory/hosts` (пример — `inventory/hosts.example`).

## GitHub Codespaces

Да, этим репозиторием можно пользоваться в **GitHub Codespaces**: в корне есть [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json). После создания codespace один раз выполнится установка пакетов из apt (`ansible`, `python3-passlib`, `sshpass`, `openssh-client`) и коллекций из `collections/requirements.yml`.

1. Загрузите проект в репозиторий на GitHub.
2. **Code → Codespaces → Create codespace on main** (или на нужной ветке).
3. В терминале codespace: `ansible-playbook --version`, затем обычные команды из таблицы выше.

**Важно:** целевые ВМ (Vagrant, облако и т.д.) должны быть **доступны по сети из интернета**, куда выходит codespace. Localhost вашего ПК из codespace не виден: укажите в `inventory/hosts` публичный IP или VPN/туннель. Для проверки синтаксиса и линтинга codespace подходит без удалённых хостов: `ansible-playbook --syntax-check …`.
