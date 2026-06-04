# 9524546 FR-005 Admin creates fleets with unique names

**Created:** 2026-06-03  
**Tester:** Artur Mukhitov  
**BRD:** v13  
**DB Schema:** v12  

---

## TC-001 — Успешное создание флота с уникальным именем (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. Флота с именем "Maintenance Fleet" не существует.

**Steps:**
1. Открыть Admin Panel → раздел управления флотами
2. Нажать "Создать флот"
3. Заполнить: `nameEn` = "Maintenance Fleet", `fleetType` = Internal, `aadGroupId` = валидный ID группы AAD
4. Сохранить

**Expected:**
- Флот создан, ответ 200/201
- В ответе — уникальный `id` (UUID)
- Запись в БД: `isDeleted = 0`, `createdAt` заполнен, `createdBy` = ID текущего Admin
- Новый флот отображается в списке флотов

**Result:** ✅ PASS

---

## TC-002 — Создание флота с дублирующимся именем (Negative)

**Precondition:** Флот с `nameEn` = "Maintenance Fleet" уже существует и активен (`isDeleted = 0`).

**Steps:**
1. Попытаться создать флот с `nameEn` = "Maintenance Fleet"

**Expected:**
- Запрос отклонён, HTTP 409 (или 422)
- Тело ответа: `isSuccess: false`, `errors` содержит код типа `FLEET_NAME_ALREADY_EXISTS`
- Новая запись в БД **не создана**

**Result:** ❌ FAIL — система позволяет создавать неограниченное количество флотов с одинаковым именем, валидация уникальности отсутствует

---

## TC-003 — Имя флота с тем же значением, что у soft-deleted флота

**Precondition:** Флот "Old Fleet" был удалён (`isDeleted = 1`).

**Steps:**
1. Создать новый флот с `nameEn` = "Old Fleet"

**Expected:**
- Флот создан успешно
- Уникальный filtered index (`WHERE isDeleted = 0`) позволяет повторное использование имени удалённого флота

**Result:** 🚫 BLOCKED — заблокирован дефектом TC-002: проверка уникальности не реализована в принципе, тест не имеет смысла до фикса

---

## TC-004 — Попытка создания флота без обязательного поля `nameEn`

**Steps:**
1. Отправить запрос создания флота с пустым / отсутствующим `nameEn`

**Expected:**
- HTTP 400/422
- `isSuccess: false`, ошибка валидации: поле `nameEn` обязательно
- Запись в БД не создана

**Result:** 🔄 RETEST — выяснено, что обязательным полем является `nameRu`, а не `nameEn`. Тест-кейс требует корректировки: повторить с `nameRu` = пусто. `nameEn` — опционально (расхождение со схемой БД v12)

---

## TC-005 — Создание флота с именем из пробелов (whitespace-only)

**Steps:**
1. Передать имя флота = `"   "` (только пробелы)

**Expected:**
- Запрос отклонён (HTTP 400/422)
- Система не принимает blank-строки как валидное имя

**Result:** ❌ FAIL — флот с именем из пробелов успешно создан, валидация отсутствует

---

## TC-006 — Создание флота пользователем без роли Admin (Negative)

**Precondition:** Пользователь авторизован с ролью Requestor или Fleet Owner.

**Steps:**
1. Отправить запрос создания флота через Postman с токеном non-Admin пользователя

**Expected:**
- HTTP 403 Forbidden
- `isSuccess: false`, `errors` содержит `FORBIDDEN`
- Запись в БД не создана

**Result:** ⏳ PENDING — требуется тестирование через Postman

---

## TC-007 — Создание флота неавторизованным пользователем (Negative)

**Steps:**
1. Отправить запрос через Postman без токена авторизации

**Expected:**
- HTTP 401 Unauthorized

**Result:** ⏳ PENDING — требуется тестирование через Postman

---

## TC-008 — Создание Internal флота с привязкой AAD-группы (FR-006)

**Steps:**
1. Создать флот через Postman: `fleetType` = Internal, `aadGroupId` = валидный AAD Group ID

**Expected:**
- Флот создан
- `aadGroupId` сохранён в БД в поле `Fleets.aadGroupId`

**Result:** ⏳ PENDING — требуется тестирование через Postman

---

## TC-009 — Создание Internal флота без `aadGroupId`

**Steps:**
1. Создать флот через Postman: `fleetType` = Internal, `aadGroupId` не указан

