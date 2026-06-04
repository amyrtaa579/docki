# 🚀 Полная установка Admin Panel с нуля

Административная панель для управления контентом Томского промышленно-гуманитарного колледжа (ТПГК).

> ⚠️ **Важно:** В этой инструкции мы установим **всё с нуля** — от Node.js до запуска службы с Nginx reverse proxy.

---

## 📦 1. Установка Node.js 20 LTS

> 💡 Если Node.js уже установлен (для MAX-бота) — пропустите этот шаг.

```powershell
# Скачайте Node.js 20 LTS (официальный установщик)
Invoke-WebRequest -Uri "https://nodejs.org/dist/v20.18.1/node-v20.18.1-x64.msi" -OutFile "C:\nodejs-installer.msi"

# Запустите установщик
Start-Process msiexec.exe -ArgumentList "/i C:\nodejs-installer.msi /qn" -Wait
```

**Обязательно перезапустите PowerShell** после установки!

### Проверьте установку

```powershell
node -v
# Должно вывести: v20.18.1

npm -v
# Должно вывести: 10.x.x
```

---

## 📦 2. Установка Git

> 💡 Если Git уже установлен (для FastAPI) — пропустите этот шаг.

```powershell
# Скачайте установщик Git
Invoke-WebRequest -Uri "https://github.com/git-for-windows/git/releases/download/v2.47.1.windows.1/Git-2.47.1-64-bit.exe" -OutFile "C:\git-installer.exe"

# Запустите установщик
Start-Process "C:\git-installer.exe"
```

В установщике можно просто нажимать **"Next"** везде.

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
git clone https://github.com/amyrtaa579/tpgk-admin.git

# Перейдите в папку проекта
cd C:\tpgk-admin
```

---

## 📚 4. Установка зависимостей

### Вариант 1: Первый запуск (нет `package-lock.json`)

```powershell
npm install
```

После этой команды появится файл `package-lock.json`.

### Вариант 2: Повторный запуск (есть `package-lock.json`)

```powershell
npm ci
```

### 📝 Разница между командами

| Команда | Когда использовать |
|---------|-------------------|
| `npm install` | Первый запуск, когда нет `package-lock.json` |
| `npm ci` | CI/CD, продакшен (когда есть `package-lock.json`) |

> ⏳ Установка может занять 2-5 минут.

---

## 🔨 5. Сборка проекта для продакшена

```powershell
# Соберите проект
npm run build
```

После этого появится папка `dist` с готовыми статическими файлами.

### ⚠️ Если сборка падает с ошибкой `@react-aria/ssr`

Это известная проблема с зависимостями. Решение — обновить `vite.config.ts`:

```powershell
# Откройте конфиг
notepad vite.config.ts
```

Замените **всё содержимое** на:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    host: '0.0.0.0',
    port: 3000,
    proxy: {
      '/api': 'http://localhost:8000',
      '/auth': 'http://localhost:8000',
      '/admin': 'http://localhost:8000',
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom', 'react-router-dom'],
          ui: ['bootstrap', 'react-bootstrap']
        }
      }
    }
  },
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom']
  }
})
```

Сохраните и повторите сборку:

```powershell
# Очистите кэш Vite
Remove-Item -Recurse -Force node_modules\.vite -ErrorAction SilentlyContinue

# Соберите заново
npm run build
```

---

## 🌐 6. Установка Nginx для Windows

Nginx будет работать как:
- Веб-сервер для раздачи статических файлов (React приложение)
- Reverse proxy для проксирования API запросов к FastAPI

### 6.1. Скачайте Nginx

```powershell
# Скачайте Nginx для Windows
Invoke-WebRequest -Uri "http://nginx.org/download/nginx-1.26.3.zip" -OutFile "C:\nginx.zip"

# Распакуйте
Expand-Archive -Path "C:\nginx.zip" -DestinationPath "C:\nginx" -Force
```

### 6.2. Настройте Nginx

```powershell
# Откройте конфигурационный файл
notepad "C:\nginx\nginx-1.26.3\conf\nginx.conf"
```

