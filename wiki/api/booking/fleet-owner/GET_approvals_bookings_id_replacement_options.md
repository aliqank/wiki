# GET /approvals/bookings/{id}/replacement-options

**Created:** 2026-06-02  
**Last updated:** 2026-06-02  
**Автор документов:** Telman Nurzhanov (SA)

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить список replacement candidates для замены техники в конкретной брони |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Fleet Owner UI` |
| Endpoint URL | `/api/booking/v1/approvals/bookings/{id}/replacement-options` |
| Метод запроса | `GET` |
| Связанные use cases | [`UC-FO-08 - Замена техники Fleet Owner`](../../../requirements/usecases/Fleet%20Owner/UC-FO-08%20-%20Замена%20техники%20Fleet%20Owner.md) |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется Fleet Owner-ом для загрузки списка допустимых replacement candidates перед заменой техники в брони.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-046 | FO can replace equipment before/after confirmation if booking not yet started | Confirmed | BRD v13 | Подготавливает допустимый набор replacement candidates |
| TCO Booking Tool | FR-047 | FO cannot replace if booking revoked, terminated, or past end date | Confirmed | BRD v13 | Guard-условия применяются до выдачи replacement candidates |
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | В списке replacement candidates должны быть достаточные данные по технике |

---

## 3. Описание логики работы метода

1. Проверить существование исходной брони и права доступа Fleet Owner.
2. Проверить, что статус исходной брони допускает замену техники: `Submitted` или `Confirmed`.
3. Определить `workCenterId`, текущий период брони и текущий `equipmentId` из исходной брони.
4. Сформировать базовый набор replacement candidates из техники, которая:
   - доступна текущему Fleet Owner по [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access);
   - имеет тот же `Work Center`, что и исходная бронь;
   - находится в допустимом business context для booking flow;
   - не совпадает с текущей единицей техники брони.
5. Для каждой replacement candidate проверить отсутствие [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) на период исходной брони.
6. Для каждой replacement candidate рассчитать [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) по активным броням той же техники на тот же период.
7. Исключить из итоговой выдачи технику, нарушающую hard availability restrictions.
8. Вернуть paginated список replacement candidates вместе с conflict indicators.

Сущности:
- читаются: [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings), [`Equipments`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#8-equipments), [`EquipmentTypes`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#1-equipmenttypes), [`EquipmentBrands`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#4-equipmentbrands), [`EquipmentModels`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#5-equipmentmodels), [`EquipmentStatuses`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#10-equipmentstatuses), [`FleetManagePermissions`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#7-fleetmanagepermissions), [`Bookings`](../../../db/2026-05-21%20-%20DB%20Schema%20v12%20%28Azure%20SQL%2C%20Equipments%2C%20Booking%29.md#22-bookings)
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Просмотр replacement candidates для броней fleet-ов, по которым у пользователя есть [Fleet Management Access](../../../glossary/Glossary.md#fleet-management-access) |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Горизонт бронирования | `BOOKING_HORIZON_DAYS` | `int` | Используется при валидации периода исходной брони | Конфигурируется Admin |
| Максимальная длительность брони | `MAX_BOOKING_DURATION_DAYS` | `int` | Используется при повторной проверке периода | Конфигурируется Admin |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к брони |
| `NOT_FOUND` | Бронь не найдена |
| `BOOKING_NOT_CHANGEABLE` | Статус брони не допускает replacement flow |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор брони | `id` | `uuid` | `+` | Должен существовать | — | Path param | |
| 2 | Поисковая строка | `search` | `string` | `-` | Поиск по TCO-номеру, госномеру, бренду, модели | — | Query param | |
| 3 | Номер страницы | `page` | `int` | `-` | >= 1 | `1` | Query param | |
| 4 | Размер страницы | `limit` | `int` | `-` | >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/approvals/bookings/8c4c8b6d-7bc0-41fb-9038-422cf55d1111/replacement-options?search=CAT&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий [`result wrapper`](../../common/Result%20Wrapper.md).

### Структура `result wrapper`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Результат выполнения метода | value | object | PaginatedResult | — | backend aggregation |  |
| 2 | Признак успешности | isSuccess | bool | boolean | — | backend |  |
| 3 | Ошибки | errors | array<object> | object[] | `[]` | backend |  |

### Структура `value.items[]`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор техники | equipmentId | uuid | UUID v4 | — | Equipments.id |  |
| 2 | ТШО-номер техники | tcoId | string | string | — | Equipments.tcoId |  |
| 3 | Госномер | stateNumber | string | string | — | Equipments.stateNumber |  |
| 4 | Бренд | brand | string | string | — | EquipmentBrands |  |
| 5 | Модель | model | string | string | — | EquipmentModels |  |
| 6 | Fleet | fleet | object | object | — | Fleets |  |
| 7 | Work Center | workCenter | object | object | — | EquipmentTypes + WorkCenters | Должен совпадать с Work Center исходной брони |
| 8 | Признак наличия конфликтов | hasConflicts | bool | boolean | `false` | backend overlap calculation from Bookings |  |
| 9 | Количество конфликтов | conflictsCount | int | integer | `0` | backend overlap calculation from Bookings |  |

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "equipmentId": "7a5af8f7-6d0e-4b6f-ae68-f72906f70001",
        "tcoId": "TCO-100245",
        "stateNumber": "KZ 123 ABC 02",
        "brand": "CAT",
        "model": "320D",
        "fleet": {
          "id": "f0000001-0000-4000-8000-000000000001",
          "name": "Maintenance Fleet"
        },
        "workCenter": {
          "id": "12a8b1ce-3aaf-4f55-8ac8-f8cf5d86c222",
          "code": "WC-100",
          "name": {
            "En": "Drilling Operations",
            "Ru": "Буровые работы",
            "Kz": "Бұрғылау жұмыстары"
          }
        },
        "hasConflicts": true,
        "conflictsCount": 1
      }
    ],
    "total": 1
  },
  "isSuccess": true,
  "errors": []
}
```

---

## Замечания

1. Метод предназначен только для Fleet Owner replacement flow и не заменяет generic equipment search.
2. Replacement candidates должны быть ограничены тем же `Work Center`, что и исходная бронь.
3. Техника с [hard availability restrictions](../../../glossary/Glossary.md#hard-availability-restriction) не должна попадать в итоговую выдачу.
4. Наличие [booking conflict context](../../../glossary/Glossary.md#booking-conflict-context) не должно автоматически исключать технику из replacement candidates.
