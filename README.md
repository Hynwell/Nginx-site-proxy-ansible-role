# Ansible роль: nginx stream proxy для обхода блокировок

## Описание
Роль настраивает nginx как TCP stream-прокси с поддержкой SNI и динамическим управлением upstream для разных сервисов и доменов. Используется для обхода блокировок — весь трафик к нужным сервисам идёт через зарубежный сервер.

## Требования
- Для работы необходим зарубежный сервер с внешним (не российским) IP-адресом.
- Сервер должен быть доступен по SSH для Ansible.

## Быстрый старт

1. **Скопируйте шаблоны файлов:**

```sh
cp template.inventory.yml inventory.yml
cp template.play.yml play.yml
```

2. **Заполните inventory.yml** — укажите внешний IP вашего зарубежного сервера:

```yaml
all:
  hosts:
    proxy:
      ansible_host: 1.2.3.4  # ← здесь ваш внешний IP
  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    ansible_user: root
```

3. **Заполните play.yml** (пример структуры сервисов и доменов см. ниже)

> Название сервиса (ключ `name`) может быть любым, например: `chatgpt`, `openai`, `spotify` и т.д.

4. **Запустите плейбук:**

```sh
ansible-playbook -i inventory.yml play.yml
```

## Как добавить сервисы и домены

В переменной `nginx_streams` указывайте сервисы и их домены:

```yaml
nginx_streams:
  - name: chatgpt
    domains:
      - chatgpt.com
      - ab.chatgpt.com
  - name: openai
    domains:
      - auth.openai.com
      - platform.openai.com
```

## Что делает роль
- Проксирует трафик на зарубежный сервер
- Выводит строку для добавления в /etc/hosts в удобном виде

## Пример вывода для hosts

```
# chatgpt
1.2.3.4 chatgpt.com ab.chatgpt.com

# openai
1.2.3.4 auth.openai.com platform.openai.com
```

## Важно
- Обязательно добавьте внешний IP вашего зарубежного сервера и нужные домены в файл `/etc/hosts` на клиентских устройствах **или** пропишите их в DNS вашего роутера.
- Без этого проксирование работать не будет!

## Важно: DNS и ошибки nginx
- nginx требует, чтобы все домены из списка были резолвимы на момент проверки конфига (nginx -t).
- Если какой-то домен не резолвится (например, xpui.app.spotify.com), nginx не запустится и выдаст ошибку вида:
  > host not found in upstream ...