**Удалите всё содержимое** и вставьте следующее:

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
        listen       80;
        server_name  localhost;

        # Корневая папка с собранным React приложением
        root   "C:/tpgk-admin/dist";
        index  index.html;

        # SPA routing - все запросы перенаправляем на index.html
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Proxy API запросов к FastAPI
        location /api/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Proxy auth запросов к FastAPI
        location /auth/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Proxy admin API запросов к FastAPI
        location /admin/ {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Кэширование статических файлов
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

Сохраните файл.

### 6.3. Проверьте конфигурацию

```powershell
# Перейдите в папку Nginx
cd "C:\nginx\nginx-1.26.3"

# Проверьте конфигурацию
.\nginx.exe -t
```

Должно вывести:
```
nginx: the configuration file C:\nginx\nginx-1.26.3/conf/nginx.conf syntax is ok
nginx: configuration file C:\nginx\nginx-1.26.3/conf/nginx.conf test is successful
```

---

## 🛠 7. Установка Nginx как службы Windows

```powershell
# Если NSSM ещё нет
if (-not (Test-Path "C:\nssm")) {
    Invoke-WebRequest -Uri "https://nssm.cc/release/nssm-2.24.zip" -OutFile "C:\nssm.zip"
    Expand-Archive -Path "C:\nssm.zip" -DestinationPath "C:\nssm"
}

# Установите Nginx как службу
C:\nssm\nssm-2.24\win64\nssm.exe install Nginx "C:\nginx\nginx-1.26.3\nginx.exe"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx AppDirectory "C:\nginx\nginx-1.26.3"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx DisplayName "Nginx Web Server"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx Description "Nginx for TPGK Admin Panel"
C:\nssm\nssm-2.24\win64\nssm.exe set Nginx Start SERVICE_AUTO_START

# Запустите службу
C:\nssm\nssm-2.24\win64\nssm.exe start Nginx
```

---

## 🔥 8. Открытие порта в брандмауэре

```powershell
# Откройте порт 80 (HTTP) для доступа к админ-панели
New-NetFirewallRule -DisplayName "Nginx HTTP" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow
```

### Проверьте правило

```powershell
Get-NetFirewallRule -DisplayName "Nginx HTTP"
```

---

## ✅ 9. Проверка работы

```powershell
# Проверьте статус службы Nginx
Get-Service Nginx
# Должно быть: Status: Running

# Проверьте, что порт 80 слушается
netstat -ano | findstr ":80 "
# Должна быть строка: TCP  0.0.0.0:80  ...  LISTENING

# Проверьте локально
Invoke-WebRequest -Uri "http://localhost"

# Проверьте с удалённой машины
Test-NetConnection -ComputerName 72.56.6.8 -Port 80
# Должно быть: TcpTestSucceeded : True
```

### Откройте в браузере

На **любом устройстве**:

```
http://72.56.6.8
```

Должна открыться **страница входа** в админ-панель. 🎉

Войдите с учётными данными администратора (которого создали при настройке FastAPI).

---

## 🔧 10. Управление службами

```powershell
# Остановить Nginx
C:\nssm\nssm-2.24\win64\nssm.exe stop Nginx

# Перезапустить Nginx
C:\nssm\nssm-2.24\win64\nssm.exe restart Nginx

# Перезагрузить конфигурацию Nginx (без остановки)
cd "C:\nginx\nginx-1.26.3"
.\nginx.exe -s reload

# Удалить службу
C:\nssm\nssm-2.24\win64\nssm.exe remove Nginx confirm

# Открыть GUI для редактирования
C:\nssm\nssm-2.24\win64\nssm.exe edit Nginx
```

---

## 🔄 11. Обновление админ-панели

Когда вышли новые изменения в репозитории:

```powershell
# Перейдите в папку проекта
cd C:\tpgk-admin

# Скачайте обновления
git pull

# Установите новые зависимости (если появились)
npm ci

# Пересоберите проект
npm run build

# Перезагрузите Nginx (не обязательно, но желательно)
cd "C:\nginx\nginx-1.26.3"
.\nginx.exe -s reload
```

> 💡 Nginx не нужно останавливать — он просто раздаёт файлы из папки `dist/`, которая обновляется при сборке.

---

## 📋 Итого

| Компонент | Статус | Путь / URL |
|-----------|--------|------------|
| Node.js 20 LTS | ✅ Установлен | Системный PATH |
| Git | ✅ Установлен | Системный PATH |
| Код Admin Panel | ✅ `C:\tpgk-admin` | Клонирован из GitHub |
| Сборка | ✅ `C:\tpgk-admin\dist` | Готовые статические файлы |
| Nginx | ✅ `C:\nginx\nginx-1.26.3` | Reverse proxy + web server |
| Служба Nginx | ✅ Запущена через NSSM | Автозапуск при загрузке Windows |
| Порт 80 | ✅ Открыт в брандмауэре | `http://72.56.6.8` |
| Admin Panel | ✅ Доступна | `http://72.56.6.8` |

---

## 🎯 Финальная проверка всей системы

| Сервис | URL | Как проверить |
|--------|-----|---------------|
| PostgreSQL | `72.56.6.8:5432` | Подключиться через pgAdmin/DBeaver |
| MinIO Console | `http://72.56.6.8:9001` | Открыть в браузере |
| Redis | `72.56.6.8:6379` | `redis-cli -h 72.56.6.8 ping` |
| FastAPI Swagger | `http://72.56.6.8:8000/docs` | Открыть в браузере |
| **Admin Panel** | `http://72.56.6.8` | Открыть в браузере |
| MAX-бот | В мессенджере MAX | Написать боту `/start` |

---

## 🚨 Если что-то не работает

### Nginx не запускается

```powershell
# Посмотрите статус
Get-Service Nginx

# Откройте GUI NSSM и посмотрите логи
C:\nssm\nssm-2.24\win64\nssm.exe edit Nginx
```

Во вкладке **"I/O"** указаны файлы логов — откройте их блокнотом.

### Страница не открывается

```powershell
# Проверьте, что Nginx запущен
Get-Service Nginx

# Проверьте, что порт 80 слушается
netstat -ano | findstr ":80 "

# Проверьте конфигурацию Nginx
cd "C:\nginx\nginx-1.26.3"
.\nginx.exe -t

# Посмотрите логи Nginx
Get-Content "C:\nginx\nginx-1.26.3\logs\error.log" -Tail 50
```

### API запросы не работают (404 или 502)

```powershell
# Проверьте, что FastAPI запущен
Get-Service FastAPI

# Проверьте, что FastAPI слушает порт 8000
netstat -ano | findstr ":8000 "

# Проверьте логи Nginx
Get-Content "C:\nginx\nginx-1.26.3\logs\error.log" -Tail 50
```

### Сборка падает с ошибкой `@react-aria/ssr`

Это проблема с зависимостями. Решение:

1. Обновите `vite.config.ts` (см. раздел 5)
2. Очистите кэш: `Remove-Item -Recurse -Force node_modules\.vite`
3. Пересоберите: `npm run build`

### `npm ci` выдаёт ошибку `npm ci can only install packages when your package.json and package-lock.json`

Значит в репозитории нет `package-lock.json`. Используйте:

```powershell
npm install
```

### Частые проблемы

| Проблема | Решение |
|----------|---------|
| `node не найден` | Перезапустите PowerShell после установки Node.js |
| `npm не найден` | Перезапустите PowerShell |
| `git не найден` | Перезапустите PowerShell после установки Git |
| Порт 80 уже занят (IIS) | Остановите IIS: `Stop-Service W3SVC` или измените порт в `nginx.conf` |
| 502 Bad Gateway при обращении к API | Проверьте, что FastAPI запущен: `Get-Service FastAPI` |
| Белый экран после входа | Проверьте консоль браузера (F12) — возможно, проблема с API |
| CORS ошибки | Проверьте настройки CORS в FastAPI (`app/main.py`) |
| Изображения из MinIO не загружаются | Проверьте, что MinIO доступен и URL правильный в БД |
| Ошибка сборки `@react-aria/ssr` | Обновите `vite.config.ts` (см. раздел 5) |
| `npm ci` не работает | Используйте `npm install` вместо `npm ci` |

### Полезные команды для диагностики

```powershell
# Все службы проекта
Get-Service | Where-Object { $_.Name -match "FastAPI|PostgreSQL|Redis|MinIO|MaxBot|Nginx" }

# Все открытые порты
netstat -ano | findstr "LISTENING"

# Проверить все порты проекта
@(80, 5432, 6379, 8000, 9000, 9001) | ForEach-Object {
    Write-Host "Port $_:" -ForegroundColor Cyan
    netstat -ano | findstr ":$_ "
}

# Логи Nginx (ошибки)
Get-Content "C:\nginx\nginx-1.26.3\logs\error.log" -Tail 100

# Логи Nginx (доступ)
Get-Content "C:\nginx\nginx-1.26.3\logs\access.log" -Tail 100
```

---

## 📝 Дополнительная информация

### Изменение порта Nginx

Если нужно запустить админ-панель на другом порту (например, 8080):

1. Откройте `C:\nginx\nginx-1.26.3\conf\nginx.conf`
2. Найдите строку: `listen 80;`
3. Замените на: `listen 8080;`
4. Перезагрузите Nginx: `.\nginx.exe -s reload`
5. Откройте порт в брандмауэре: `New-NetFirewallRule -DisplayName "Nginx 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow`

### HTTPS (SSL)

Для настройки HTTPS:

1. Получите SSL сертификат (Let's Encrypt или купите)
2. Добавьте в `nginx.conf`:

```nginx
server {
    listen 443 ssl;
    server_name admin.example.com;

    ssl_certificate "C:/path/to/cert.pem";
    ssl_certificate_key "C:/path/to/key.pem";

    # ... остальная конфигурация
}

# Редирект с HTTP на HTTPS
server {
    listen 80;
    server_name admin.example.com;
    return 301 https://$server_name$request_uri;
}
```

3. Откройте порт 443 в брандмауэре

### Разработка (dev-режим)

Для разработки можно запустить Vite dev-сервер:

```powershell
cd C:\tpgk-admin
npm run dev
```

Админ-панель будет доступна на `http://localhost:3000` с горячей перезагрузкой.
````

---

## 📋 Что изменилось по сравнению с предыдущей версией

| Раздел | Изменение |
|--------|-----------|
| **Раздел 4** | Разделён на два варианта: `npm install` (первый запуск) и `npm ci` (повторный) |
| **Раздел 5** | Добавлен подраздел "Если сборка падает с ошибкой `@react-aria/ssr`" с решением через `vite.config.ts` |
| **Раздел 11** | Упрощён — Nginx не нужно останавливать для обновления |
| **Частые проблемы** | Добавлены 2 новые записи: про `@react-aria/ssr` и про `npm ci` |

Теперь документация полностью отражает реальный опыт установки и все подводные камни, с которыми ты столкнулся. 🎯