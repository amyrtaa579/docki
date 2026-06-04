# 🤖 Установка и запуск MAX-бота (TPGK Client)

MAX-бот для Томского промышленно-гуманитарного колледжа (ТПГК). Взаимодействует с FastAPI-бэкендом.

> ⚠️ **Важное отличие от остальных сервисов:** MAX-бот — это **клиент**. Он сам НЕ слушает никаких портов, а только делает исходящие запросы к твоему API и к серверам MAX. Поэтому **открывать порты в брандмауэре для него не нужно**.

---

## 📦 1. Установка Node.js

Бот написан на TypeScript и работает на Node.js.

```powershell
# Скачайте Node.js 20 LTS (официальный установщик)
Invoke-WebRequest -Uri "https://nodejs.org/dist/v20.18.1/node-v20.18.1-x64.msi" -OutFile "C:\nodejs-installer.msi"

# Запустите установщик (можно через GUI или тихо)
Start-Process msiexec.exe -ArgumentList "/i C:\nodejs-installer.msi /qn" -Wait
```

После установки **обязательно перезапустите PowerShell**, чтобы подхватился PATH.

### Проверьте установку

```powershell
# Перезапустите PowerShell перед этой командой!
node -v
# Должно вывести: v20.18.1

npm -v
# Должно вывести: 10.x.x
```

---

## 📥 2. Клонирование репозитория

```powershell
# Перейдите в папку проектов
cd C:\

# Клонируйте репозиторий клиента
git clone https://github.com/amyrtaa579/tpgk-client.git

# Перейдите в папку проекта
cd C:\tpgk-client
```

---

## ⚙️ 3. Настройка переменных окружения (.env)

```powershell
# Создайте .env файл
notepad .env
```

Вставьте следующее содержимое и **сохраните**:

```env
# Токен бота в мессенджере MAX (получите у @MasterBot в MAX)
BOT_TOKEN=your_bot_token_here

# URL API бэкенда (ваш FastAPI на этом же сервере)
API_URL=http://72.56.6.8:8000

# API ключ для авторизации (если настроен в FastAPI)
API_KEY=your_api_key_here

# Таймаут запросов к API (мс)
API_TIMEOUT=30000

# Уровень логирования
LOG_LEVEL=info

# ID администраторов бота (через запятую)
ADMIN_IDS=123456789
```

> 💡 **Важно:** `API_URL` должен указывать на **твой FastAPI**. Если бэкенд и бот на одном сервере — можно использовать `http://localhost:8000/api/v1` или `http://72.56.6.8:8000/api/v1`.

---

## 📦 4. Установка зависимостей

```powershell
# Установите все зависимости из package.json
npm ci
```

> `npm ci` — это "чистая" установка строго по `package-lock.json`. Она быстрее и надёжнее, чем `npm install`.

---

## 🔨 5. Сборка проекта

```powershell
# Скомпилируйте TypeScript в JavaScript
npm run build
```

После этого появится папка `dist` с готовым к запуску кодом.

---

## ▶️ 6. Первый запуск (для проверки)

Перед тем как делать службу, запустите бота вручную и убедитесь, что всё работает:

```powershell
# Запуск в режиме продакшена
npm start
```

**Что должно произойти:**
- В консоли появятся логи запуска
- Бот подключится к MAX API
- В логах будет что-то вроде: `Bot started successfully`

**Проверьте в MAX:**
1. Найдите своего бота в мессенджере MAX
2. Напишите ему `/start`
3. Должно появиться главное меню с кнопками: 🏫 О колледже, 🎓 Специальности и т.д.

Если всё работает — нажмите `Ctrl+C` чтобы остановить. Теперь сделаем из него службу.

---

## 🛠 7. Установка как службы Windows (через NSSM)

Чтобы бот работал 24/7 и перезапускался после перезагрузки сервера.

### 7.1. Создайте скрипт запуска

```powershell
# Создайте bat-файл для запуска бота
@'
@echo off
cd /d C:\tpgk-client
set PATH=C:\Program Files\nodejs;%PATH%
npm start
'@ | Out-File -FilePath "C:\start-maxbot.bat" -Encoding ASCII
```

> ⚠️ Службы Windows не наследуют PATH из пользовательской сессии, поэтому прописываем путь к Node.js вручную.

### 7.2. Установите службу через NSSM

