# 9624763 FR-New-50 Admin manages all reference/handbook values via Admin Panel

**Created:** 2026-06-03  
**Tester:** Artur Mukhitov  
**BRD:** v13 | FR-NEW-50  
**DB Schema:** v12  

---

## Охватываемые справочники

FR-NEW-50 покрывает управление всеми справочными сущностями через Admin Panel:

| Справочник | Base URL |
|---|---|
| Work Centers | `POST/PUT/DELETE /api/admin/v1/work-centers` |
| Measurement Units | `POST/PUT/DELETE /api/admin/v1/measurement-units` |
| Properties | `POST/PUT/DELETE /api/admin/v1/properties` |
| Business Partners | `POST/PUT/DELETE /api/admin/v1/business-partners` |
| Maintenance Partners | `POST/PUT/DELETE /api/admin/v1/maintenance-partners` |

> Тест-кейсы сгруппированы по операциям. Каждый TC по умолчанию прогоняется на Work Centers как основном справочнике с API-спецификацией. Отклонения по другим справочникам фиксируются отдельно.

---

## БЛОК 1: Создание записи справочника (CREATE)

### TC-001 — Успешное создание записи (Happy Path)

**Precondition:** Admin авторизован. Work center с кодом "WC-100" не существует.

**Steps:**
1. `POST /api/admin/v1/work-centers`
2. Body:
```json
{
  "code": "WC-100",
  "name": { "En": "Drilling Operations", "Ru": "Буровые работы", "Kz": "Бұрғылау жұмыстары" }
}
```

**Expected:**
- HTTP 201
- `isSuccess: true`
- `value.id` — UUID v4
- `value.code` = "WC-100"
- `value.equipmentTypesCount` = 0
- Запись в БД: `isDeleted = 0`, `createdAt` заполнен, `createdBy` = userId Admin

**Result:** ✅ PASS с замечанием — HTTP 200 вместо ожидаемого 201

---

### TC-002 — Создание с дублирующимся `code` (Negative)

**Precondition:** Work center с `code` = "WC-100" уже существует (`isDeleted = 0`).

**Steps:**
1. `POST /api/admin/v1/work-centers` с `code` = "WC-100"

**Expected:**
- HTTP 422 `VALIDATION_ERROR`
- `isSuccess: false`
- Новая запись в БД не создана

**Result:** ❌ FAIL — HTTP 500 `UNHANDLED: "An error occurred while saving the entity changes"` вместо 422 `VALIDATION_ERROR`; дубль `code` не обработан, сервер падает. Доп. замечания: фактический endpoint `api/v1/work-center` (ед. ч.) vs `api/admin/v1/work-centers` в спеке; 500 не задокументирован в Swagger

---

### TC-003 — Создание с дублирующимся `name.Ru` (Negative)

**Precondition:** Work center с `name.Ru` = "Буровые работы" уже существует.

**Steps:**
1. `POST /api/admin/v1/work-centers` с другим `code`, но тем же `name.Ru`

**Expected:**
- HTTP 422 `VALIDATION_ERROR` — локализованное имя уже существует
- Новая запись не создана

**Result:** ❌ FAIL — HTTP 200, запись создана с дублирующимся `name.Ru`; уникальность по имени не проверяется

---

### TC-004 — Создание без обязательного поля `code` (Negative)

**Steps:**
1. `POST /api/admin/v1/work-centers` без поля `code`

**Expected:**
- HTTP 422 `VALIDATION_ERROR`

**Result:** ❌ FAIL — HTTP 500 `UNHANDLED` (тот же ответ, что TC-002); отсутствие `code` не валидируется, сервер падает

---

### TC-005 — Создание без обязательного поля `name.Ru` (Negative)

**Steps:**
1. `POST /api/admin/v1/work-centers` с `name` = `{ "En": "Test", "Kz": null }` — без `Ru`

**Expected:**
- HTTP 422 `VALIDATION_ERROR` — `name.Ru` обязателен

**Result:** ❌ FAIL — HTTP 200, запись создана без `name.Ru`; обязательность поля не проверяется

---

### TC-006 — Создание с `name.En` и `name.Kz` = null (опциональные поля)

