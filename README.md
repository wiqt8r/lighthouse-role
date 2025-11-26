# Ansible Role: lighthouse-role

Роль для автоматизированной установки и настройки веб-интерфейса [Lighthouse](https://github.com/VKCOM/lighthouse) — системы мониторинга и управления для ClickHouse.

## Описание роли

Роль выполняет следующие задачи:
- Устанавливает Nginx и Git
- Создаёт каталог для статики Lighthouse
- Клонирует репозиторий Lighthouse
- Развертывает конфигурацию Nginx из шаблона
- Запускает и включает службу Nginx

## Требования

- Ansible 2.12+
- Поддерживаемые ОС:
  - Семейство RedHat (AlmaLinux, CentOS и т.п.)
  - Семейство Debian (если добавить поддержку через `apt`)
- Права на установку пакетов и управление systemd (обычно `become: true`)

## Переменные роли

### Переменные по умолчанию (defaults/main.yml)

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `lighthouse_repo_url` | `"https://github.com/VKCOM/lighthouse.git"` | URL Git-репозитория Lighthouse |
| `lighthouse_docroot` | `"/var/www/lighthouse"` | Каталог для статики Lighthouse |
| `lighthouse_nginx_conf` | `"/etc/nginx/conf.d/lighthouse.conf"` | Путь до конфигурационного файла Nginx |
| `lighthouse_server_name` | `"_"` | Имя виртуального хоста |
| `lighthouse_listen_port` | `80` | Порт для Nginx |

### Внутренние переменные (vars/main.yml)

| Переменная | Описание |
|------------|----------|
| `lighthouse_nginx_service_name` | Имя службы веб-сервера (по умолчанию nginx) |

## Обработчики

| Обработчик | Описание |
|------------|----------|
| `restart nginx` | Перезапускает службу Nginx через ansible.builtin.service |

## Шаблоны

| Файл | Описание |
|------|----------|
| `templates/lighthouse.conf.j2` | Конфигурация виртуального хоста Nginx для Lighthouse |

### Пример шаблона

```nginx
server {
    listen {{ lighthouse_listen_port }};
    server_name {{ lighthouse_server_name }};

    root {{ lighthouse_docroot }};
    index index.html index.htm;

    location / {
        try_files \$uri \$uri/ =404;
    }
}
```

## Пример использования

### Установка роли через requirements.yml

```yaml
---
- src: https://github.com/wiqt8r/lighthouse-role.git
  scm: git
  version: "v0.1.0"
  name: lighthouse-role
```

Установка роли:
```bash
ansible-galaxy install -r requirements.yml -p roles
```

### Пример playbook

```yaml
---
- name: Install Lighthouse
  hosts: lighthouse
  roles:
    - role: lighthouse-role
  tags: [lighthouse]
```

### Переопределение переменных

При необходимости параметры можно переопределить в playbook:

```yaml
lighthouse_docroot: "/srv/lighthouse"
lighthouse_server_name: "lighthouse.example.com"
lighthouse_listen_port: 8080
```

## Зависимости

Роль не имеет жёстких зависимостей от других ролей. Для корректной работы Lighthouse должен иметь доступ к инстансу ClickHouse, который может быть развернут отдельной ролью.

## Лицензия

Этот проект распространяется под лицензией MIT.

## Автор

**wiqt8r** - [GitHub](https://github.com/wiqt8r)

## Благодарности

- [VKCOM](https://github.com/VKCOM) за отличный инструмент мониторинга ClickHouse
- [Lighthouse](https://github.com/VKCOM/lighthouse) за удобный веб-интерфейс
