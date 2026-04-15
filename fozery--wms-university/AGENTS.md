# Copilot instructions for wms-university

Purpose: краткие, конкретные указания, чтобы AI-агент мог быстро стать продуктивным в этом NestJS + Drizzle проекте.

1) Быстрый старт
- Запустить локально: `npm install` → `npm run start:dev`.
- Swagger доступен на `/swagger` после запуска (`src/main.ts`).
- Скрипты миграций: `npm run migrate:generate:dev` / `npm run migrate:up:dev` (drizzle-kit).
- Линт/формат: `npm run lint` / `npm run lint:fix` (biome).

2) Архитектурная «карта» (big picture)
- Приложение — NestJS (см. `src/app.module.ts`, `src/main.ts`).
- DB слой — Drizzle ORM поверх `postgres` клиентa через `DrizzleModule` (`src/common/modules/drizzle/drizzle.module.ts`).
  - DB схемы определены в `src/common/modules/drizzle/schema.ts`.
  - Drizzle использует `casing: 'snake_case'` для полей.
- Сессии — Redis (`ioredis`) глобально через `RedisModule` (`src/common/modules/redis/redis.module.ts`).
  - `AuthService` сохраняет сессии в Redis под ключом `user_session:{id}` (`src/modules/auth/auth.service.ts`).
- Конфигурация через `TypedConfigModule` (`src/common/modules/config`).

3) Проектные конвенции и паттерны
- DTO/валидация: используется `typebox` + кастомные пайпы/декораторы (`src/common/typebox` и `TypeboxBody` в контроллерах).
- Модульность: контроллеры/сервисы располагаются в `src/modules/*`. Примеры: `src/modules/auth`, `src/modules/users`, `src/modules/nomenclature` (пустой контроллер — место для реализации).
- DB: таблицы/enum-ы описываются через `drizzle-orm/pg-core` (см. `usersTable`, `roleEnum`).
- Имена файлов/экспорта: предпочитается экспорт массивов контроллеров/providers из `index.ts` в модуле.

4) Авторизация и потоки
- Cookie-сессии: `AuthController.login` ставит `sessionId` cookie, `AuthGuard` читает его и проверяет в Redis (`src/modules/auth/*`, `src/common/guards/auth.guard.ts`).
- Роли: `roleEnum` и `roles.guard.ts` используются для контроля доступа.

5) Что искать перед редактированием
- Пустые/заглушечные файлы: `src/modules/nomenclature/nomenclature.controller.ts` — проверить спецификацию и реализовать.
- Миграции: `migrations/` содержит SQL; при добавлении полей нужно сгенерировать миграцию (`drizzle-kit`).
- Консистентность схем: при изменении `schema.ts` убедиться, что Drizzle миграции соответствуют.

6) Тесты и CI
- Jest настроен для `src` (`package.json` -> `jest.rootDir = src`).
- E2E тесты запускаются через `npm run test:e2e`.
- Добавляем тест на уровень сервиса/контроллера при изменениях API.

7) Примеры задач для AI-агента (как начать)
- Реализовать CRUD для `nomenclature` модулей: создать схемы, контроллеры, сервис, миграции и Swagger-декорации.
- Добавить поле в `usersTable`: обновить `schema.ts`, создать миграцию, обновить DTO и тесты.
- Добавить проверку ролей на определённый эндпоинт: использовать `@Roles(...)` и `roles.guard.ts`.

8) Где оставить изменения и как предлагать PR
- Изменения должны быть минимальны и локализованы (не меняйте стили вне задачи).
- Добавляйте миграции в `migrations/` и обновляйте `drizzle.config.ts` при необходимости.
- В PR: ссылка на спецификацию (`backend-api-specs.md`) и краткий список изменённых файлов.

Если что-то в этом файле неполно или вы хотите, чтобы я включил примеры кода/фрагменты — скажите, укажите приоритеты (эндпоинты или DB), и я обновлю инструкцию.

9) TypeBox: примеры схем и использование в контроллерах

- Конвенция: 1 роут — 1 схема. Для каждого маршрута создаём отдельный файл-схему в `src/modules/<module>/schemas/`.
  - Пример: `src/modules/auth/schemas/login.ts` содержит `loginBodySchema` и экспорт `type LoginBodySchemaType = Static<typeof loginBodySchema>`.

- Пример схемы для `auth/login` (TypeBox):

```ts
// src/modules/auth/schemas/login.ts
import { Type } from 'typebox';
import { Static } from 'typebox';

export const loginBodySchema = Type.Object({
  login: Type.String({ minLength: 1 }),
  password: Type.String({ minLength: 6 }),
});

export type LoginBodySchemaType = Static<typeof loginBodySchema>;
```