**Steps:**
1. `POST /api/admin/v1/work-centers` с `name` = `{ "En": null, "Ru": "Тест", "Kz": null }`

**Expected:**
- HTTP 201, запись создана
- `name.En = null`, `name.Kz = null` в БД

**Result:** ❌ FAIL — HTTP 200, запись создана с `name.En = null` и `name.Kz = null`; по актуальным требованиям все три поля (`En`, `Ru`, `Kz`) обязательны — валидация отсутствует

---

### TC-007 — Создание с `code` из пробелов (whitespace-only)

**Steps:**
1. Передать `code` = `"   "`

**Expected:**
- HTTP 422 `VALIDATION_ERROR`

**Result:** ❌ FAIL — HTTP 200, запись создана с `code` и всеми тремя `name` полями из пробелов; whitespace не валидируется

---

### TC-008 — Повторное использование `code` удалённой записи

**Precondition:** Work center с `code` = "WC-OLD" soft-удалён (`isDeleted = 1`).

**Steps:**
1. Создать новый work center с `code` = "WC-OLD"

**Expected:**
- HTTP 201 — filtered unique index (`WHERE isDeleted = 0`) позволяет повторное использование

**Result:** ✅ PASS — HTTP 200, повторное использование `code` удалённой записи разрешено

---

## БЛОК 2: Обновление записи справочника (UPDATE)

### TC-009 — Успешное обновление записи (Happy Path)

**Steps:**
1. `PUT /api/admin/v1/work-centers/{id}` с новыми значениями `code` и `name`

**Expected:**
- HTTP 200
- Значения обновлены в БД
- `updatedAt` = текущий datetime, `updatedBy` = userId Admin

**Result:** ✅ PASS — HTTP 200, запись обновлена

---

### TC-010 — Обновление `code` на уже существующий (Negative)

**Steps:**
1. Попытаться обновить work center, указав `code` другого активного work center

**Expected:**
- HTTP 422 `VALIDATION_ERROR`
- Запись не изменена

**Result:** ❌ FAIL — HTTP 500 `UNHANDLED` (тот же ответ, что TC-002/004); дубль `code` при UPDATE не обработан, сервер падает

---

### TC-011 — Обновление несуществующей записи (Negative)

**Steps:**
1. `PUT /api/admin/v1/work-centers/{random-uuid}`

**Expected:**
- HTTP 404 `NOT_FOUND`

**Result:** ❌ FAIL — HTTP 500 `UNHANDLED` вместо 404; несуществующий `id` при PUT не обрабатывается, сервер падает

---

## БЛОК 3: Удаление записи справочника (DELETE / Soft Delete)

### TC-012 — Успешное soft-удаление записи без привязок (Happy Path)

**Precondition:** Work center существует, к нему не привязан ни один Equipment Type.

**Steps:**
1. `DELETE /api/admin/v1/work-centers/{id}`

**Expected:**
- HTTP 204 No Content (без тела ответа)
- В БД: `isDeleted = 1`, `deletedAt` заполнен, `deletedBy` = userId Admin
- Запись не возвращается в `GET /work-centers`
- Запись недоступна для выбора при создании Equipment Type

**Result:** ✅ PASS с замечанием — HTTP 200 вместо ожидаемого 204

---

### TC-013 — Удаление work center, привязанного к активным Equipment Types (Guard)

**Precondition:** К work center привязан хотя бы один активный Equipment Type.

**Steps:**
1. `DELETE /api/admin/v1/work-centers/{id}`

**Expected:**
- HTTP 409 `WORK_CENTER_IN_USE`
- `errors[0].details.equipmentTypesCount` > 0
- Запись в БД не изменена

**Result:** ⏳ SKIP — TODO: нет work center с привязанными Equipment Types для подготовки precondition

---

### TC-014 — Удаление несуществующей записи (Negative)

**Steps:**
1. `DELETE /api/admin/v1/work-centers/{random-uuid}`

**Expected:**
- HTTP 404 `NOT_FOUND`

**Result:** ❌ FAIL — HTTP 200 вместо 404; сервер не проверяет существование записи перед удалением

---

### TC-015 — Повторное удаление уже удалённой записи (Negative)

**Precondition:** Work center уже soft-удалён.

