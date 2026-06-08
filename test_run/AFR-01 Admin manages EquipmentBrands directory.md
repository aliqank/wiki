# AFR-01 Admin manages EquipmentBrands directory

**Created:** 2026-06-08  
**Tester:** Artur Mukhitov  
**API:** /api/v1/equipment-brand  
**Env:** dev  

---

## Сводка

| PASS | FAIL | WARN | SKIP |
|------|------|------|------|
| 7    | 7    | 4    | 4    |

Всего тест-кейсов: 22 | Покрыто: 18 | Пропущено: 4

---

## TC-001 — GET — Список без параметров (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. В БД есть активные записи EquipmentBrands (isDeleted = false).

**Steps:**
1. Открыть Swagger → EquipmentBrand → GET /api/v{version}/equipment-brand
2. Нажать Try it out
3. Поле version = 1, остальные поля пустые
4. Нажать Execute

**Expected:**
- 200 OK
- Список всех активных брендов (isDeleted = false)
- Записи отсортированы по sortOrder ASC, затем nameRu ASC
- Поля: id, name.ru, name.kz, name.en, sortOrder

**Result:** ✅ PASS — Вернулся 200 OK. Записи: Ekomak, Toyota, KIA. Все поля присутствуют. Замечание: у всех записей sortOrder = 1, проверить корректность тестовых данных.

---

## TC-002 — GET — Поиск по searchTerm (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. В БД есть бренд Toyota.

**Steps:**
1. GET /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1, searchTerm = toy
4. Нажать Execute

**Expected:**
- 200 OK
- Возвращается только Toyota (совпадение по nameEn)
- total = 1
- Поиск case-insensitive

**Result:** ✅ PASS — Вернулся 200 OK. items: [Toyota], total: 1. Поиск работает корректно.

---

## TC-003 — GET — Пагинация (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. В БД 3 активных бренда.

**Steps:**
1. GET /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1, PageIndex = 2, PageSize = 2
4. Нажать Execute

**Expected:**
- 200 OK
- Вернулась 1 запись (третья страница из 3 при размере 2)
- total = 3 (общее число активных записей)

**Result:** ✅ PASS — Вернулся 200 OK. items: [KIA], total: 3. Пагинация работает корректно.

---

## TC-004 — GET — Запрос без токена

**Precondition:** Токен не передаётся в запросе.

**Steps:**
1. Открыть Swagger → Authorize → Logout у Bearer
2. GET /api/v{version}/equipment-brand
3. Нажать Execute без авторизации

**Expected:**
- 401 Unauthorized

**Result:** ✅ PASS — Вернулся 401 Unauthorized.

---

## TC-005 — GET — Запрос без роли Admin

**Precondition:** Пользователь авторизован без роли Admin.

**Steps:**
1. Убрать роль Admin у пользователя
2. Авторизоваться в Swagger с токеном без роли Admin
3. GET /api/v{version}/equipment-brand
4. Нажать Execute

**Expected:**
- 403 Forbidden

**Result:** ❌ FAIL — БАГ: Авторизация по роли Admin не проверяется. Вернулся 200 OK. Список брендов возвращается без проверки роли. Любой авторизованный пользователь может получить список брендов.

---

## TC-006 — POST — Создание бренда (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. Бренд Caterpillar не существует.

**Steps:**
1. POST /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1
4. Body: `{ name: { ru: "Катерпиллер", kz: "Катерпиллер", en: "Caterpillar" }, sortOrder: 0 }`
5. Нажать Execute

**Expected:**
- 201 Created
- Запись создана, возвращается новый id (UUID)
- sortOrder = MAX(sortOrder) + 1

**Result:** ⚠ WARN — Вернулся 200 OK (не 201). isSuccess: true. id: 28cd8543-7acc-4d21-76d8-08dec54e8414. Замечание: возвращается 200 вместо 201 Created.

---

## TC-007 — POST — Отсутствует nameRu

**Precondition:** Пользователь авторизован с ролью Admin.

**Steps:**
1. POST /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1
4. Body: `{ name: { ru: "", kz: "Катерпиллер", en: "Caterpillar" }, sortOrder: 0 }`
5. Нажать Execute

**Expected:**
- 400 Bad Request
- Ошибка валидации на поле name.ru

