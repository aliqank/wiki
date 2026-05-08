Create a formatted API description file in `source/results/` following the project's standard template.

**Invocation:** `/describe-api [METHOD /endpoint/path]`  
**Example:** `/describe-api GET /api/admin/v1/equipment-types`

---

## Steps

### 1. Determine method and endpoint

- If arguments were passed (e.g., `GET /api/admin/v1/equipment-types`), use them directly.
- If no arguments, ask the user:
  - "What is the HTTP method? (GET / POST / PUT / PATCH / DELETE)"
  - "What is the endpoint path? (e.g., `/api/admin/v1/equipment-types`)"

### 2. Find source material

Open `materials.md` and identify files relevant to this endpoint or its entity:
- API spec in `source/results/` (look for "API спецификация" or similar)
- DB schema — latest version (currently v6): `source/results/2026-05-08 - Схема БД v6 (Equipments, Bookings).md`
- BRD — current version: `source/results/2026-05-05 - HDV HDE BRD v13.md`
- Any existing wiki file for this endpoint: `wiki/api/...`
- Any working drafts in `source/working_docs/`

Read all relevant files. Extract: entity fields, functional requirements (FR numbers), access permissions, error cases, and business logic.

### 3. Compose the API description

Build the file with ALL of the following sections (mandatory, none may be omitted):

---

#### Карточка метода

Table at the top:

| Параметр | Значение |
|---|---|
| Описание | One-sentence description of what the method does |
| Доступ только авторизованным пользователям | `+` or `-` |
| Модуль системы | e.g., `Admin Panel` |
| Endpoint URL | Full path, e.g., `/api/admin/v1/equipment-types` |
| Метод запроса | `GET` / `POST` / `PUT` / `PATCH` / `DELETE` |
| Согласовано | (leave blank) |

---

#### 1. Задачи, в рамках которых вносятся изменения в метод

Brief context: is this a new method, a change to an existing one, or a publication of an already-described spec?

---

#### 2. Функциональные требования

Preface line: `Общий перечень требований: Требования к системе DMMS`

Table:

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|

Find matching FR numbers in BRD v13. Typically 2–4 rows.

---

#### 3. Описание логики работы метода

Numbered steps describing the backend logic. Always include:
- Step 1: Read from DB (add `WHERE isDeleted = false` if the entity has an `isDeleted` field)
- Steps for filtering, sorting, aggregation (if any)
- Final step: return data in the result wrapper

End with the entities list:

```
Сущности, участвующие в методе:
- читаются: Table1, Table2, ...
- изменяются: Table3 (or: изменения не выполняются)
- транзакционность: required / не требуется
```

---

#### 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|

---

#### 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|

If none used, add a single row: `| — | — | — | Не используются | — |`

---

#### 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|

Standard errors for authorized endpoints:
- `UNAUTHORIZED` / `401 Unauthorized`
- `FORBIDDEN` / `403 Forbidden`

Add method-specific errors as needed (e.g., `NOT_FOUND`, `VALIDATION_ERROR`).

After the table: `HTTP-коды: \`401 Unauthorized\`, \`403 Forbidden\`[, ...]`

---

#### 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|

"Раздел нахождения параметра" values: `Query param`, `Path param`, `Request body`, `Header`

If no parameters: write "У метода нет параметров."

---

#### 8. Пример запроса

````http
METHOD /api/.../endpoint?param=value
Authorization: Bearer <token>
Content-Type: application/json
````

If there is a request body, include a JSON example. Otherwise: "У метода нет request body."

---

#### 9. Возвращаемые данные

State which wrapper is used:
- **All methods**: "Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API)."
- **Paginated GET**: additionally state "В обёртке, в значении `value`, возвращается `PaginatedResult`."
- State the item type: "Тип элемента коллекции `items`: `EntityName`"

Then provide the structure table(s):

**For paginated responses** — two tables:
1. Wrapper + PaginatedResult structure (rows: value, items, total, isSuccess, errors)
2. Item entity structure (named `EntityName`)

**For non-paginated responses** — one table:
1. Wrapper structure (rows: value with concrete type, isSuccess, errors)

Table columns: `№ | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий`

Use nested numbering for nested fields: 1, 1.1, 1.2, 2, 2.1...

---

#### 10. Пример ответа

```json
{
  "value": { ... },
  "isSuccess": true,
  "errors": []
}
```

Use realistic placeholder data. Include both success and (if needed) error examples.

---

#### Замечания (optional)

Add a numbered list if there are non-obvious constraints, DB decisions, or guard logic that developers must know.

---

### 4. Save the file

- **File name format:** `YYYY-MM-DD - METHOD_entity_name.md`
  - Use today's date
  - METHOD in uppercase (GET, POST, etc.)
  - entity_name in snake_case from the endpoint path (e.g., `equipment_types` from `/equipment-types`; append `_id` for `/:id` routes)
  - Example: `2026-05-08 - GET_equipment_types_id.md`
  - No `api_description_` prefix
- **Path:** `source/results/`
- **Header at top of file:**

```
**Created:** YYYY-MM-DD  
**Last updated:** YYYY-MM-DD  
**Author:** Telman Nurzhanov (SA)
```

### 5. Update materials.md

Add a one-line entry in the appropriate section of `materials.md`:

```
| **API description: METHOD /endpoint** | `results/YYYY-MM-DD - METHOD_entity_name.md` — one-line summary |
```

### 6. Report to user

Tell the user the path to the saved file and briefly note what source material was used.
