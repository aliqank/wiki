**Created:** 2026-05-12  
**Last updated:** 2026-05-12  
**Author:** Telman Nurzhanov (SA)

---

# GET /maintenance-partners

## Карточка метода

| Параметр | Значение |
|---|---|
| Описание | Получить постраничный список партнеров ТО |
| Доступ только авторизованным пользователям | `+` |
| Модуль системы | `Admin Panel` |
| Endpoint URL | `/api/admin/v1/maintenance-partners` |
| Метод запроса | `GET` |
| Согласовано | |

---

## 1. Задачи, в рамках которых вносятся изменения в метод

Новый метод. Используется для экрана управления справочником партнеров технического обслуживания.

---

## 2. Функциональные требования

Общий перечень требований: Требования к системе DMMS

| Наименование проекта | Номер требования | Описание требования | Статус | Источник | Комментарий |
|---|---|---|---|---|---|
| TCO Booking Tool | FR-014 | Admin редактирует параметры техники и связанные справочные сущности | Confirmed | BRD v13 | Метод нужен для управления справочником партнеров ТО |
| TCO Booking Tool | FR-NEW-50 | Admin manages all reference/handbook values via Admin Panel | Confirmed | BRD v13 | Справочник управляется через Admin Panel |

---

## 3. Описание логики работы метода

1. Получить записи из `MaintenancePartners` WHERE `isDeleted = false`.
2. Если передан `search`, применить фильтр по `name` и `contactInfo`.
3. Для каждой записи рассчитать `contractsCount` = COUNT(`EquipmentMaintenanceContracts` WHERE `partnerId` = `MaintenancePartners.id` AND `isDeleted = false`).
4. Отсортировать по `name ASC`.
5. Вернуть страницу данных в формате общего `result wrapper` с `PaginatedResult` внутри `value`.

Сущности, участвующие в методе:
- читаются: `MaintenancePartners`, `EquipmentMaintenanceContracts`
- изменения не выполняются
- транзакционность не требуется, метод read-only

---

## 4. Разрешения доступа к методу

| Наименование разрешения | Описание разрешения |
|---|---|
| `Admin` | Доступ к административной панели и справочнику партнеров ТО |

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
| 1 | Поиск по названию или контакту | `search` | `string` | `-` | Если передан, используется как фильтр по `name` и `contactInfo` | — | Query param | |
| 2 | Номер страницы | `page` | `int` | `-` | Целое число >= 1 | `1` | Query param | |
| 3 | Размер страницы | `limit` | `int` | `-` | Целое число >= 1 | `20` | Query param | |

---

## 8. Пример запроса

```http
GET /api/admin/v1/maintenance-partners?search=Shell&page=1&limit=20
Authorization: Bearer <token>
Content-Type: application/json
```

---

## 9. Возвращаемые данные

В `value` возвращается `PaginatedResult<MaintenancePartnerListItem>`.

### Структура `MaintenancePartnerListItem`

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Формат | Значение по умолчанию | Источник данных | Комментарий |
|---|---|---|---|---|---|---|---|
| 1 | Идентификатор партнера ТО | `id` | `uuid` | UUID v4 | — | `MaintenancePartners.id` | |
| 2 | Название партнера | `name` | `string` | string | — | `MaintenancePartners.name` | |
| 3 | Контактная информация | `contactInfo` | `string` | string | `null` | `MaintenancePartners.contactInfo` | |
| 4 | Количество контрактов | `contractsCount` | `int` | integer | `0` | COUNT(`EquipmentMaintenanceContracts`) | Нужен для guard удаления |

---

## 10. Пример ответа

```json
{
  "value": {
    "items": [
      {
        "id": "m0000001-0000-4000-8000-000000000001",
        "name": "Shell Service",
        "contactInfo": "shell@example.com",
        "contractsCount": 5
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

1. `contractsCount` используется для guard-логики удаления.
