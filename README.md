# NurCRM Manablock

Локальное настольное приложение для Windows-моноблока:

- `desktop/` - Electron + React интерфейс с обязательным логином.
- `backend/` - FastAPI-сервер, работающий локально на `127.0.0.1:8000`.
- `rust-core/` - Rust/PyO3 модуль для ускорения тяжёлых заказов.
- `installer/` - шаблон Inno Setup для итогового одного установщика.

## Архитектура

1. Пользователь проходит регистрацию или логин.
2. UI работает внутри Electron.
3. Electron поднимает локальный Python backend.
4. Простые заказы обрабатываются в Python.
5. Весовые заказы вызывают `heavy_engine.process_heavy_order()` из Rust.
6. Пользователи и основные заказы хранятся в PostgreSQL.
7. SQLite используется только как локальный офлайн-кэш.

## Быстрый старт

### 1. Установить зависимости

```powershell
cd .\backend
python -m pip install -U pip
python -m pip install .[build]
```

```powershell
cd .\desktop
npm install
```

### 2. Подготовить PostgreSQL

Создайте локальную БД `nurcrm` и при необходимости переопределите:

```powershell
$env:NURCRM_POSTGRES_DSN="postgresql+psycopg://postgres:postgres@127.0.0.1:5432/nurcrm"
```

### 3. Установить Rust для heavy-модуля

Нужен `rustup`/`cargo`. После установки:

```powershell
cd .\rust-core
python -m maturin develop --release
```

Если Rust ещё не установлен, backend продолжит работать на Python fallback для тяжёлых заказов.

### 4. Запуск в dev режиме

Backend:

```powershell
cd .\backend
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Desktop:

```powershell
cd .\desktop
npm run dev
```

## Сборка Windows-приложения

### Собрать backend.exe

```powershell
.\scripts\build_backend.ps1
```

### Собрать Electron приложение

```powershell
.\scripts\build_desktop.ps1
```

Итоговый Windows bundle будет в `desktop/dist-electron/`.

### Сделать единый установщик

1. Установите Inno Setup.
2. Откройте `installer/NurCRM.iss`.
3. Соберите инсталлятор.

Готовый файл появится в `build/installer/NurCRM-Manablock-Setup.exe`. Его уже можно передавать через Telegram как один установщик.

## Ограничения текущего каркаса

- Для полной сборки heavy-модуля нужен установленный Rust.
- Подпись установщика и автообновления не добавлены.
- Лицензирование не реализовано, приложение работает локально без внешних серверов.
