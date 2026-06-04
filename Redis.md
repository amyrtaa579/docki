## 🚀 Установка Redis на Windows

### 1. Скачиваем Redis

```powershell
# Скачайте Redis 5.0.14.1 (последняя стабильная версия для Windows)
Invoke-WebRequest -Uri "https://github.com/tporadowski/redis/releases/download/v5.0.14.1/Redis-x64-5.0.14.1.zip" -OutFile "C:\redis.zip"

# Распакуйте
Expand-Archive -Path "C:\redis.zip" -DestinationPath "C:\redis" -Force
```

### 2. Настройте для удалённых подключений

```powershell
# Отредактируйте конфиг
$redisConf = "C:\redis\redis.windows.conf"

# 1. Разрешить подключения с любого IP
(Get-Content $redisConf) -replace "bind 127.0.0.1", "bind 0.0.0.0" | Set-Content $redisConf

# 2. (Опционально) Установить пароль
# Найдите строку "# requirepass foobared" и замените на:
# requirepass redis123
```

### 3. Установите как службу Windows (через NSSM)

```powershell
# Если NSSM ещё нет
Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"

# Установите Redis как службу
C:\nssm\nssm-2.24\win64\nssm.exe install Redis "C:\redis\redis-server.exe" "C:\redis\redis.windows.conf"
C:\nssm\nssm-2.24\win64\nssm.exe set Redis AppDirectory "C:\redis"
C:\nssm\nssm-2.24\win64\nssm.exe set Redis DisplayName "Redis Server"
C:\nssm\nssm-2.24\win64\nssm.exe set Redis Description "Redis Server for Windows"
C:\nssm\nssm-2.24\win64\nssm.exe start Redis
```

### 4. Откройте порт в брандмауэре

```powershell
New-NetFirewallRule -DisplayName "Redis" -Direction Inbound -LocalPort 6379 -Protocol TCP -Action Allow
```

### 5. Проверьте работу

```powershell
# Проверьте статус службы
Get-Service Redis

# Проверьте, слушает ли порт
netstat -ano | findstr "6379"

# Локальный тест
C:\redis\redis-cli.exe ping
# Должно вернуть: PONG

# С удалённой машины
Test-NetConnection -ComputerName 72.56.6.8 -Port 6379
```

---

## 📋 Итого

| Сервис | Порт | URL |
|--------|------|-----|
| PostgreSQL | 5432 | `72.56.6.8:5432` |
| MinIO API | 9000 | `http://72.56.6.8:9000` |
| MinIO Console | 9001 | `http://72.56.6.8:9001` |
| Redis | 6379 | `72.56.6.8:6379` |

**После установки Redis можно переходить к FastAPI!** 🚀
