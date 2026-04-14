# Copilot / AI agent instructions for this repository

Краткие, практические указания, чтобы быстро быть продуктивным в этой кодовой базе.

1. Большая картина
- **Архитектура**: это единый репозиторий API + Telegram-бот + админка.
  - Веб-API: `main.py` (FastAPI, запуск через `uvicorn main:app`).
  - Telegram-бот: `bot_app/` + `bot.py` (Aiogram). Основной entrypoint для бота — `bot_app/main.py`.
  - Админка: `admin/` (SQLAdmin) — представления регистрируются в `main.py` из `admin_views`.
  - Слой услуг (бизнес-логика): `services/*.py` — используйте его для любой логики, не для прямого доступа к БД.
  - DAO/CRUD: `crud/*.py` — все операции с БД должны быть в этих модулях.

2. Ключевые паттерны и конвенции
- **Service-Layer Mandatory**: контроллеры (routers, admin, бот) не должны обращаться к БД напрямую. Вызов: `await SomeService.method(session, ...)`.
- **Сессия как DI**: сессия (`AsyncSession`) передаётся первым аргументом в сервисы. Создание/запрос сессий через `core/database.py` (см. `get_session`, `async_session_maker`).
- **Асинхронность**: проект полностью async (FastAPI + aiogram + httpx). Используйте `async/await` везде.
- **Именования**: сервисы — `*_service.py`, dao — `crud/*.py`, роуты — `routers/*.py`, админ-views — `admin/*.py`.
- **Формат возвращаемых данных**: сервисы часто возвращают словари, подготовленные для бота (см. `services/product_service.py::_to_dict`).

3. Запуск и отладка локально
- Виртуальное окружение: проект ожидает `.venv` с исполняемыми `python` и `uvicorn` (см. `manage.sh`).
- Запуск всех сервисов: `./manage.sh start` (создаёт `uvicorn main:app` и запускает `bot.py`).
- Логи: `logs/server.log` и `logs/bot.log` (manage.sh пишет туда через `nohup`).
- Для запуска API вручную: `uvicorn main:app --reload` (в окружении проекта).

4. Инициализация БД и фоновые задачи
- Схема создаётся при старте приложения в `main.py` в `lifespan()` через `await init_db()` (см. `core/database.py`).
- Фоновые задачи (ценовые синки и т.п.) запускаются из `main.py` (например, `services/scheduler_service.py`).

5. Интеграции и секреты
- Google API (Docs/Drive): `client_secret.json` + `token.json` (в корне). Генерация токена: `python scripts/get_token.py`.
- Ветка `feature/gdocs-service-phase28` содержит изменения, связанные с переходом на OAuth2 (user account) — проверяйте `services/google_service.py` и `services/document_service.py`.

6. Частые изменения, которых стоит избегать
- Не писать SQL/DB-логику в `admin/`, `routers/` или в обработчиках бота — всегда через `crud` + `services`.
- Не блокируйте event-loop (не вызывайте синхронный I/O в async коде без выполнения в executor).

7. Быстрые примеры поиска по коду
- Чтобы найти логику поиска продуктов см. `services/product_service.py` (использует `crud/product.py` и `thefuzz`).
- Для примера регистрации роутеров в боте см. `bot_app/main.py` (порядок регистрации влияет на обработку команд).

8. Что делать при неясностях
- Открывать файлы: `core/config.py` (настройки), `core/database.py` (сессии), `services/` (бизнес-логика), `crud/` (DAO).
- Если требуется работа с Google Drive/Docs — сначала ознакомьтесь с `AI_AGENT_CONTEXT.md` и `scripts/get_token.py`.

Если нужно — могу сократить или расширить этот файл примерами кода (несколько сниппетов), или слить/обновить существующую инструкцию, если вы хотите сохранить старое содержание.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mvnby) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
