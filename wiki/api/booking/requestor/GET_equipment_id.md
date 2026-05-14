# GET /equipment/{id}

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить карточку единицы техники в booking-контексте |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Requestor UI` |
| Endpoint URL | `/api/booking/v1/equipment/{id}` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется перед добавлением техники в заявку и при просмотре деталей выбранной единицы.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-041 | Equipment attributes displayed in booking | Confirmed | BRD v13 | Метод возвращает карточку техники |
| TCO Booking Tool | FR-NEW-05 | FO uploads multiple photos; Requestor sees them in request form | Confirmed | BRD v13 | В ответе нужны фото |

---

## 3. Описание логики работы метода

1. Получить запись из `Equipments` WHERE `id = :id` AND `isDeleted = false`.
2. Подтянуть связанные справочники: `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `Locations`, `Fleets`.
3. Подтянуть динамические свойства и фотографии техники.
4. Вернуть агрегированную карточку техники в общем `result wrapper`.

Сущности, участвующие в методе:
- читаются: `Equipments`, `EquipmentTypes`, `EquipmentBrands`, `EquipmentModels`, `EquipmentProperties`, `EquipmentPhotos`, `Locations`, `Fleets`
- изменения не выполняются

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Requestor` | Просмотр техники для создания заявки |
| `ServiceWorkProcessor` | Просмотр техники для Service Work Requests |

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
| `FORBIDDEN` | Нет необходимой роли |
| `NOT_FOUND` | Техника не найдена или удалена |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `404 Not Found`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Идентификатор техники | `id` | `uuid` | `+` | Должен существовать | — | Path param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/equipment/c3b5af91-61f8-4bc0-bd88-d099d3e90001
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

Возвращаемые данные обёрнуты в общий `result wrapper`. В `value` возвращается объект `BookingEquipmentCard`.

Основные поля `BookingEquipmentCard`: `id`, `equipmentNumber`, `equipmentType`, `brand`, `model`, `ownershipType`, `shareType`, `baseLocation`, `fleet`, `photos[]`, `properties[]`, `plannedEngineHoursPerDay`, `plannedMileagePerDay`.

---

## 10. Пример ответа

```json
{
  "value": {
    "id": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
    "equipmentNumber": "TCO-100245",
    "equipmentType": {
      "id": "7b4f4b4d-52d4-4a77-b6b7-f2b7d7c81111",
      "name": {
        "En": "Excavator",
        "Ru": "Экскаватор",
        "Kz": "Экскаватор"
      }
    },
    "brand": "CAT",
    "model": "320D",
    "ownershipType": "TcoOwned",
    "shareType": "Shared",
    "photos": [],
    "properties": []
  },
  "isSuccess": true,
  "errors": []
}
```
