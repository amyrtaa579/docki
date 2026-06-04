
## 🚀 Установка и запуск FastAPI (Anmicius API)

FastAPI backend для Томского промышленно-гуманитарного колледжа (ТПГК).

### 1. **Клонирование репозитория**

```powershell
# Перейдите в папку проектов
cd C:\

# Клонируйте репозиторий
git clone https://github.com/amyrtaa579/tpgk-server.git

# Перейдите в папку проекта
cd C:\tpgk-server
```

### 2. **Создание виртуального окружения**

```powershell
# Создайте виртуальное окружение
python -m venv venv

# Активируйте его (Windows)
.\venv\Scripts\activate

# После активации в начале строки появится (venv)
```

### 3. **Установка зависимостей**

```powershell
# Обновите pip
python -m pip install --upgrade pip

# Установите все зависимости из requirements.txt
pip install -r requirements.txt
```

### 4. **Настройка .env**

```powershell
# Скопируйте пример файла окружения
Copy-Item .env.example .env

# Откройте .env в блокноте и отредактируйте под свои сервисы
notepad .env
```

Пример содержимого `.env`:
```env
DATABASE_URL=postgresql+asyncpg://postgres:postgres123@72.56.6.8:5432/tpgk
REDIS_URL=redis://72.56.6.8:6379/0
MINIO_ENDPOINT=72.56.6.8:9000
MINIO_ACCESS_KEY=admin
MINIO_SECRET_KEY=password123
SECRET_KEY=your-super-secret-key
ALGORITHM=HS256
```

### 5. **Применение миграций**

```powershell
# Создайте базу данных tpgk в PostgreSQL (если ещё не создана)
& "C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres -c "CREATE DATABASE tpgk;"

# Примените все миграции Alembic
alembic upgrade head
```

### 6. **Создание администратора**

```powershell
# Запустите скрипт инициализации админа
python init-admin.py

# Следуйте инструкциям в консоли для создания учётной записи
```

### 7. **Запуск сервера**

```powershell
# Запустите FastAPI через uvicorn
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 8. **Установка как службы Windows (через NSSM)**

```powershell
# Создайте скрипт запуска FastAPI
@'
@echo off
cd /d C:\tpgk-server
call venv\Scripts\activate
uvicorn app.main:app --host 0.0.0.0 --port 8000
'@ | Out-File -FilePath "C:\start-fastapi.bat" -Encoding ASCII

# Установите FastAPI как службу
C:\nssm\nssm-2.24\win64\nssm.exe install FastAPI "C:\start-fastapi.bat"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI AppDirectory "C:\tpgk-server"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI DisplayName "FastAPI Server"
C:\nssm\nssm-2.24\win64\nssm.exe set FastAPI Description "FastAPI Backend for TPGK"
C:\nssm\nssm-2.24\win64\nssm.exe start FastAPI
```

### 9. **Откройте порт в брандмауэре**

```powershell
New-NetFirewallRule -DisplayName "FastAPI" -Direction Inbound -LocalPort 8000 -Protocol TCP -Action Allow
```

### 10. **Проверка работы**

```powershell
# Проверьте статус службы
Get-Service FastAPI

# Проверьте, слушает ли порт 8000
netstat -ano | findstr "8000"

# Проверьте доступность API
Invoke-RestMethod -Uri "http://localhost:8000/docs"

# С удалённой машины
Test-NetConnection -ComputerName 72.56.6.8 -Port 8000
```

---

## 📋 Итого

| Сервис | Порт | URL |
|--------|------|-----|
| PostgreSQL | 5432 | `72.56.6.8:5432` |
| MinIO API | 9000 | `http://72.56.6.8:9000` |
| MinIO Console | 9001 | `http://72.56.6.8:9001` |
| Redis | 6379 | `72.56.6.8:6379` |
| FastAPI | 8000 | `http://72.56.6.8:8000` |
| Swagger Docs | 8000 | `http://72.56.6.8:8000/docs` |
