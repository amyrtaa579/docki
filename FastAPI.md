# 🚀 Полная установка FastAPI (Anmicius API) с нуля

FastAPI backend для Томского промышленно-гуманитарного колледжа (ТПГК).

> ⚠️ **Важно:** В этой инструкции мы установим **всё с нуля** — от интерпретатора Python до запуска службы.

---

## 🐍 1. Установка Python 3.11.9

```powershell
# Скачайте установщик Python 3.11.9
Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.11.9/python-3.11.9-amd64.exe" -OutFile "C:\python-installer.exe"

# Запустите установщик (откроется окно)
Start-Process "C:\python-installer.exe"
```

### ⚠️ ВАЖНО в установщике:

Когда откроется окно установщика:

1. ✅ **Обязательно поставьте галочку** внизу: `Add python.exe to PATH`
2. Нажмите **"Install Now"** (или "Customize installation" → убедитесь, что pip тоже выбран)
3. Дождитесь окончания установки

### Проверьте установку

**Обязательно перезапустите PowerShell** после установки, иначе PATH не подхватится!

```powershell
python --version
# Должно вывести: Python 3.11.9

pip --version
# Должно вывести: pip 24.x.x from ... (python 3.11)
```

---

## 📦 2. Установка Git

```powershell
# Скачайте установщик Git
Invoke-WebRequest -Uri "https://github.com/git-for-windows/git/releases/download/v2.47.1.windows.1/Git-2.47.1-64-bit.exe" -OutFile "C:\git-installer.exe"

# Запустите установщик
Start-Process "C:\git-installer.exe"
```

В установщике можно просто нажимать **"Next"** везде — настройки по умолчанию подходят.

### Проверьте установку

**Перезапустите PowerShell!**

```powershell
git --version
# Должно вывести: git version 2.47.1
```

---

## 📥 3. Клонирование репозитория

```powershell
# Перейдите в корень диска
cd C:\

# Клонируйте репозиторий
git clone https://github.com/amyrtaa579/tpgk-server.git

# Перейдите в папку проекта
cd C:\tpgk-server
```

---

## 📦 4. Создание виртуального окружения

```powershell
# Создайте виртуальное окружение в папке venv
python -m venv venv

# Активируйте его
.\venv\Scripts\activate

# После активации в начале строки появится (venv)
# Например: (venv) PS C:\tpgk-server>
```

> 💡 **Зачем это нужно:** виртуальное окружение изолирует зависимости проекта. Все библиотеки установятся в `C:\tpgk-server\venv`, а не в систему.

---

## 📚 5. Установка зависимостей

```powershell
# Обновите pip
python -m pip install --upgrade pip

# Установите все зависимости из requirements.txt
pip install -r requirements.txt
```

> ⏳ Установка может занять 2-5 минут в зависимости от скорости интернета.

---

## ⚙️ 6. Настройка .env

```powershell
# Скопируйте пример файла окружения
Copy-Item .env.example .env

# Откройте .env в блокноте и отредактировать
notepad .env
```

---

## 🗄 7. Создание базы данных и применение миграций

### 7.1. Создайте базу данных `tpgk` в PostgreSQL

```powershell
# Подключитесь к PostgreSQL
& "C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres

# В открывшейся консоли psql выполните:
CREATE DATABASE tpgk;
\q
```

### 7.2. Примените миграции Alembic

```powershell
# Убедитесь, что виртуальное окружение активировано (venv)
# Если нет — активируйте:
.\venv\Scripts\activate

# Примените все миграции
alembic upgrade head
```

Должно появиться что-то вроде:
```
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Running upgrade -> 001_initial, initial migration
...
```

---

## 👤 8. Создание администратора

```powershell
# Запустите скрипт инициализации админа
python init-admin.py
```

Следуйте инструкциям в консоли:
- Введите email администратора
- Введите пароль
- Введите имя

---

## ▶️ 9. Первый запуск (для проверки)

Перед тем как делать службу, запустите сервер вручную и убедитесь, что всё работает:

```powershell
# Запустите FastAPI через uvicorn
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

**Что должно произойти:**
- В консоли появится: `Uvicorn running on http://0.0.0.0:8000`
- Откройте в браузере: `http://localhost:8000/docs`
- Должна открыться страница **Swagger UI** с документацией API

Если всё работает — нажмите `Ctrl+C` чтобы остановить. Теперь сделаем из него службу.

---

## 🛠 10. Установка как службы Windows (через NSSM)

Чтобы FastAPI работал 24/7 и перезапускался после перезагрузки сервера.

### 10.1. Создайте скрипт запуска

```powershell
# Создайте bat-файл для запуска FastAPI
@'
@echo off
cd /d C:\tpgk-server
call venv\Scripts\activate
uvicorn app.main:app --host 0.0.0.0 --port 8000
'@ | Out-File -FilePath "C:\start-fastapi.bat" -Encoding ASCII
```

### 10.2. Установите службу через NSSM

> 💡 Если NSSM уже установлен (ставили для MinIO/Redis) — используйте существующий. Если нет — скачайте:

```powershell
# Если NSSM ещё нет
if (-not (Test-Path "C:\nssm")) {
    Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
    Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"
}

# Установите FastAPI как службу
C:\nssm\nssm-2.24\win64\nssm.exe install FastAPI "C:\start-fastapi.bat"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI AppDirectory "C:\tpgk-server"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI DisplayName "FastAPI Server"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI Description "FastAPI Backend for TPGK"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI Start SERVICE_AUTO_START

# Запустите службу
C:\nssm\nssm-2.24\win64\nssm.exe start FastAPI
```

---

