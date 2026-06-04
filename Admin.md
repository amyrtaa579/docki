Отлично! На основе README и вашего стека напишу подробную документацию по развёртыванию Admin Panel на Windows Server, в том же стиле, что и предыдущие инструкции.

---

# 🎨 Установка Anmicius Admin Panel на Windows Server

## 📋 Принцип работы

Admin Panel — это **React SPA** (Single Page Application). После сборки она превращается в набор статических файлов (HTML, JS, CSS), которые нужно раздавать через веб-сервер и проксировать API-запросы к FastAPI бэкенду.

Мы настроим это через **Nginx для Windows** — он будет:
1. Раздавать статические файлы админ-панели (порт **3000**)
2. Проксировать запросы `/api/*`, `/auth/*`, `/admin/*` на FastAPI (порт **8000**)

```
Браузер → :3000 (Nginx) → статика (React SPA)
                           → /api/* → :8000 (FastAPI)
                           → /auth/* → :8000 (FastAPI)
                           → /admin/* → :8000 (FastAPI)
```

---

## 📦 1. Установка Node.js

Node.js нужен для сборки React-приложения.

```powershell
# Скачайте Node.js 20 LTS (рекомендуемая версия)
Invoke-WebRequest -Uri "https://nodejs.org/dist/v20.11.0/node-v20.11.0-x64.msi" -OutFile "C:\nodejs-installer.msi"

# Установите тихо (silent install)
Start-Process msiexec.exe -ArgumentList "/i C:\nodejs-installer.msi /quiet /norestart" -Wait

# Перезапустите PowerShell и проверьте
node --version
# Должно вывести: v20.11.0

npm --version
# Должно вывести: 10.x.x
```

---

## 📥 2. Клонирование репозитория

```powershell
# Перейдите в корень диска
cd C:\

# Клонируйте репозиторий админ-панели
git clone https://github.com/amyrtaa579/tpgk-admin.git

# Перейдите в папку проекта
cd C:\tpgk-admin
```

---

## ⚙️ 3. Настройка API-прокси для продакшена

В продакшене прокси Vite не работает (он только для разработки). Поэтому настроим прокси через переменную окружения.

### 3.1. Проверьте текущий api.ts

```powershell
# Откройте файл для просмотра
notepad C:\tpgk-admin\src\services\api.ts
```

Убедитесь, что `API_BASE_URL` выглядит так:

```typescript
const API_BASE_URL = '/api/v1';
```

> 💡 **Важно:** именно `/api/v1` (относительный путь), а не `http://localhost:8000/api/v1`.  
> Так запросы будут идти на тот же домен/порт, где крутится админ-панель, а Nginx проксирует их на FastAPI.

---

## 📦 4. Установка зависимостей и сборка

```powershell
# Убедитесь, что вы в папке проекта
cd C:\tpgk-admin

# Установите все зависимости
npm install

# Запустите сборку для продакшена
npm run build
```

После успешной сборки появится папка `dist/`:

```powershell
# Проверьте, что папка dist создалась
Get-ChildItem C:\tpgk-admin\dist
```

Должны быть файлы: `index.html`, папка `assets/` с JS и CSS.

---

## 🌐 5. Установка и настройка Nginx для Windows

### 5.1. Скачайте Nginx

```powershell
# Скачайте Nginx для Windows
Invoke-WebRequest -Uri "https://nginx.org/download/nginx-1.24.0.zip" -OutFile "C:\nginx.zip"

# Распакуйте
Expand-Archive -Path "C:\nginx.zip" -DestinationPath "C:\" -Force
# После распаковки папка будет: C:\nginx-1.24.0

# Переименуйте для удобства
Rename-Item -Path "C:\nginx-1.24.0" -NewName "nginx"
```

### 5.2. Настройте конфиг Nginx

```powershell
# Откройте конфиг для редактирования
notepad C:\nginx\nginx-1.26.3\conf\nginx.conf
```

**Полностью замените содержимое** на это:

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       3000;
        server_name  _;
        
        # Максимальный размер загружаемого файла
        client_max_body_size 100M;

        # Статические файлы админ-панели
        root   C:/tpgk-admin/dist;
        index  index.html;

        # SPA routing — все не-файловые запросы направляем на index.html
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Проксирование API-запросов на FastAPI
        location /api/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /auth/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /admin/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

> 💡 **Обратите внимание:** пути в `root` используют **прямые слэши** `/` (даже на Windows), а буква диска — заглавная `C:`.

### 5.3. Проверьте конфиг

```powershell
# Проверьте синтаксис конфигурации
cd C:\nginx\nginx-1.26.3
.\nginx.exe -t
```

Должно вывести:
```
nginx: the configuration file C:\nginx/conf/nginx.conf syntax is ok
nginx: configuration file C:\nginx/conf/nginx.conf test is successful
```

---

## 🔥 6. Откройте порт в брандмауэре

```powershell
# Откройте порт 3000 для админ-панели
New-NetFirewallRule -DisplayName "Admin Panel (Nginx)" -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow
```

Проверьте:

```powershell
Get-NetFirewallRule -DisplayName "Admin Panel*"
```

---

## 🛠 7. Установка Nginx как службы Windows (через NSSM)