- Пример схем для `nomenclature` (create + list queries):

```ts
// src/modules/nomenclature/schemas/createItem.ts
import { Type } from 'typebox';
import { Static } from 'typebox';

export const createItemSchema = Type.Object({
  code: Type.String({ minLength: 1 }),
  name: Type.String({ minLength: 1 }),
  unit: Type.String(),
  type: Type.Union([Type.Literal('material'), Type.Literal('product')]),
  min_quantity: Type.Number({ minimum: 0 }),
  description: Type.Optional(Type.String()),
});

export type CreateItemSchemaType = Static<typeof createItemSchema>;

// src/modules/nomenclature/schemas/listQueries.ts
import { Type } from 'typebox';
export const listQueriesSchema = Type.Object({
  q: Type.Optional(Type.String()),
  limit: Type.Optional(Type.Number()),
  offset: Type.Optional(Type.Number()),
  sort: Type.Optional(Type.String()),
});
export type ListQueriesSchemaType = Static<typeof listQueriesSchema>;
```

- Использование в контроллере с декораторами `TypeboxBody` / `TypeboxQueries`:

```ts
// src/modules/auth/auth.controller.ts (пример использования)
@Post('login')
public async login(
  @TypeboxBody(loginBodySchema) body: LoginBodySchemaType,
  @Res({ passthrough: true }) res: Response
): Promise<void> { /* ... */ }

// src/modules/nomenclature/nomenclature.controller.ts (список)
@Get()
public async list(@TypeboxQueries(listQueriesSchema) queries: ListQueriesSchemaType) {
  // queries.limit / queries.q и т.д.
}
```

- Важно: `TypeboxBody`/`TypeboxQueries` выполняют валидацию и трансформацию до вызова контроллера. Всегда импортируйте `Static<typeof schema>` и используйте его в сигнатуре метода для точной типизации.

10) Структура модуля и экспорты

- Рекомендуемая структура модуля `nomenclature`:

```
src/modules/nomenclature/
  controllers/
    nomenclature.controller.ts
  services/
    nomenclature.service.ts
  schemas/
    createItem.ts
    updateItem.ts
    listQueries.ts
  types/
    item.ts        # API типы (camelCase)
  utils/
    mappers.ts     # mapper db -> api
  index.ts        # экспортирует arrays: controllers, providers
```

- В текущем проекте `@Module()` не создаётся в каждом модуле вручную: вместо этого модули экспортируют массивы контроллеров и провайдеров через `index.ts`, а `src/app.module.ts` собирает их в одном месте.
  - Пример: `src/modules/auth/index.ts` экспортирует `authControllers` и `authProviders`, которые затем добавляются в `src/app.module.ts`.

11) Внутренние типы и мапперы (DB -> API)

- Drizzle использует `snake_case` в БД, а API возвращает `camelCase`. Введите мапперы в `src/modules/*/utils/mappers.ts` для конвертации полей и базовой логики (null -> undefined, преобразование decimal -> string и т.п.).

Пример маппера для `item`:

```ts
// src/modules/nomenclature/utils/mappers.ts
import { InferSelectModel } from 'drizzle-orm';
import { itemsTable } from 'src/common/modules/drizzle/schema';
import { Item } from '../types/item';

export function mapItemDbToApi(dbRow: InferSelectModel<typeof itemsTable>): Item {
  return {
    id: dbRow.id,
    code: dbRow.code,
    name: dbRow.name,
    unit: dbRow.unit,
    type: dbRow.type,
    minQuantity: dbRow.min_quantity,
    description: dbRow.description ?? undefined,
  };
}
```

- Храните API-типы в `src/modules/<module>/types/*.ts` и используйте их в контроллерах, сервисах и Swagger-описаниях.

12) Дополнительные советы специфичные для проекта

- Конфигурация: используйте `TypedConfigService` (`src/common/modules/config/config.module.ts`) вместо прямого `process.env`.
- Сессии: `AuthService` сохраняет сессии в Redis с ключом `user_session:{sessionId}` и контроллер ставит cookie `sessionId` — следите за `SESSION_MAX_AGE`.
- Swagger: применяйте `@ApiSwagger({ request: { body: schema }, response: { 200: resultSchema } })` рядом с методами контроллера для полной автодокументации.

Если хотите, могу: добавить шаблоны файлов (createItem, controller, service, mapper) для `nomenclature` и применить патч автоматически.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FOZERY) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
