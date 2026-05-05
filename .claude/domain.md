# Контекст домена — TCO HDV/HDE Booking Tool

## Проект
**HDV/HDE Booking Tool** для клиента **TCO (Tengizchevroil)**.
Домен: бронирование тяжёлой техники и оборудования на нефтяном промысле.
Роль пользователя: **бизнес-аналитик / системный аналитик**.

## Среда
Enterprise-окружение TCO: **Azure AAD, JDE E1, PSWS, DataLake, GIS (MAPH/Atlas)**.

## Scope baseline
- В Phase 1 через систему бронируются только **TCO Owned** и **Long-term rented** единицы техники.
- **On-demand BP (Showcase)** не участвует в booking workflow и показывается только как витрина / каталог.
- Внешняя цепочка согласования из исходного BRD не является целевой моделью для текущего scope.

## Ключевые сущности
- **Equipment** — единица техники HDV/HDE
- **Fleet** — парк техники
- **Request** — заявка, объединяющая несколько booking items
- **Booking** — отдельное бронирование конкретной единицы техники
- **Request Item** — элемент заявки, который обрабатывается как самостоятельный booking

## Классификация техники

### Ownership
- **TCO Owned** — техника TCO, цепочка согласования: `Requestor → Fleet Owner`
- **Long-term rented** — долгосрочно арендованная техника, цепочка согласования: `Requestor → Fleet Owner → FleetOwners' Supervisor`
- **On-demand BP (Showcase)** — только витрина, без бронирования в системе

### Usage Status
- **Assigned**
- **Shared without Conditions**
- **Shared with Conditions**

### Mobility
- **Self-propelled**
- **Unwheeled**
- **Stationary**

## Роли
| Роль | Описание |
|------|----------|
| Requestor | Подаёт заявки на технику |
| Service Work Processor (SWP) | Работает с JDE-sourced Service Work Requests |
| Fleet Owner (FO) | Управляет парком и согласует бронирования |
| FleetOwners' Supervisor | Финально согласует Long-term rented бронирования после FO |
| Transportation Responsible | Обрабатывает транспортировку unwheeled техники |
| Admin | Настраивает систему и справочники |

## Цепочки согласования
- **TCO Owned:** `Requestor → Fleet Owner → Confirmed`
- **Long-term rented:** `Requestor + Justification → Fleet Owner → Confirmed by FO → FleetOwners' Supervisor → Confirmed/Declined`
- **Unwheeled:** отдельный транспортный сценарий с ролью `Transportation Responsible`

## Ключевые бизнес-правила
- Приоритет подбора и бронирования — внутренний флот TCO.
- Для **Long-term rented** обязательно поле **Justification** на уровне booking item.
- Для **Assigned** техники доступ к бронированию может быть ограничен Admin-настройками.
- Таймауты согласования FO и Supervisor должны настраиваться через Admin Panel.
- Горизонт бронирования, максимальная длительность и другие временные параметры не должны быть захардкожены.
- Закрытие бронирований — manual only, без автоматического завершения по дате/времени.

## Интеграции
- **Azure AAD** — аутентификация и role/group provisioning
- **JDE E1** — Work Orders / Service Work Requests
- **PSWS** — контактные данные пользователей
- **DataLake / Cosmos DB via API** — телеметрия, usage rate, GIS-related data
- **GIS (MAPH/Atlas)** — карта через iframe-интеграцию

## Источники истины
- **Финальный BRD для разработки:** `wiki/brd/BRD.md`
- **Wiki navigation:** `wiki/navigation.md`
- **Черновые и рабочие заметки:** `Документация в процессе работы с требованиями/`
- **Подготовленные аналитические результаты:** `Результаты claude/`
- **Навигатор по материалам:** `Список материалов.md`

## Примечание по публикации артефактов
- Черновая работа ведётся в `Документация в процессе работы с требованиями/`.
- После аналитической проработки результаты оформляются в `.md` в `Результаты claude/`.
- В `wiki/` материалы переносятся только по прямой команде пользователя.
