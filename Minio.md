## 🚀 Установка MinIO на Windows
### 1. **Исправленная установка IIS (если нужен)**

```powershell
# Правильный синтаксис:
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```


### 2. **Установка MinIO (без Docker)**

```powershell
# Скачайте MinIO сервер
Invoke-WebRequest -Uri "https://dl.min.io/server/minio/release/windows-amd64/minio.exe" -OutFile "C:\minio.exe"

# Создайте папку для данных
New-Item -ItemType Directory -Path "C:\minio-data" -Force

# Создайте скрипт запуска MinIO
@'
@echo off
set MINIO_ROOT_USER=admin
set MINIO_ROOT_PASSWORD=password123
C:\minio.exe server C:\minio-data --address ":9000" --console-address ":9001"
'@ | Out-File -FilePath "C:\start-minio.bat" -Encoding ASCII
```

### 3. **Установка как службы Windows (через NSSM)**

```powershell
# Скачайте NSSM
Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"

# Установите MinIO как службу
C:\nssm\nssm-2.24\win64\nssm.exe install MinIO "C:\start-minio.bat"
C:\nssm\nssm-2.24\win64\nssm.exe set MinIO AppDirectory "C:\"
C:\nssm\nssm-2.24\win64\nssm.exe start MinIO
```
### 4. Настройка брандмауэра

```powershell
# Открой порт 9000 (MinIO API — для S3-клиентов и FastAPI)
New-NetFirewallRule -DisplayName "MinIO API" -Direction Inbound -LocalPort 9000 -Protocol TCP -Action Allow

# Открой порт 9001 (MinIO Console — веб-интерфейс)
New-NetFirewallRule -DisplayName "MinIO Console" -Direction Inbound -LocalPort 9001 -Protocol TCP -Action Allow
```
### Проверь, что правила созданы:

```powershell
Get-NetFirewallRule -DisplayName "MinIO*"
```

Должны появиться две записи со статусом `Enabled`.

### 5. **Проверка доступа**

После настройки:

- **MinIO**: `http://72.56.6.8:9000`
- **MinIO Console**: `http://72.56.6.8:9001`
