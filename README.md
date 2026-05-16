# Домашнее задание к занятию 4 «Оркестрация группой Docker контейнеров на примере Docker Compose»

**Выполнил:** Шаров Олег  
**Дата:** Май 2026  
**Репозиторий:** [myth3916/docker-task1](https://github.com/Myth3916/docker-task1)

---


## 🔗 Ссылки

| Задача | Ссылка |
|--------|--------|
| Репозиторий на GitHub | [ссылка](https://github.com/Myth3916/docker-task1) |
| Docker Hub репозиторий | [myth3916/custom-nginx](https://hub.docker.com/r/myth3916/custom-nginx) |

---

## ✅ Задача 1: Сборка и отправка образа custom-nginx

### 🔹 Шаги выполнения:

1. Установлен Docker и Docker Compose на Arch Linux:
   ```bash
   sudo pacman -S docker docker-compose-plugin
   sudo systemctl enable --now docker
   sudo usermod -aG docker $USER
   ```

2. Создан `Dockerfile`:
   ```dockerfile
   FROM nginx:1.29.0
   COPY index.html /usr/share/nginx/html/index.html
   RUN chmod 644 /usr/share/nginx/html/index.html
   ```

3. Собран образ:
   ```bash
   docker build -t myth3916/custom-nginx:1.0.0 .
   ```

4. Отправлен в Docker Hub:
   ```bash
   docker login
   docker push myth3916/custom-nginx:1.0.0
   ```

### 🔹 Скриншоты:

![Docker Hub репозиторий](./screenshots/task1-dockerhub.png)
*Рисунок 1: Образ custom-nginx:1.0.0 в Docker Hub*

---

## ✅ Задача 2: Запуск и проверка контейнера

### 🔹 Команды:

```bash
# Запуск контейнера
docker run -d \
  --name Oleg-custom-nginx-t2 \
  -p 127.0.0.1:8080:80 \
  myth3916/custom-nginx:1.0.0

# Переименование
docker rename Oleg-custom-nginx-t2 custom-nginx-t2

# Проверка
curl http://127.0.0.1:8080
```

### 🔹 Исправление прав доступа (403 Forbidden):

```bash
docker exec -u root custom-nginx-t2 chmod 644 /usr/share/nginx/html/index.html
```

### 🔹 Скриншоты:

* Вывод команд запуска, переименования и проверки*
![Запуск и проверка контейнера](./screenshots/task2-run-rename.png)
![Запуск и проверка контейнера](./screenshots/task2-verify-check.png)
![Запуск и проверка контейнера](./screenshots/task2-curl-check.png)

---

## 🔄 Задача 3: Работа с потоками и конфигурацией (в процессе)

### 🔹 Подключение к контейнеру:

```bash
# Подключение к stdout/stdin
docker attach custom-nginx-t2
# Нажатие Ctrl-C останавливает контейнер, т.к. сигнал передаётся основному процессу (nginx)

# Перезапуск
docker start custom-nginx-t2

# Вход в интерактивный терминал
docker exec -it custom-nginx-t2 bash
```

### 🔹 Почему контейнер остановился после Ctrl-C?

> При использовании `docker attach` мы подключаемся к основному процессу контейнера. 
> Нажатие `Ctrl-C` отправляет сигнал `SIGINT`, который nginx обрабатывает как команду на завершение работы.
> Поэтому контейнер останавливается.

### 🔹 Скриншоты:

![Attach и restart](./screenshots/task3-attach.png)

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

# Очистка
docker system prune -a
```

---

> 💡 **Примечание:** Все скриншоты сохранены в папке `./screenshots/`
```

---

## 📁 Структура папки проекта

Создай такую структуру:

```
~/github/docker-task1/
├── README.md           # Этот файл
├── Dockerfile          # Твой Dockerfile
├── index.html          # Твой index.html
└── screenshots/        # Папка для скриншотов
    ├── task1-dockerhub.png
    ├── task2-run-check.png
    └── task3-attach.png
```

### 🔹 Создай папку для скриншотов:
```bash
mkdir -p ~/github/docker-task1/screenshots
```