**Steps:**
1. Повторно отправить `DELETE /api/admin/v1/work-centers/{id}`

**Expected:**
- HTTP 404 `NOT_FOUND` — удалённая запись не найдена как активная

**Result:** ❌ FAIL — HTTP 200 вместо 404; повторное удаление не отклоняется

---

## БЛОК 4: Чтение справочника (READ)

### TC-016 — Список возвращает только активные записи

**Steps:**
1. `GET /api/admin/v1/work-centers`

**Expected:**
- В списке присутствуют только записи с `isDeleted = 0`
- Soft-удалённые записи не отображаются

**Result:** ✅ PASS — удалённые записи в списке отсутствуют

---

## БЛОК 5: Ролевой доступ (RBAC)

### TC-017 — Операции справочника без роли Admin (Negative)

**Precondition:** Пользователь с ролью Requestor или Fleet Owner.

**Steps:**
1. `POST /api/admin/v1/work-centers` (Postman)
2. `PUT /api/admin/v1/work-centers/{id}` (Postman)
3. `DELETE /api/admin/v1/work-centers/{id}` (Postman)

**Expected:**
- HTTP 403 `FORBIDDEN` для каждого запроса

**Result:** ❌ FAIL — все три операции (POST, PUT, DELETE) вернули HTTP 200 с non-Admin токеном; RBAC не проверяется. Системный дефект, подтверждён в FR-005 и FR-009

---

### TC-018 — Операции без авторизации (Negative)

**Steps:**
1. Отправить запросы без токена

**Expected:**
- HTTP 401 `UNAUTHORIZED`

**Result:** ✅ PASS — HTTP 401 на все запросы без токена

---

## БЛОК 6: Применимость к другим справочникам

### TC-019 — Те же проверки для Measurement Units

Повторить TC-001, TC-002, TC-005, TC-012 для `POST/DELETE /api/admin/v1/measurement-units`.

**Notes:** Уточнить обязательные поля и guard-условие удаления (аналог `WORK_CENTER_IN_USE`). Фактический endpoint: `api/v1/measurement-unit`. Схема отличается: `displayName` (string) + `sortOrder` (int) вместо `name {En/Ru/Kz}`.

**TC-001 (happy path):** ✅ PASS — HTTP 200 (вместо 201)

**TC-002 (дубль code):** ❌ FAIL — HTTP 500 `UNHANDLED` (аналогично Work Centers)

**TC-005 (без обязательного поля):** ✅ PASS с замечанием — HTTP 400 `ERR-APP-010006: "The Code field is required."`; валидация работает (в отличие от Work Centers), но статус 400 вместо 422 и не задокументирован в Swagger

**TC-012 (soft delete):** ✅ PASS — HTTP 200

**Result (итог TC-019):** TC-001 ✅, TC-002 ❌, TC-005 ✅ с замечанием, TC-012 ✅

---

### TC-020 — Те же проверки для Maintenance Partners

Повторить TC-001, TC-002, TC-005, TC-012 для `POST/DELETE /api/admin/v1/maintenance-partners`.

**Notes:** Фактический endpoint: `api/v1/maintenance-partner`. Схема: `name {ru, kz, en}` + `phoneNumber` + `email` + `address`. Уникальное поле — `address` (не `name`).

**TC-001 (happy path):** ✅ PASS — HTTP 200

**TC-002 (дубль):** ❌ FAIL — HTTP 500 `UNHANDLED` при дублировании `address`

**TC-005 (без обязательного поля):** ❌ FAIL — HTTP 200, запись создана без `name.ru`; валидация отсутствует

**TC-012 (soft delete):** ✅ PASS — HTTP 200

**Result (итог TC-020):** TC-001 ✅, TC-002 ❌ (500), TC-005 ❌ (200), TC-012 ✅

---

### TC-021 — Те же проверки для Business Partners

Повторить TC-001, TC-002, TC-005, TC-012 для `POST/DELETE /api/admin/v1/business-partners`.

**Notes:** Фактический endpoint: `api/v1/business-partner`. Схема: `name`, `number`, `description`, `trn`, `bin`, `country`, `city`, `address`, `email`, `phoneNumber`. `name` — plain string (не локализованная). Уникальность не проверяется — дубль создаётся с HTTP 200.

