# Домашнее задание к занятию 4 «Оркестрация группой Docker контейнеров на примере Docker Compose»

**Выполнил:** Шаров Олег  
**Дата:** Май 2026  
**Репозиторий:** [Myth3916/docker-task1](https://github.com/Myth3916/docker-task1)

---

## 🔗 Ссылки

| Ресурс | Ссылка |
|--------|--------|
| Репозиторий на GitHub | [Myth3916/docker-task1](https://github.com/Myth3916/docker-task1) |
| Docker Hub репозиторий | [myth3916/custom-nginx](https://hub.docker.com/r/myth3916/custom-nginx) |

---

## ✅ Задача 1: Сборка и отправка образа custom-nginx

### 🔹 Шаги выполнения:

1. **Установка Docker на Arch Linux:**
   ```bash
   sudo pacman -S docker docker-compose-plugin
   sudo systemctl enable --now docker
   sudo usermod -aG docker $USER
   ```

2. **Создан Dockerfile:**
   ```dockerfile
   FROM nginx:1.29.0
   COPY index.html /usr/share/nginx/html/index.html
   RUN chmod 644 /usr/share/nginx/html/index.html
   ```

3. **Сборка образа:**
   ```bash
   docker build -t myth3916/custom-nginx:1.0.0 .
   ```

4. **Отправка в Docker Hub:**
   ```bash
   docker login
   docker push myth3916/custom-nginx:1.0.0
   ```

### 🔹 Скриншот:

![Docker Hub репозиторий](./screenshots/task1-dockerhub.png)
*Рисунок 1: Образ custom-nginx:1.0.0 в Docker Hub*

---

## ✅ Задача 2: Запуск и проверка контейнера

### 🔹 Команды:

**Запуск контейнера:**
```bash
docker run -d \
  --name Oleg-custom-nginx-t2 \
  -p 127.0.0.1:8080:80 \
  myth3916/custom-nginx:1.0.0
```

**Переименование:**
```bash
docker rename Oleg-custom-nginx-t2 custom-nginx-t2
```

**Исправление прав доступа (403 Forbidden):**
```bash
docker exec -u root custom-nginx-t2 chmod 644 /usr/share/nginx/html/index.html
```

**Проверка доступности:**
```bash
curl http://127.0.0.1:8080
```

### 🔹 Скриншоты:

![Запуск и переименование](./screenshots/task2-run-rename.png)
*Рисунок 2: Запуск контейнера и переименование*

![Проверочная команда](./screenshots/task2-verify-check.png)
*Рисунок 3: Вывод команды date, docker ps, ss, logs, base64*

![Проверка curl](./screenshots/task2-curl-check.png)
*Рисунок 4: Проверка доступности страницы через curl*

---

## ✅ Задача 3: Работа с потоками и конфигурацией

### 🔹 3.1. Подключение к стандартному потоку контейнера

**Команды:**
```bash
docker attach custom-nginx-t2
# Нажатие Ctrl-C
docker ps -a
docker start custom-nginx-t2
```

**Почему контейнер остановился после Ctrl-C?**

> При использовании `docker attach` мы подключаемся к основному процессу контейнера (nginx). 
> Нажатие `Ctrl-C` отправляет сигнал `SIGINT` (сигнал прерывания, код 2), который nginx 
> обрабатывает как команду на завершение работы. Nginx корректно завершает все worker-процессы 
> и останавливается, что приводит к остановке контейнера.

### 🔹 Скриншоты:

![Attach и остановка](./screenshots/task3-attach-stop.png)
*Рисунок 5: Подключение к контейнеру и остановка через Ctrl-C*

![Перезапуск контейнера](./screenshots/task3-restart.png)
*Рисунок 6: Перезапуск контейнера командой docker start*

---

### 🔹 3.2. Вход в контейнер и изменение конфигурации

**Команды:**
```bash
docker exec -it custom-nginx-t2 bash
apt-get update
apt-get install -y nano
nano /etc/nginx/conf.d/default.conf
# Изменено: listen 80 → listen 81
nginx -s reload
curl http://127.0.0.1:80   # Ошибка подключения
curl http://127.0.0.1:81   # Успешный ответ
exit
```

### 🔹 Скриншот:

![Редактирование конфига и reload](./screenshots/task3-nginx-reload.png)
*Рисунок 7: Изменение порта nginx и проверка работы*

---

### 🔹 3.3. Проблема с портами

**Проверка на хостовой машине:**
```bash
ss -tlpn | grep 127.0.0.1:8080
docker port custom-nginx-t2
curl http://127.0.0.1:8080
```

**Суть возникшей проблемы:**

> При запуске контейнера был настроен проброс портов: `-p 127.0.0.1:8080:80`. 
> Это означает: "всё, что приходит на порт хоста 8080, перенаправлять на порт контейнера 80".
> 
> После изменения конфигурации nginx внутри контейнера на `listen 81`, он перестал слушать 
> порт 80. Docker продолжает пересылать трафик на порт 80, но там никто не слушает. 
> В результате соединение сбрасывается с ошибкой "Recv failure: Соединение разорвано 
> другой стороной".
> 
> **Решение:** нужно либо вернуть порт 80 в конфиге nginx, либо пересоздать контейнер с 
> правильным маппингом портов (например, `-p 127.0.0.1:8080:81`).

### 🔹 Скриншот:

![Проблема с портами](./screenshots/task3-port-problem.png)
*Рисунок 8: Демонстрация проблемы с портами*

---

### 🔹 3.4. Удаление работающего контейнера

**Команда:**
```bash
docker rm -f custom-nginx-t2
```

**Объяснение:** Флаг `-f` (force) принудительно останавливает работающий контейнер 
(отправляет SIGKILL) и затем удаляет его.

### 🔹 Скриншот:

![Удаление контейнера](./screenshots/task3-force-remove.png)
*Рисунок 9: Принудительное удаление контейнера*

---

## ⏳ Задача 4: Общие тома (Volume) между контейнерами

*Выполняется...*

---

## ⏳ Задача 5: Docker Compose и Portainer

*Выполняется...*

---

## 🛠️ Полезные команды

```bash
# Проверить образы
docker images

# Проверить контейнеры
docker ps -a

# Просмотр логов
docker logs <container_name>

# Вход в контейнер
docker exec -it <container_name> bash

# Подключение к потоку
docker attach <container_name>

# Очистка
docker system prune -a
```

---

## 📚 Источники

- [Docker Hub](https://hub.docker.com)
- [Docker Documentation](https://docs.docker.com)
- [Nginx Documentation](https://nginx.org/en/docs/)

---

> 💡 **Примечание:** Все скриншоты сохранены в папке `./screenshots/`
```