**Result:** ❌ FAIL — БАГ: Валидация обязательного поля nameRu не работает. Вернулся 200 OK. Запись создана с пустым name.ru.

---

## TC-008 — POST — Отсутствует nameKz

**Precondition:** Пользователь авторизован с ролью Admin.

**Steps:**
1. POST /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1
4. Body: `{ name: { ru: "Тест", kz: "", en: "Test" }, sortOrder: 0 }`
5. Нажать Execute

**Expected:**
- 400 Bad Request
- Ошибка валидации на поле name.kz

**Result:** ❌ FAIL — БАГ: Валидация обязательного поля nameKz не работает. Вернулся 200 OK. Запись создана с пустым name.kz.

---

## TC-009 — POST — Дублирующий nameEn

**Precondition:** Пользователь авторизован с ролью Admin. Бренд Caterpillar уже существует.

**Steps:**
1. POST /api/v{version}/equipment-brand
2. Нажать Try it out
3. version = 1
4. Body с nameEn = Caterpillar (дубликат)
5. Нажать Execute

**Expected:**
- 409 Conflict
- Дубликат не создаётся

**Result:** ❌ FAIL — БАГ: Уникальность nameEn не проверяется. Вернулся 200 OK. Дубликат создан. Возможно создание дублирующих записей.

---

## TC-010 — POST — Запрос без роли Admin

**Precondition:** Пользователь авторизован без роли Admin.

**Steps:**
1. Убрать роль Admin у пользователя
2. POST /api/v{version}/equipment-brand с валидным телом
3. Нажать Execute

**Expected:**
- 403 Forbidden

**Result:** ❌ FAIL — БАГ: Авторизация по роли Admin не проверяется для POST. Вернулся 200 OK. Запись создана. Любой пользователь может создавать бренды.

---

## TC-011 — PUT — Обновление nameRu (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. Бренд с id 28cd8543-7acc-4d21-76d8-08dec54e8414 существует.

**Steps:**
1. PUT /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. id = 28cd8543-7acc-4d21-76d8-08dec54e8414, version = 1
4. Body: `{ name: { ru: "Катерпиллер Обновлённый", kz: "Катерпиллер", en: "Caterpillar" }, sortOrder: 0 }`
5. Нажать Execute
6. Проверить через GET searchTerm=Катерпиллер

**Expected:**
- 200 OK
- name.ru обновлено
- updatedAt и updatedBy заполнены в БД

**Result:** ✅ PASS — Вернулся 200 OK. isSuccess: true. GET подтвердил: name.ru = Катерпиллер Обновлённый. Audit fields (updatedAt/updatedBy) не проверены — нет доступа к БД.

---

## TC-012 — PUT — Обновление sortOrder (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. Существующий бренд.

**Steps:**
1. PUT /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. Указать валидный id, version = 1
4. Body с sortOrder = 1
5. Нажать Execute
6. Проверить GET — порядок изменился

**Expected:**
- 200 OK
- sortOrder обновлён
- GET-список отражает новый порядок

**Result:** ✅ PASS — Вернулся 200 OK. isSuccess: true. Поле sortOrder обновлено.

---

## TC-013 — PUT — Несуществующий id

**Precondition:** Пользователь авторизован с ролью Admin.

**Steps:**
1. PUT /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. id = 00000000-0000-0000-0000-000000000999 (несуществующий), version = 1
4. Валидное тело
5. Нажать Execute

**Expected:**
- 404 Not Found

**Result:** ⚠ WARN — Вернулся 409 Conflict с сообщением: Entity not found, EntityName: EquipmentBrand. Замечание: возвращается 409 вместо 404 для несуществующей записи.

---

## TC-014 — PUT — Мягко удалённый id

**Precondition:** Пользователь авторизован с ролью Admin. Бренд с указанным id soft-deleted (isDeleted = true).

**Steps:**
1. PUT /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. id = удалённого бренда, version = 1
4. Валидное тело
5. Нажать Execute

**Expected:**
- 404 Not Found (или 410 Gone)

**Result:** ⚠ WARN — Вернулся 409 Conflict. Замечание: возвращается 409 вместо 404 для удалённой записи.

---

## TC-015 — PUT — Пустое тело

