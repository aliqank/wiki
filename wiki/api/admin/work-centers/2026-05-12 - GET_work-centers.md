**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Author:** Telman Nurzhanov (SA)

---

# GET /work-centers

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список work centers для справочника Admin Panel |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/work-centers` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником work centers в Admin Panel. Возвращает список записей с данными, достаточными для отображения и guard-логики удаления.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-013 | Admin создаёт equipment types / equipment-related master data | Confirmed | BRD v13 | Work centers являются связанным справочником для типов техники |
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для управления справочником work centers |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Work centers управляются через Admin Panel |

---

## 3. Описание логики работы метода

1. Получить записи из `WorkCenters` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `WorkCenters.name.En`, `WorkCenters.name.Ru`, `WorkCenters.name.Kz` и `WorkCenters.code`.
3. Для каждой записи рассчитать `equipmentTypesCount` = COUNT(`EquipmentTypes` WHERE `workCenterId` = `WorkCenters.id` AND `isDeleted = false`).
4. Отсортировать результат по `name.Ru ASC`, затем по `code ASC`.
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `WorkCenters`, `EquipmentTypes`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику work centers |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| — | — | — | Не используются | — |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | У пользователя нет роли `Admin` |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Поисковая строка по коду или названию | `search` | `string` | `-` | Если передан, используется как фильтр по `WorkCenters.name.En`, `WorkCenters.name.Ru`, `WorkCenters.name.Kz` и `WorkCenters.code` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/work-centers?search=WC&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

У метода нет request body.

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper` (использовать во всех API).  
В обёртке, в значении `value`, возвращается `PaginatedResult`.  
Тип элемента коллекции `items`: `WorkCenterListItem`

### Структура `result wrapper` + `PaginatedResult`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | `value` | `object` | `PaginatedResult<WorkCenterListItem>` | — | backend aggregation | Содержит `items` и `total` |
| 1.1 | Элементы текущей страницы | `items` | `array<object>` | `WorkCenterListItem[]` | `[]` | `WorkCenters` + агрегаты | |
| 1.2 | Общее количество записей | `total` | `int` | integer | `0` | COUNT по `WorkCenters` с учётом фильтра | Без учёта размера страницы |
| 2 | Признак успешности | `isSuccess` | `bool` | boolean | — | backend | |
| 3 | Ошибки | `errors` | `array<object>` | `ApiError[]` | `[]` | backend | При успешном ответе — пустой массив |

### Структура `WorkCenterListItem`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор work center | `id` | `uuid` | UUID v4 | — | `WorkCenters.id` | |
| 2 | Код work center | `code` | `string` | string | — | `WorkCenters.code` | |
| 3 | Наименование work center | `name` | `object` | `LocalizedName` | — | `WorkCenters.name` | `{ En, Ru, Kz }` |
| 4 | Количество типов техники | `equipmentTypesCount` | `int` | integer | `0` | COUNT(`EquipmentTypes`) | Нужен для guard удаления |

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
        "code": "WC-100",
        "name": {
          "En": "Drilling Operations",
          "Ru": "Буровые работы",
          "Kz": "Бұрғылау жұмыстары"
        },
        "equipmentTypesCount": 6
      }
    ],
    "total": 8
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод применяет скрытый фильтр `isDeleted = false` как к `WorkCenters`, так и к `EquipmentTypes` при расчёте `equipmentTypesCount`.
2. `equipmentTypesCount` используется во frontend для guard-логики удаления work center.
3. Поле `name` возвращается как локализованный объект `{ En, Ru, Kz }`.
