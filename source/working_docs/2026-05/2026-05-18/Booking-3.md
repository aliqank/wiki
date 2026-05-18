1. Переименовать поле "BookingRequests.workOrderJdeId" в "BookingRequests.workOrderNumber", чтобы явно звучало. не только в схеме БД но и в API
2. в ответ "GET /booking-requests/my" добавить поле принадлежности техники внешнему БП (isExternal boolean) в объекте booking 

---

UC-2

1. в описании POST /booking-requests (9 пункт) - кажется часть полей дублируется и в описании result wrapper и в описании value.
кажется эта проблема во всех описаниях API. Исправить везде, wraper должен описывать только верхние уровни, value - все, что внутри value

2. Подготовить описания api для справочников, указанных в UC-2:
справочник типов техники (GET /api/booking/v1/reference/equipment-types);
справочник fleet owners (GET /api/booking/v1/reference/fleet-owners);
справочник work centers (GET /api/booking/v1/reference/work-centers);
справочник / reference values для ownershipType (GET /api/booking/v1/reference/ownership-types);
справочник / reference values для shareType (GET /api/booking/v1/reference/share-types).
GET /api/booking/v1/reference/equipment-types/{equipmentTypeId}/properties.