**Precondition:** Пользователь авторизован с ролью Admin.

**Steps:**
1. PUT /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. Указать валидный id, version = 1
4. Body = {}
5. Нажать Execute

**Expected:**
- 400 Bad Request

**Result:** ✅ PASS — Вернулся 400 Bad Request.

---

## TC-016 — DELETE — Бренд без связанных моделей (Happy Path)

**Precondition:** Пользователь авторизован с ролью Admin. Бренд не используется ни в одной записи EquipmentModels.

**Steps:**
1. DELETE /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. id = 28cd8543-7acc-4d21-76d8-08dec54e8414, version = 1
4. Нажать Execute
5. Проверить в БД isDeleted = true

**Expected:**
- 204 No Content
- isDeleted = true в БД
- deletedAt и deletedBy заполнены

**Result:** ⚠ WARN — Вернулся 200 OK (не 204). isSuccess: true. Запись удалена (soft delete). Замечание: возвращается 200 вместо 204 No Content. Audit fields не проверены — нет доступа к БД.

---

## TC-017 — DELETE — Бренд со связанными моделями

**Precondition:** Пользователь авторизован с ролью Admin. Бренд используется хотя бы в одной активной записи EquipmentModels.

**Steps:**
1. DELETE /api/v{version}/equipment-brand/{id}
2. Указать id бренда со связанными моделями
3. Нажать Execute

**Expected:**
- 409 Conflict
- Guard срабатывает, запись не удалена

**Result:** ⏳ SKIP — Пропущено: требуется знание конкретного id бренда с активными моделями в EquipmentModels.

---

## TC-018 — DELETE — Повторное удаление уже удалённого id

**Precondition:** Пользователь авторизован с ролью Admin. Бренд уже удалён (isDeleted = true).

**Steps:**
1. DELETE /api/v{version}/equipment-brand/{id}
2. Нажать Try it out
3. Указать id уже удалённого бренда, version = 1
4. Нажать Execute

**Expected:**
- 404 Not Found

**Result:** ❌ FAIL — БАГ: Повторный DELETE по удалённому id возвращает 200 вместо 404. Вернулся 200 OK. isSuccess: true. Отсутствует проверка isDeleted перед удалением.

---

## TC-019 — DELETE — Запрос без роли Admin

**Precondition:** Пользователь авторизован без роли Admin.

**Steps:**
1. Убрать роль Admin у пользователя
2. DELETE /api/v{version}/equipment-brand/{id} с валидным id
3. Нажать Execute

**Expected:**
- 403 Forbidden

**Result:** ❌ FAIL — БАГ: Авторизация по роли Admin не проверяется для DELETE. Вернулся 200 OK. Запись удалена. Любой пользователь может удалять бренды.

---

## TC-020 — Audit fields — После POST

**Precondition:** Пользователь авторизован с ролью Admin. Доступ к БД.

**Steps:**
1. Выполнить POST создание бренда
2. `SELECT createdAt, createdBy, updatedAt, updatedBy FROM EquipmentBrands WHERE id = '{новый id}'`

**Expected:**
- createdAt заполнен
- createdBy = id текущего пользователя
- updatedAt = NULL
- updatedBy = NULL

**Result:** ⏳ SKIP — Пропущено: требуется прямой доступ к БД.

---

## TC-021 — Audit fields — После PUT

**Precondition:** Пользователь авторизован с ролью Admin. Доступ к БД.

**Steps:**
1. Выполнить PUT обновление бренда
2. `SELECT updatedAt, updatedBy FROM EquipmentBrands WHERE id = '{id}'`

**Expected:**
- updatedAt заполнен
- updatedBy = id текущего пользователя

**Result:** ⏳ SKIP — Пропущено: требуется прямой доступ к БД.

---

## TC-022 — Audit fields — После DELETE

**Precondition:** Пользователь авторизован с ролью Admin. Доступ к БД.

**Steps:**
1. Выполнить DELETE бренда
2. `SELECT isDeleted, deletedAt, deletedBy FROM EquipmentBrands WHERE id = '{id}'`

**Expected:**
- isDeleted = true
- deletedAt заполнен
- deletedBy = id текущего пользователя

**Result:** ⏳ SKIP — Пропущено: требуется прямой доступ к БД.