```powershell
# Если NSSM ещё не установлен (используем тот, что ставили для FastAPI)
# Если нет — скачайте:
if (-not (Test-Path "C:\nssm")) {
    Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
    Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"
}

# Установите MAX-бот как службу
C:\nssm\nssm-2.24\win64\nssm.exe install MaxBot "C:\start-maxbot.bat"
C:\nssm\nssm-2.24\win64\nssm.exe set MaxBot AppDirectory "C:\tpgk-client"
C:\nssm\nssm-2.24\win64\nssm.exe set MaxBot DisplayName "TPGK MAX Bot"
C:\nssm\nssm-2.24\win64\nssm.exe set MaxBot Description "MAX messenger bot for TPGK college"

# Запустите службу
C:\nssm\nssm-2.24\win64\nssm.exe start MaxBot
```

---

## ✅ 8. Проверка работы

```powershell
# Проверьте статус службы
Get-Service MaxBot

# Проверьте, что процесс node.exe запущен
Get-Process node -ErrorAction SilentlyContinue

# Проверьте, что бот отвечает в MAX
# (просто напишите боту /start в мессенджере)
```

---

## 🔧 9. Управление службой

```powershell
# Остановить бота
C:\nssm\nssm-2.24\win64\nssm.exe stop MaxBot

# Перезапустить бота (после обновления кода)
C:\nssm\nssm-2.24\win64\nssm.exe restart MaxBot

# Удалить службу (если нужно)
C:\nssm\nssm-2.24\win64\nssm.exe remove MaxBot confirm

# Открыть GUI для редактирования (включая логи)
C:\nssm\nssm-2.24\win64\nssm.exe edit MaxBot
```

---

## 🔄 10. Обновление бота

Когда вышли новые изменения в репозитории:

```powershell
# Перейдите в папку проекта
cd C:\tpgk-client

# Остановите службу
C:\nssm\nssm-2.24\win64\nssm.exe stop MaxBot

# Скачайте обновления
git pull

# Установите новые зависимости (если появились)
npm ci

# Пересоберите проект
npm run build

# Запустите обратно
C:\nssm\nssm-2.24\win64\nssm.exe start MaxBot
```

---

## 📢 11. Массовая рассылка (опционально)

Если нужно разослать сообщение всем пользователям бота:

```powershell
# Перейдите в папку проекта
cd C:\tpgk-client

# Запустите скрипт рассылки
npx ts-node broadcast.ts
```

> ⚠️ Перед рассылкой убедитесь, что в `known_chats.json` есть сохранённые ID чатов. Они автоматически собираются, когда пользователи пишут боту.

---

## 📋 Итого

| Компонент | Статус | Примечание |
|-----------|--------|------------|
| Node.js | Установлен | v20.x LTS |
| Код бота | `C:\tpgk-client` | Клонирован из GitHub |
| .env | Настроен | Указывает на `http://72.56.6.8:8000/api/v1` |
| Служба MaxBot | Запущена через NSSM | Автозапуск при загрузке Windows |
| Порты в брандмауэре | **НЕ нужны** | Бот работает как клиент |

---

## 🎯 Финальная проверка всей системы

Убедитесь, что все сервисы работают вместе:

| Сервис | URL | Как проверить |
|--------|-----|---------------|
| PostgreSQL | `72.56.6.8:5432` | Подключиться через pgAdmin/DBeaver |
| MinIO Console | `http://72.56.6.8:9001` | Открыть в браузере |
| Redis | `72.56.6.8:6379` | `redis-cli -h 72.56.6.8 ping` |
| FastAPI Swagger | `http://72.56.6.8:8000/docs` | Открыть в браузере |
| **MAX-бот** | В мессенджере MAX | Написать боту `/start` |

Если бот в MAX отвечает, показывает меню и подгружает данные из API — **всё работает!** 🎉

---

## 🚨 Если бот не отвечает

**Шаг 1. Проверьте статус службы:**
```powershell
Get-Service MaxBot
```

**Шаг 2. Посмотрите логи** (NSSM пишет их в файлы):
```powershell
C:\nssm\nssm-2.24\win64\nssm.exe edit MaxBot
```
Во вкладке **"I/O"** увидите пути к `stdout` и `stderr` логам. Откройте их блокнотом.

**Шаг 3. Частые проблемы:**

| Проблема                 | Решение                                               |
| ------------------------ | ----------------------------------------------------- |
| `Cannot find module`     | Запустите `npm ci` заново                             |
| `ECONNREFUSED` к API_URL | Проверьте, что FastAPI запущен: `Get-Service FastAPI` |
| `401 Unauthorized` к API | Проверьте `API_KEY` в `.env`                          |
| Бот не отвечает в MAX    | Проверьте `BOT_TOKEN` — возможно, он истёк            |
| `node не найден`         | Проверьте PATH в `start-maxbot.bat`                   |
