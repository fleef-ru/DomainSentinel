# DomainSentinel 

# Migrated to [TelegramBots / DomainSentinel GitLab](https://fleef.ru/telegrambots/domainsentinel) - There will be no updates here
# Перешёл на [TelegramBots/DomainSentinel GitLab](https://fleef.ru/telegrambots/domainsentinel) - обновлений здесь не будет


**DomainSentinel** — это Telegram-бот для мониторинга изменений в списке доменов. Бот автоматически отслеживает добавление и удаление доменов, отправляя уведомления подписанным пользователям. Проект разработан для обеспечения актуальной информации о доменах и упрощения управления ими.

## Содержание

- [Особенности](#особенности)
- [Технологии](#технологии)
- [Установка](#установка)
  - [Предварительные требования](#предварительные-требования)
  - [Установка с использованием Docker](#установка-с-использованием-docker)
  - [Установка из исходного кода](#установка-из-исходного-кода)
    - [На Debian/Linux](#на-debianlinux)
    - [На Windows](#на-windows)
- [Конфигурация](#конфигурация)
- [Использование](#использование)
  - [Команды бота](#команды-бота)
- [Логирование](#логирование)
- [Часто задаваемые вопросы (FAQ)](#часто-задаваемые-вопросы-faq)
- [Вклад](#вклад)
- [Лицензия](#лицензия)
- [Контакты](#контакты)

## Особенности

- **Автоматический мониторинг доменов:** Отслеживает изменения в списке доменов и уведомляет пользователей.
- **Уведомления пользователям:** Отправка уведомлений о добавленных и удалённых доменах.
- **Поддержка нескольких методов установки:** Возможность развертывания через Docker или из исходного кода на различных операционных системах.
- **Рейт-лимитинг:** Ограничение частоты использования команд для предотвращения спама.
- **Кэширование WHOIS-запросов:** Снижение количества повторных запросов и ускорение работы бота.

## Технологии

- **Python 3.8+**
- **Pyrogram** — библиотека для взаимодействия с Telegram API.
- **aiohttp** — асинхронные HTTP-запросы.
- **aiosqlite** — асинхронный интерфейс для SQLite.
- **APScheduler** — планировщик задач.
- **python-whois** — библиотека для получения WHOIS-данных.
- **Tenacity** — библиотека для повторных попыток при ошибках.
- **aiofiles** — асинхронная работа с файлами.
- **Docker** — контейнеризация приложения.
- **systemd** — управление сервисами на Linux.

## Установка

### Предварительные требования

- **Операционная система:**
  - [Debian 10+](https://www.debian.org/)
  - [Windows 10+](https://www.microsoft.com/windows)
  - **Docker** (опционально, для установки через Docker)
- **Python 3.8+** (для установки из исходного кода)
- **Git** (опционально, для клонирования репозитория)

### Установка с использованием Docker

Docker упрощает развертывание и управление приложением, обеспечивая изоляцию окружения. Вы можете использовать Docker для локальной установки без необходимости публикации образа в удалённый репозиторий.

#### Шаг 1: Установка Docker

Если Docker ещё не установлен на вашем компьютере, следуйте официальной [инструкции по установке Docker](https://docs.docker.com/get-docker/) для вашей операционной системы.

#### Шаг 2: Создание Dockerfile

Убедитесь, что в корневой директории вашего проекта (`DomainSentinel`) находится файл `Dockerfile` со следующим содержимым:

```dockerfile
# Используем официальный Python-образ
FROM python:3.8-slim

# Установим рабочую директорию
WORKDIR /app

# Копируем файлы проекта
COPY . /app

# Создаём виртуальное окружение
RUN python -m venv venv

# Активируем виртуальное окружение и устанавливаем зависимости
RUN /bin/bash -c "source venv/bin/activate && pip install --upgrade pip && pip install -r requirements.txt"

# Устанавливаем переменные окружения
ENV PATH="/app/venv/bin:$PATH"

# Запускаем бота
CMD ["python", "main.py"]
```

#### Шаг 3: Создание файла `.env`

Создайте файл `.env` в корневой директории проекта и добавьте необходимые переменные:

```dotenv
API_ID=YOUR_API_ID
API_HASH=YOUR_API_HASH
BOT_TOKEN=YOUR_BOT_TOKEN
CHECK_INTERVAL=60
SOURCE_URL=""  # Любой удалённый ресурс с доменами на новой строке
WHOIS_TIMEOUT=10
DATABASE_PATH=domains.db
USERS_FILE=users.json
ADMIN_USER_IDS=[1234567890]  # Замените на реальные Telegram user_id администраторов
```

**Примечание:** Замените `YOUR_API_ID`, `YOUR_API_HASH`, `YOUR_BOT_TOKEN` и `ADMIN_USER_IDS` на свои собственные значения.

#### Шаг 4: Сборка Docker-образа

Перейдите в директорию проекта и выполните сборку образа:

```bash
docker build -t domainsentinel:latest .
```

#### Шаг 5: Запуск Контейнера

Запустите контейнер:

```bash
docker run -d --name domainsentinel \
  --env-file .env \
  domainsentinel:latest
```

**Объяснение Команд:**

- `-d` — запуск контейнера в фоновом режиме (detached).
- `--name domainsentinel` — имя контейнера.
- `--env-file .env` — передача переменных окружения из файла `.env`.

#### Шаг 6: Управление Контейнером

- **Проверка статуса контейнера:**

  ```bash
  docker ps -a
  ```

- **Просмотр логов контейнера:**

  ```bash
  docker logs -f domainsentinel
  ```

- **Перезапуск контейнера:**

  ```bash
  docker restart domainsentinel
  ```

- **Остановка контейнера:**

  ```bash
  docker stop domainsentinel
  ```

#### Дополнительно: Использование Docker Compose (Опционально)

Если вы предпочитаете использовать Docker Compose для управления сервисами, создайте файл `docker-compose.yml`:

```yaml
version: '3.8'

services:
  domainsentinel:
    build: .
    container_name: domainsentinel
    env_file:
      - .env
    restart: always
```

Запустите сервис:

```bash
docker-compose up -d
```

### Установка из исходного кода

Если вы предпочитаете установить приложение напрямую на вашу систему без использования Docker, следуйте следующим инструкциям.

#### На Debian/Linux

##### Шаг 1: Подготовка окружения

1. **Установка необходимых пакетов:**

   ```bash
   sudo apt update
   sudo apt install -y python3 python3-pip python3-venv git
   ```

2. **Клонирование репозитория:**

   ```bash
   git clone https://github.com/fleef-me/DomainSentinel.git
   cd DomainSentinel
   ```

##### Шаг 2: Создание и активация виртуального окружения

```bash
python3 -m venv venv
source venv/bin/activate
```

##### Шаг 3: Установка зависимостей

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

##### Шаг 4: Настройка systemd

1. **Создание сервисного файла:**

   ```bash
   sudo nano /etc/systemd/system/domainsentinel.service
   ```

2. **Добавление конфигурации:**

   ```ini
   [Unit]
   Description=DomainSentinel Telegram Bot
   After=network.target

   [Service]
   User=your_username  # Замените на ваше имя пользователя
   Group=your_username # Замените на вашу группу пользователя
   WorkingDirectory=/path/to/DomainSentinel  # Замените на путь к проекту
   Environment="PATH=/path/to/DomainSentinel/venv/bin"  # Замените на путь к виртуальному окружению
   ExecStart=/path/to/DomainSentinel/venv/bin/python main.py  # Замените на путь к main.py

   # Перезапуск при сбое
   Restart=always
   RestartSec=5

   # Логирование
   StandardOutput=syslog
   StandardError=syslog
   SyslogIdentifier=domainsentinel

   [Install]
   WantedBy=multi-user.target
   ```

   **Примечание:** Замените `your_username` и `/path/to/DomainSentinel` на соответствующие значения вашей системы.

3. **Сохранение и выход:**

   Нажмите `CTRL + O` для сохранения и `CTRL + X` для выхода из редактора.

4. **Перезагрузка systemd и запуск сервиса:**

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start domainsentinel.service
   sudo systemctl enable domainsentinel.service
   ```

5. **Проверка статуса сервиса:**

   ```bash
   sudo systemctl status domainsentinel.service
   ```

6. **Просмотр логов:**

   ```bash
   sudo journalctl -u domainsentinel.service -f
   ```

#### На Windows

Для установки на Windows рекомендуется использовать Docker, так как нативная поддержка systemd отсутствует. Однако, если вы хотите установить из исходного кода, следуйте этим шагам:

##### Шаг 1: Установка Python

1. **Скачайте Python 3.8+** с [официального сайта](https://www.python.org/downloads/windows/).
2. **Установите Python**, убедившись, что выбрали опцию "Add Python to PATH".

##### Шаг 2: Установка Git (опционально)

1. **Скачайте Git** с [официального сайта](https://git-scm.com/download/win).
2. **Установите Git** следуя инструкциям установщика.

##### Шаг 3: Клонирование репозитория

Откройте **PowerShell** или **Командную строку** и выполните:

```powershell
git clone https://github.com/fleef-me/DomainSentinel.git
cd DomainSentinel
```

##### Шаг 4: Создание и активация виртуального окружения

```powershell
python -m venv venv
.\venv\Scripts\activate
```

##### Шаг 5: Установка зависимостей

```powershell
pip install --upgrade pip
pip install -r requirements.txt
```

##### Шаг 6: Запуск бота

```powershell
python main.py
```

**Примечание:** Для автоматического запуска бота при старте системы рекомендуется использовать Планировщик заданий Windows или сторонние инструменты, такие как [NSSM (Non-Sucking Service Manager)](https://nssm.cc/).

## Конфигурация

Файл `.env` содержит все необходимые переменные конфигурации. Убедитесь, что вы правильно настроили его перед запуском приложения.

### Пример `.env`:

```dotenv
API_ID=YOUR_API_ID
API_HASH=YOUR_API_HASH
BOT_TOKEN=YOUR_BOT_TOKEN
CHECK_INTERVAL=60
SOURCE_URL=""  # Любой удалённый ресурс с доменами на новой строке
WHOIS_TIMEOUT=10
DATABASE_PATH=domains.db
USERS_FILE=users.json
ADMIN_USER_IDS=[1234567890]  # Замените на реальные Telegram user_id администраторов
```

**Описание переменных:**

- `API_ID` и `API_HASH`: Получите из [Telegram API](https://my.telegram.org/auth).
- `BOT_TOKEN`: Токен вашего бота, полученный от [BotFather](https://t.me/botfather).
- `CHECK_INTERVAL`: Интервал проверки изменений в минутах.
- `SOURCE_URL`: URL источника списка доменов. Если используется локальный файл, настройте `LOCAL_SOURCE` и `SOURCE_PATH` в `config.py`.
- `WHOIS_TIMEOUT`: Таймаут для WHOIS-запросов в секундах.
- `DATABASE_PATH`: Путь к базе данных SQLite.
- `USERS_FILE`: Файл для хранения списка пользователей.
- `ADMIN_USER_IDS`: Список `user_id` администраторов.

**Примечание:** Все пути и пользовательские данные в файле `.env` заменены на примеры. Убедитесь, что вы заменили их на свои собственные значения.

## Использование

После установки и настройки вы можете управлять ботом через Telegram, используя следующие команды.

### Команды бота

- **/start** — Подписаться на уведомления о изменениях в доменах.
- **/stop** — Отписаться от уведомлений.
- **/status** — Проверить количество подписанных пользователей.
- **/check** — Запустить ручную проверку изменений (доступно администраторам).
- **/check_test** — Инициировать тестовую проверку системы оповещений (доступно администраторам).
- **/add_domain <домен>** — Добавить домен в список (доступно администраторам).
- **/remove_domain <домен>** — Удалить домен из списка (доступно администраторам).

#### Пример использования команд

**Добавление домена:**

```plaintext
/add_domain example-test.com
```

**Удаление домена:**

```plaintext
/remove_domain example-test.com
```

**Запуск тестовой проверки:**

```plaintext
/check_test
```

## Логирование

Все логи приложения сохраняются в системный журнал и могут быть просмотрены с помощью соответствующих инструментов.

### Просмотр логов на Linux

```bash
sudo journalctl -u domainsentinel.service -f
```

### Просмотр логов Docker

```bash
docker logs -f domainsentinel
```

## Часто задаваемые вопросы (FAQ)

### 1. Как начать использовать DomainSentinel?

**Ответ:**
Чтобы начать использовать DomainSentinel, выполните следующие шаги:

1. **Установка:** Следуйте инструкциям по [установке](#установка) для вашей операционной системы (Docker, Debian/Linux или Windows).
2. **Настройка:** Настройте файл `.env`, указав необходимые переменные конфигурации, такие как `API_ID`, `API_HASH`, `BOT_TOKEN` и другие.
3. **Запуск:** Запустите бота через Docker или systemd в зависимости от выбранного метода установки.
4. **Подписка:** В Telegram отправьте команду `/start` вашему боту, чтобы подписаться на уведомления.

### 2. Как добавить или удалить домен из списка мониторинга?

**Ответ:**
Только администраторы могут добавлять или удалять домены. Для этого используйте следующие команды в Telegram:

- **Добавление домена:**

  ```plaintext
  /add_domain example.com
  ```

- **Удаление домена:**

  ```plaintext
  /remove_domain example.com
  ```

**Примечание:** Замените `example.com` на нужный вам домен.

### 3. Почему я не получаю уведомления?

**Ответ:**
Возможные причины отсутствия уведомлений:

1. **Не подписаны на уведомления:** Убедитесь, что вы отправили команду `/start` вашему боту.
2. **Бот не работает:** Проверьте статус бота и его логи.
   - **Docker:** `docker logs -f domainsentinel`
   - **systemd:** `sudo journalctl -u domainsentinel.service -f`
3. **Изменений в списке доменов нет:** Если в списке доменов не произошло изменений, уведомлений не будет.
4. **Проблемы с подключением к Telegram API:** Проверьте корректность `API_ID`, `API_HASH` и `BOT_TOKEN` в файле `.env`.

### 4. Что делать, если бот не запускается?

**Ответ:**
1. **Проверьте логи:**
   - **Docker:** `docker logs -f domainsentinel`
   - **systemd:** `sudo journalctl -u domainsentinel.service -f`
2. **Убедитесь, что все зависимости установлены корректно.**
3. **Проверьте файл `.env` на наличие ошибок и корректность значений переменных.**
4. **Проверьте доступность интернета на сервере.**

### 5. Как обновить бот до последней версии?

#### Для Docker:

1. **Перейдите в директорию проекта:**

   ```bash
   cd /path/to/DomainSentinel
   ```

2. **Соберите новый Docker-образ:**

   ```bash
   docker build -t domainsentinel:latest .
   ```

3. **Перезапустите контейнер:**

   ```bash
   docker stop domainsentinel
   docker rm domainsentinel
   docker run -d --name domainsentinel \
     --env-file .env \
     domainsentinel:latest
   ```

#### Для установки из исходного кода:

1. **Перейдите в директорию проекта:**

   ```bash
   cd /path/to/DomainSentinel
   ```

2. **Обновите код:**

   ```bash
   git pull origin main
   ```

3. **Активируйте виртуальное окружение и обновите зависимости:**

   ```bash
   source venv/bin/activate
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

4. **Перезапустите сервис:**

   ```bash
   sudo systemctl restart domainsentinel.service
   ```

### 6. Как изменить настройки бота?

**Ответ:**
1. **Откройте файл `.env`** в корневой директории проекта.
2. **Измените необходимые переменные**, такие как `CHECK_INTERVAL`, `SOURCE_URL`, `WHOIS_TIMEOUT` и другие.
3. **Сохраните изменения.**
4. **Перезапустите бот** для применения новых настроек:
   - **Docker:** `docker restart domainsentinel`
   - **systemd:** `sudo systemctl restart domainsentinel.service`

### 7. Как добавить новых администраторов?

**Ответ:**
1. **Откройте файл `.env`.**
2. **Добавьте новые `user_id`** в список `ADMIN_USER_IDS`. Пример:

   ```dotenv
   ADMIN_USER_IDS=[1234567890, 0987654321]  # Замените на реальные Telegram user_id администраторов
   ```

3. **Сохраните файл.**
4. **Перезапустите бот** для применения изменений:
   - **Docker:** `docker restart domainsentinel`
   - **systemd:** `sudo systemctl restart domainsentinel.service`

### 8. Что делать, если меня удалили из списка администраторов?

**Ответ:**
Если вы больше не являетесь администратором, обратитесь к текущим администраторам проекта, чтобы они добавили вас обратно в список `ADMIN_USER_IDS` в файле `.env`. После этого перезапустите бот.

### 9. Как просмотреть логи бота?

**Ответ:**
- **Для Docker:**

  ```bash
  docker logs -f domainsentinel
  ```

- **Для systemd:**

  ```bash
  sudo journalctl -u domainsentinel.service -f
  ```

### 10. Какие зависимости нужны для работы бота?

**Ответ:**
Основные зависимости указаны в файле `requirements.txt`. Они включают:

- Pyrogram
- aiohttp
- aiosqlite
- APScheduler
- python-whois
- tenacity
- aiofiles

### 11. Можно ли использовать бота без Docker?

**Ответ:**
Да, вы можете установить и запустить бота из исходного кода, следуя инструкциям в разделе [Установка из исходного кода](#установка-из-исходного-кода).

### 12. Как настроить автоматический запуск бота при старте системы?

#### На Debian/Linux с systemd:

Сервис уже настроен для автоматического запуска при загрузке системы. Убедитесь, что сервис включён:

```bash
sudo systemctl enable domainsentinel.service
```

#### На Windows:

Используйте Планировщик заданий или инструменты вроде [NSSM (Non-Sucking Service Manager)](https://nssm.cc/) для создания службы, которая будет запускать бота при старте системы.

### 13. Как отключить бота?

**Ответ:**
- **Для Docker:**

  ```bash
  docker stop domainsentinel
  ```

- **Для systemd:**

  ```bash
  sudo systemctl stop domainsentinel.service
  ```

### 14. Как изменить интервал проверки доменов?

**Ответ:**
1. **Откройте файл `.env`.**
2. **Измените значение переменной `CHECK_INTERVAL`.** Например, для интервала в 30 минут:

   ```dotenv
   CHECK_INTERVAL=30
   ```

3. **Сохраните файл.**
4. **Перезапустите бот** для применения новых настроек:
   - **Docker:** `docker restart domainsentinel`
   - **systemd:** `sudo systemctl restart domainsentinel.service`

### 15. Что делать, если бот не подключается к Telegram API?

**Ответ:**
1. **Проверьте корректность переменных `API_ID`, `API_HASH` и `BOT_TOKEN`** в файле `.env`.
2. **Убедитесь, что у бота нет ограничений или блокировок в Telegram.**
3. **Проверьте подключение к интернету на сервере.**
4. **Просмотрите логи** для обнаружения ошибок подключения.

### 16. Как изменить список доменов вручную?

**Ответ:**
Администраторы могут добавлять или удалять домены с помощью команд `/add_domain` и `/remove_domain`. Также можно напрямую редактировать файл `domains.lst`, если используется локальный источник, но это не рекомендуется, так как может привести к несоответствиям с базой данных.

### 17. Как добавить пользователей для подписки?

**Ответ:**
Пользователи могут самостоятельно подписаться на уведомления, отправив боту команду `/start` в Telegram. Чтобы отписаться, они могут использовать команду `/stop`.

### 18. Что делать, если команда бота не работает?

**Ответ:**
1. **Проверьте подключение бота к интернету.**
2. **Убедитесь, что бот запущен и работает корректно.**
3. **Проверьте логи бота** на наличие ошибок.
4. **Убедитесь, что команды вводятся правильно.** Например, команды должны начинаться с `/` и быть отправлены в приватном чате с ботом.

### 19. Как изменить `ADMIN_USER_IDS` после запуска бота?

**Ответ:**
1. **Откройте файл `.env`.**
2. **Измените список `ADMIN_USER_IDS`, добавив или удалив нужные `user_id`.**
3. **Сохраните файл.**
4. **Перезапустите бот** для применения изменений:
   - **Docker:** `docker restart domainsentinel`
   - **systemd:** `sudo systemctl restart domainsentinel.service`

### 20. Где хранится база данных и файлы пользователей?

**Ответ:**
- **База данных:** Переменная `DATABASE_PATH` в файле `.env` указывает путь к базе данных SQLite. По умолчанию это `domains.db`.
- **Файл пользователей:** Переменная `USERS_FILE` в файле `.env` указывает путь к файлу `users.json`, где хранится список подписанных пользователей.

## Вклад

Мы приветствуем вклады в проект! Если вы хотите помочь, следуйте следующим шагам:

1. **Форкните репозиторий.**
2. **Создайте новую ветку:**

   ```bash
   git checkout -b feature/your-feature
   ```

3. **Внесите изменения и закоммитьте их:**

   ```bash
   git commit -m "Добавлена новая фича"
   ```

4. **Запушьте ветку:**

   ```bash
   git push origin feature/your-feature
   ```

5. **Создайте Pull Request.**

## Лицензия

Этот проект лицензирован под лицензией [MIT](LICENSE).

## Контакты

Если у вас есть вопросы или предложения, свяжитесь с нами:

- **Email:** <for request>
- **Telegram:** [@fleef](https://t.me/fleef)

---

## Дополнительные Разделы (по желанию)

### Скриншоты
![screen1.jpg](source%2Fscreen1.jpg)
![screen2.jpg](source%2Fscreen2.jpg)
