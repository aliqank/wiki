# GET /reports/usage-rate

**Created:** 2026-05-14  
**Last updated:** 2026-05-14  
**Author:** OpenCode

---

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить данные Usage Rate Dashboard |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Booking / Reporting` |
| Endpoint URL | `/api/booking/v1/reports/usage-rate` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для дашборда usage rate по технике.

---

## 2. Функциональные требования

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-NEW-46 | Usage Rate reports: by day / department / equipment unit | Pending details | BRD v13 | Базовый отчетный метод |
| TCO Booking Tool | FR-NEW-53..63 | Usage Rate Dashboard requirements | Confirmed / Pending | BRD v13 | Покрывает табы, фильтры и агрегаты |

---

## 3. Описание логики работы метода

1. Проверить права доступа пользователя к дашборду.
2. Выбрать список техники и агрегированные показатели usage rate за запрошенный период.
3. Применить фильтры по ownership, department, equipmentType, tab, granularity.
4. Вернуть таблицу и служебные метаданные дашборда.

Сущности:
- читаются: `Equipments`, `EquipmentTypes`, `Bookings`, внешние usage-rate агрегаты

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `FleetOwner` | Usage rate по своей технике |
| `Admin` | Полный usage rate dashboard |

---

## 5. Настройки системы, используемые в методе

| Наименование | Код | Тип значения | Описание | Значение по умолчанию |
|---|---|---|---|---|
| Частота обновления usage rate | `USAGE_RATE_REFRESH_POLICY` | `string` | Справочно для UI: обновление 1 раз в конце рабочего дня | `daily` |

---

## 6. Ошибки, возвращаемые методом

| Код | Описание ошибки |
|---|---|
| `UNAUTHORIZED` | Пользователь не авторизован |
| `FORBIDDEN` | Нет доступа к usage rate dashboard |
| `VALIDATION_ERROR` | Невалидные фильтры периода |

HTTP-коды: `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`

---

## 7. Параметры метода

| № | Описание параметра | Наименование параметра модели | Тип параметра (backend) | Обязательно для заполнения (+ not nullable / - nullable) | Требование валидаций (если требуется) | Значение по умолчанию | Раздел нахождения параметра | Комментарий |
|---|---|---|---|---|---|---|---|---|
| 1 | Дата начала периода | `from` | `date` | `+` | Не позже `to` | — | Query param | |
| 2 | Дата окончания периода | `to` | `date` | `+` | Не раньше `from` | — | Query param | |
| 3 | Таба usage rate | `tab` | `enum` | `-` | `EngineHours / Mileage / Combined` | `Combined` | Query param | |
| 4 | Гранулярность | `granularity` | `enum` | `-` | `Daily / Monthly` | `Daily` | Query param | |
| 5 | Тип владения | `ownershipType` | `enum` | `-` | `TcoOwned / LongTermRented` | — | Query param | |
| 6 | Тип техники | `equipmentTypeId` | `uuid` | `-` | Если передан, должен существовать | — | Query param | |

---

## 8. Пример запроса

```http
GET /api/booking/v1/reports/usage-rate?from=2026-05-01&to=2026-05-31&tab=Combined&granularity=Daily
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается объект `UsageRateDashboardResult`.

Основные поля: `period`, `tab`, `granularity`, `items[]`.

Элемент `items[]`: `equipmentId`, `equipmentNumber`, `equipmentTypeName`, `targetValue`, `averageUsageRate`, `bookingCount`, `cells[]`.

---

## 10. Пример ответа

```json
{
  "value": {
    "period": {
      "from": "2026-05-01",
      "to": "2026-05-31"
    },
    "tab": "Combined",
    "granularity": "Daily",
    "items": [
      {
        "equipmentId": "c3b5af91-61f8-4bc0-bd88-d099d3e90001",
        "equipmentNumber": "TCO-100245",
        "equipmentTypeName": {
          "En": "Excavator",
          "Ru": "Экскаватор",
          "Kz": "Экскаватор"
        },
        "targetValue": 100,
        "averageUsageRate": 82.4,
        "bookingCount": 5,
        "cells": []
      }
    ]
  },
  "isSuccess": true,
  "errors": []
}
```