**Expected:**
- ⚠️ Требует уточнения: является ли `aadGroupId` обязательным для Internal флотов?
- По BRD (FR-006): привязка AAD-группы обязательна → ожидается HTTP 400/422

**Result:** ⏳ PENDING — требуется тестирование через Postman

---

## TC-010 — Регистронезависимость проверки уникальности имени

**Precondition:** Флот с именем "Maintenance Fleet" существует и активен.

**Steps:**
1. Попытаться создать флот с именем "maintenance fleet" (нижний регистр)
2. Попытаться создать флот с именем "MAINTENANCE FLEET" (верхний регистр)

**Expected:**
- Если uniqueness case-insensitive → оба должны вернуть 409
- Если case-sensitive → оба создадутся (нежелательно с бизнес-точки зрения)

**Result:** 🚫 BLOCKED — заблокирован дефектом TC-002: проверка уникальности не реализована в принципе, регистронезависимость проверить невозможно

---

## TC-011 — Граничное значение длины имени (255 символов)

**Steps:**
1. Создать флот с именем ровно в 255 символов
2. Создать флот с именем в 256 символов

**Expected:**
- 255 символов: флот создан успешно
- 256 символов: HTTP 400/422, ошибка валидации

**Result:** ❌ FAIL — удалось создать запись с именем более 500 символов, ограничение длины не реализовано

---

## TC-012 — Поля `nameRu`, `nameKz` опциональны

**Steps:**
1. Создать флот, передав только `nameEn`, без `nameRu` и `nameKz`

**Expected:**
- Флот создан
- `nameRu = NULL`, `nameKz = NULL` в БД

**Result:** 🔄 RETEST — выяснено, что `nameRu` является обязательным полем, а `nameEn` — опциональным. Тест-кейс требует пересмотра. Расхождение со схемой БД v12 (там `nameEn not null`, `nameRu null`)

---

## TC-013 — Аудит-поля при создании флота

**Steps:**
1. Создать флот с авторизованным Admin-пользователем

**Expected:**
- `Fleets.createdAt` = текущий datetime (UTC)
- `Fleets.createdBy` = `userId` создавшего Admin
- `Fleets.updatedAt = NULL`
- `Fleets.isDeleted = 0`

**Result:** ⏭ NOT TESTED

---

## TC-014 — Созданный флот доступен в справочнике `/reference/fleets`

**Steps:**
1. Создать флот "New Test Fleet"
2. Запросить `GET /api/booking/v1/reference/fleets`

**Expected:**
- Новый флот присутствует в списке с корректными `fleetId` и `fleetName`

**Result:** ⏭ NOT TESTED

---

## Итог прогона

| TC | Название | Результат |
|---|---|---|
| TC-001 | Happy path | ✅ PASS |
| TC-002 | Дубликат имени | ❌ FAIL |
| TC-003 | Имя soft-deleted флота | 🚫 BLOCKED (TC-002) |
| TC-004 | Без обязательного поля | 🔄 RETEST |
| TC-005 | Whitespace-only имя | ❌ FAIL |
| TC-006 | Роль non-Admin | ⏳ PENDING (Postman) |
| TC-007 | Без авторизации | ⏳ PENDING (Postman) |
| TC-008 | С aadGroupId | ⏳ PENDING (Postman) |
| TC-009 | Без aadGroupId | ⏳ PENDING (Postman) |
| TC-010 | Регистронезависимость | 🚫 BLOCKED (TC-002) |
| TC-011 | Граница длины 255 | ❌ FAIL |
| TC-012 | nameRu/nameKz опциональны | 🔄 RETEST |
| TC-013 | Аудит-поля | ⏭ NOT TESTED |
| TC-014 | Флот в /reference/fleets | ⏭ NOT TESTED |

**Найденные дефекты:**
1. **TC-002** — отсутствует валидация уникальности имени флота
2. **TC-005** — принимаются имена из пробелов
3. **TC-011** — отсутствует ограничение длины поля имени (создано >500 символов)
4. **TC-004 / TC-012** — обязательность полей в реализации (`nameRu`) расходится со схемой БД v12 (`nameEn not null`)

---

## Заметки

- TC-006..TC-009 требуют прогона через Postman — нужна помощь с запросами
- TC-004 и TC-012 требуют уточнения у SA: какое поле является обязательным по дизайну — `nameEn` (по DB Schema v12) или `nameRu` (по факту реализации)
- API-спецификация Admin fleet creation в `wiki/api/admin/` отсутствует