## 🔥 11. Открытие порта в брандмауэре

```powershell
# Создайте правило для входящих подключений на порт 8000
New-NetFirewallRule -DisplayName "FastAPI" -Direction Inbound -LocalPort 8000 -Protocol TCP -Action Allow
```

### Проверьте правило

```powershell
Get-NetFirewallRule -DisplayName "FastAPI"
```

---

## ✅ 12. Проверка работы

```powershell
# Проверьте статус службы
Get-Service FastAPI
# Должно быть: Status: Running

# Проверьте, что порт 8000 слушается
netstat -ano | findstr "8000"
# Должна быть строка: TCP  0.0.0.0:8000  ...  LISTENING

# Проверьте локально
Invoke-RestMethod -Uri "http://localhost:8000/docs"

# Проверьте с удалённой машины (со своего домашнего ПК)
Test-NetConnection -ComputerName 72.56.6.8 -Port 8000
# Должно быть: TcpTestSucceeded : True
```

### Откройте в браузере

На **любом устройстве** (телефон, ноутбук, рабочий ПК):

```
http://72.56.6.8:8000/docs
```

Должна открыться страница **Swagger UI** — интерактивная документация API. 🎉

---

## 🔧 13. Управление службой

```powershell
# Остановить FastAPI
C:\nssm\nssm-2.24\win64\nssm.exe stop FastAPI

# Перезапустить (после обновления кода)
C:\nssm\nssm-2.24\win64\nssm.exe restart FastAPI

# Удалить службу (если нужно)
C:\nssm\nssm-2.24\win64\nssm.exe remove FastAPI confirm

# Открыть GUI для редактирования (включая логи)
C:\nssm\nssm-2.24\win64\nssm.exe edit FastAPI
```

---

## 🔄 14. Обновление API

Когда вышли новые изменения в репозитории:

```powershell
# Перейдите в папку проекта
cd C:\tpgk-server

# Активируйте виртуальное окружение
.\venv\Scripts\activate

# Остановите службу
C:\nssm\nssm-2.24\win64\nssm.exe stop FastAPI

# Скачайте обновления
git pull

# Установите новые зависимости (если появились)
pip install -r requirements.txt

# Примените новые миграции (если появились)
alembic upgrade head

# Запустите обратно
C:\nssm\nssm-2.24\win64\nssm.exe start FastAPI
```

---

## 📋 Итого

| Компонент | Статус | Путь / URL |
|-----------|--------|------------|
| Python 3.11.9 | ✅ Установлен | Системный PATH |
| Git | ✅ Установлен | Системный PATH |
| Код API | ✅ `C:\tpgk-server` | Клонирован из GitHub |
| Виртуальное окружение | ✅ `C:\tpgk-server\venv` | Активируется через `.\venv\Scripts\activate` |
| .env | ✅ Настроен | Указывает на PostgreSQL, Redis, MinIO |
| База данных tpgk | ✅ Создана | Миграции применены |
| Служба FastAPI | ✅ Запущена через NSSM | Автозапуск при загрузке Windows |
| Порт 8000 | ✅ Открыт в брандмауэре | `http://72.56.6.8:8000` |
| Swagger Docs | ✅ Доступен | `http://72.56.6.8:8000/docs` |

---

## 🎯 Финальная проверка всей системы

| Сервис | URL | Как проверить |
|--------|-----|---------------|
| PostgreSQL | `72.56.6.8:5432` | Подключиться через pgAdmin/DBeaver |
| MinIO Console | `http://72.56.6.8:9001` | Открыть в браузере |
| Redis | `72.56.6.8:6379` | `redis-cli -h 72.56.6.8 ping` |
| **FastAPI Swagger** | `http://72.56.6.8:8000/docs` | Открыть в браузере |
| MAX-бот | В мессенджере MAX | Написать боту `/start` |

---

## 🚨 Если что-то не работает

### Служба не запускается

```powershell
# Посмотрите статус
Get-Service FastAPI

# Откройте GUI NSSM и посмотрите логи
C:\nssm\nssm-2.24\win64\nssm.exe edit FastAPI
```

Во вкладке **"I/O"** указаны файлы логов — откройте их блокнотом и посмотрите ошибку.

### Частые проблемы

| Проблема | Решение |
|----------|---------|
| `python не найден` | Перезапустите PowerShell после установки Python |
| `pip не найден` | Перезапустите PowerShell |
| `git не найден` | Перезапустите PowerShell после установки Git |
| `ModuleNotFoundError` | Активируйте venv: `.\venv\Scripts\activate` и запустите `pip install -r requirements.txt` |
| `connection refused` к PostgreSQL | Проверьте, что служба PostgreSQL запущена: `Get-Service postgresql-x64-16` |
| `connection refused` к Redis | Проверьте: `Get-Service Redis` |
| Ошибка миграции `relation already exists` | Проверьте, что БД `tpgk` создана и пуста, либо используйте `alembic stamp head` |
| Порт 8000 не открывается снаружи | Проверьте внешний брандмауэр (панель хостинга/VPS) |
| Служба стартует и сразу падает | Посмотрите логи NSSM — обычно проблема в пути к Python или venv |

### Полезные команды для диагностики

```powershell
# Все службы, связанные с проектом
Get-Service | Where-Object { $_.Name -match "FastAPI|PostgreSQL|Redis|MinIO|MaxBot" }

# Все открытые порты
netstat -ano | findstr "LISTENING"

# Проверить все порты проекта
@(5432, 6379, 8000, 9000, 9001) | ForEach-Object {
    Write-Host "Port $_:" -ForegroundColor Cyan
    netstat -ano | findstr ":$_ "
}
```

---
