
## 🐘 Установка PostgreSQL

### **Способ 1: Через официальный установщик (рекомендуется)**

```powershell
# Скачайте установщик PostgreSQL 16
Invoke-WebRequest -Uri "https://get.enterprisedb.com/postgresql/postgresql-16.1-1-windows-x64.exe" -OutFile "C:\postgresql-installer.exe"

# установите вручную, запустив:
C:\postgresql-installer.exe
```

### **Способ 2: Zip-архив (более легкий)**

```powershell
# Скачайте PostgreSQL zip
Invoke-WebRequest -Uri "https://get.enterprisedb.com/postgresql/postgresql-16.1-1-windows-x64-binaries.zip" -OutFile "C:\postgresql.zip"

# Распакуйте
Expand-Archive -Path "C:\postgresql.zip" -DestinationPath "C:\postgresql"

# Создайте папку для данных
New-Item -ItemType Directory -Path "C:\postgresql\data" -Force

# Инициализируйте базу данных
C:\postgresql\bin\initdb.exe -D C:\postgresql\data -U postgres -pw postgres123

# Создайте службу PostgreSQL
C:\postgresql\bin\pg_ctl.exe register -D C:\postgresql\data -N PostgreSQL16

# Запустите службу
Start-Service PostgreSQL16
```

### **Откройте порт в брандмауэре**

```powershell
New-NetFirewallRule -DisplayName "PostgreSQL" -Direction Inbound -LocalPort 5432 -Protocol TCP -Action Allow
```

Проверим, что правило работает
```powershell
# Посмотреть все правила, связанные с PostgreSQL
Get-NetFirewallRule -DisplayName "PostgreSQL"

# Проверить, слушает ли PostgreSQL порт 5432
netstat -ano | findstr "5432"
```

### **Проверьте установку**

```powershell
# Проверить располложение psql.exe
Get-ChildItem -Path C:\ -Recurse -Filter "psql.exe" -ErrorAction SilentlyContinue | Select-Object FullName

# Проверьте статус службы
Get-Service | Where-Object { $_.Name -like '*post*'}

# Подключитесь к PostgreSQL
& "C:\Program Files\PostgreSQL\16\bin\psql.exe" -U postgres -h localhost
```

### **Настройте удаленный доступ (если нужно)**

```powershell
# 1. Найдём правильный путь к данным
# Путь к конфигурационным файлам
$pgConf = "C:\Program Files\PostgreSQL\16\data\postgresql.conf"
$pgHba = "C:\Program Files\PostgreSQL\16\data\pg_hba.conf"

# Проверим, существуют ли они
Test-Path $pgConf
Test-Path $pgHba

# 2. Если файлы существуют выполним
# 2.1. Разрешить слушать все интерфейсы в postgresql.conf
(Get-Content "C:\Program Files\PostgreSQL\16\data\postgresql.conf") -replace "#listen_addresses = 'localhost'", "listen_addresses = '*'" | Set-Content "C:\Program Files\PostgreSQL\16\data\postgresql.conf"

# 2.2. Добавить правило в pg_hba.conf
@"
# Allow remote connections
host    all             all             0.0.0.0/0               md5
"@ | Add-Content "C:\Program Files\PostgreSQL\16\data\pg_hba.conf"

# 2.3. Перезапустить службу
Restart-Service postgresql-x64-16
```

---

**После установки PostgreSQL будет доступен:**
- **Host**: `72.56.6.8` (или localhost)
- **Port**: `5432`
- **User**: `postgres`
- **Password**: `postgres123`