```powershell
# Если NSSM ещё нет — скачайте
if (-not (Test-Path "C:\nssm")) {
    Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
    Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"
}

# Установите Nginx как службу
C:\nssm\nssm-2.24\win64\nssm.exe install Nginx "C:\nginx\nginx.exe"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx AppDirectory "C:\nginx"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx DisplayName "Nginx Web Server"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx Description "Nginx for Admin Panel"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx Start SERVICE_AUTO_START

# Запустите службу
C:\nssm\nssm-2.24\win64\nssm.exe start Nginx
```

---

## ▶️ 8. Проверка работы

### 8.1. Локальные проверки

```powershell
# Проверьте статус службы Nginx
Get-Service Nginx
# Должно быть: Status: Running

# Проверьте, что порт 3000 слушается
netstat -ano | findstr ":3000"
# Должна быть строка: TCP  0.0.0.0:3000  ...  LISTENING

# Проверьте локально через curl (если есть) или PowerShell
Invoke-WebRequest -Uri "http://localhost:3000" -UseBasicParsing
# Должен вернуть HTML-код админ-панели (статус 200)
```

### 8.2. Проверка с удалённой машины

```powershell
Test-NetConnection -ComputerName 72.56.6.8 -Port 3000
# Должно быть: TcpTestSucceeded : True
```

### 8.3. Откройте в браузере

На **любом устройстве** в сети:

```
http://72.56.6.8:3000
```

Должна открыться страница входа в админ-панель. 🎉

---

## 🔄 9. Обновление админ-панели

Когда вышли новые изменения в репозитории:

```powershell
# Перейдите в папку проекта
cd C:\tpgk-admin

# Остановите Nginx (пока пересобираем)
C:\nssm\nssm-2.24\win64\nssm.exe stop Nginx

# Скачайте обновления
git pull

# Установите новые зависимости (если появились)
npm install

# Пересоберите проект
npm run build

# Запустите Nginx обратно
C:\nssm\nssm-2.24\win64\nssm.exe start Nginx
```

> 💡 Nginx останавливаем на время сборки, чтобы старые файлы не кэшировались. Сама сборка занимает ~30-60 секунд.

---

## 🔧 10. Управление службой Nginx

```powershell
# Остановить
C:\nssm\nssm-2.24\win64\nssm.exe stop Nginx

# Перезапустить (например, после изменения конфига)
C:\nssm\nssm-2.24\win64\nssm.exe restart Nginx

# Удалить службу (если нужно)
C:\nssm\nssm-2.24\win64\nssm.exe remove Nginx confirm

# Открыть GUI для просмотра логов
C:\nssm\nssm-2.24\win64\nssm.exe edit Nginx
```

Во вкладке **I/O** можно посмотреть пути к логам Nginx (по умолчанию `C:\nginx\logs\`).

---

## 📋 11. Итоговая таблица всех сервисов

| Сервис | Порт | URL | Служба Windows |
|--------|------|-----|----------------|
| PostgreSQL | 5432 | `72.56.6.8:5432` | `PostgreSQL16` |
| Redis | 6379 | `72.56.6.8:6379` | `Redis` |
| MinIO API | 9000 | `http://72.56.6.8:9000` | `MinIO` |
| MinIO Console | 9001 | `http://72.56.6.8:9001` | `MinIO` |
| FastAPI | 8000 | `http://72.56.6.8:8000` | `FastAPI` |
| **Admin Panel** | **3000** | **`http://72.56.6.8:3000`** | **`Nginx`** |

---

## 🧪 12. Финальная проверка всей системы

```powershell
# Все службы проекта
Get-Service | Where-Object { $_.Name -match "FastAPI|PostgreSQL|Redis|MinIO|Nginx" }

# Все порты проекта
@(3000, 5432, 6379, 8000, 9000, 9001) | ForEach-Object {
    $result = netstat -ano | Select-String ":$_ "
    if ($result) {
        Write-Host "✅ Порт $_ слушается" -ForegroundColor Green
    } else {
        Write-Host "❌ Порт $_ НЕ слушается" -ForegroundColor Red
    }
}
```

---

## 🚨 Если что-то не работает

### Nginx не запускается

```powershell
# Проверьте логи
Get-Content C:\nginx\nginx-1.26.3\logs\error.log -Tail 30

# Проверьте конфиг
C:\nginx\nginx-1.26.3\nginx.exe -t

# Частая проблема: порт 3000 уже занят
netstat -ano | findstr ":3000"
```

### Белый экран / 404 при обновлении страницы

Убедитесь, что в конфиге Nginx есть `try_files $uri $uri/ /index.html;` — это нужно для SPA-роутинга.

### API-запросы возвращают 404 или CORS-ошибку

Проверьте, что FastAPI запущен на порту 8000:

```powershell
netstat -ano | findstr ":8000"
```

И что в `api.ts` стоит относительный путь `/api/v1`, а не абсолютный URL.

### Не загружаются изображения из MinIO

Убедитесь, что в `.env` бэкенда правильно указаны `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`.

### Порт 3000 не открыт снаружи

```powershell
# Проверьте правило брандмауэра
Get-NetFirewallRule -DisplayName "Admin Panel*" | Format-List

# Проверьте, не блокирует ли внешний брандмауэр хостинга
```

---

## Контакты для связи

- **FastAPI (Anmicius API)**: `http://72.56.6.8:8000/docs` — Swagger документация
- **Admin Panel**: `http://72.56.6.8:3000` — веб-интерфейс управления
- **MinIO Console**: `http://72.56.6.8:9001` — управление файлами

---

Теперь админ-панель должна быть доступна по `http://72.56.6.8:3000` и полностью готова к работе! Если возникнут вопросы при настройке — пишите.