**TC-001 (happy path):** ✅ PASS — HTTP 200

**TC-002 (дубль):** ❌ FAIL — HTTP 200, дубль создан без ошибки; уникальность не проверяется

**TC-005 (без name):** ❌ FAIL — HTTP 500 `UNHANDLED`

**TC-012 (soft delete):** ✅ PASS — HTTP 200

**Result (итог TC-021):** TC-001 ✅, TC-002 ❌ (200), TC-005 ❌ (500), TC-012 ✅

---

### TC-022 — Те же проверки для Properties

Повторить TC-001, TC-002, TC-005, TC-012 для `POST/DELETE /api/admin/v1/properties`.

**Notes:** Фактический endpoint: `api/v1/property`. Схема: `name {ru, kz, en}` + `dataTypeId` (UUID) + `unitId` (UUID) + `enumValues`. Самая сложная схема из всех справочников.

**TC-001 (happy path):** ✅ PASS — HTTP 200

**TC-002 (дубль):** ❌ FAIL — HTTP 200, дубль создан без ошибки; уникальность не проверяется

**TC-005 (без name.ru):** ❌ FAIL — HTTP 200, запись создана без обязательного поля

**TC-012 (soft delete):** ✅ PASS — HTTP 200

**Result (итог TC-022):** TC-001 ✅, TC-002 ❌ (200), TC-005 ❌ (200), TC-012 ✅

---

## Итог прогона

| TC | Название | Результат |
|---|---|---|
| TC-001 | Создание — happy path (Work Centers) | ✅ PASS (HTTP 200 вместо 201) |
| TC-002 | Дублирующийся code | ❌ FAIL (500 вместо 422) |
| TC-003 | Дублирующийся name.Ru | ❌ FAIL (200, дубль не проверяется) |
| TC-004 | Без поля code | ❌ FAIL (500 вместо 422) |
| TC-005 | Без name.Ru | ❌ FAIL (200, поле не обязательное) |
| TC-006 | name.En и name.Kz = null | ❌ FAIL (200, все три поля должны быть обязательны) |
| TC-007 | code из пробелов | ❌ FAIL (200, whitespace не валидируется) |
| TC-008 | code удалённой записи повторно | ✅ PASS |
| TC-009 | Обновление — happy path | ✅ PASS |
| TC-010 | Обновление code на существующий | ❌ FAIL (500 вместо 422) |
| TC-011 | Обновление несуществующей записи | ❌ FAIL (500 вместо 404) |
| TC-012 | Soft delete — happy path | ✅ PASS (HTTP 200 вместо 204) |
| TC-013 | Guard: удаление привязанного work center | ⏳ SKIP (TODO: нет precondition) |
| TC-014 | Удаление несуществующей записи | ❌ FAIL (200 вместо 404) |
| TC-015 | Повторное удаление | ❌ FAIL (200 вместо 404) |
| TC-016 | GET — только активные записи | ✅ PASS |
| TC-017 | RBAC — non-Admin | ❌ FAIL (200 на все три операции) |
| TC-018 | RBAC — без авторизации | ✅ PASS |
| TC-019 | Measurement Units | TC-001 ✅, TC-002 ❌ (500), TC-005 ✅ (400), TC-012 ✅ |
| TC-020 | Maintenance Partners | TC-001 ✅, TC-002 ❌ (500), TC-005 ❌ (200), TC-012 ✅ |
| TC-021 | Business Partners | TC-001 ✅, TC-002 ❌ (200), TC-005 ❌ (500), TC-012 ✅ |
| TC-022 | Properties | TC-001 ✅, TC-002 ❌ (200), TC-005 ❌ (200), TC-012 ✅ |

---

## Заметки

- По итогам US 9524546: `name.Ru` является обязательным, `name.En` — опциональным. TC-005 и TC-006 подтверждают или опровергают это для всех справочников
- Guard при удалении описан только для Work Centers (`WORK_CENTER_IN_USE`). Для других справочников уточнить guard-условия у SA
- TC-008 может оказаться заблокированным дефектом уникальности из US 9524546 (если уникальность не проверяется — тест не имеет смысла)
