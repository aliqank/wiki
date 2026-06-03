# Result Wrapper

**Created:** 2026-06-03  
**Last updated:** 2026-06-03  
**Автор документов:** Telman Nurzhanov (SA)

---

## Назначение

Этот документ описывает общий `result wrapper`, который должен использоваться во всех API-методах системы.

Источник подготовки:
- [`source/results/2026-05-05 - Шаблон обертки результата API.md`](../../../source/results/2026-05-05%20-%20%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%20%D0%BE%D0%B1%D0%B5%D1%80%D1%82%D0%BA%D0%B8%20%D1%80%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%D0%B0%20API.md)

---

## Структура

| № | Описание поля | Наименование поля модели | Тип параметра (backend) | Комментарий |
|---|---|---|---|---|
| 1 | Результат | `value` | `<T>` | Модель, коллекция моделей, `PaginatedResult` или `null` при ошибке |
| 2 | Признак успешности | `isSuccess` | `bool` | Показывает успешность выполнения метода |
| 3 | Ошибки | `errors` | `array<object>` | Коллекция ошибок; при успешном ответе должна быть пустой |
| 3.1 | Сообщение ошибки | `message` | `string` | Текст ошибки, не локализуется внутри wrapper |
| 3.2 | Код ошибки | `code` | `string` | Код ошибки для клиентской обработки и локализации |
| 3.3 | Поле | `property` | `string \| null` | Заполняется для ошибок валидации конкретного поля |
| 3.4 | Признаки / параметры | `tags` | `Dictionary<string, string>` | Дополнительный машинно-читаемый контекст ошибки |

---

## JSON Examples

### Success Response

```json
{
  "value": {
    "field": "value"
  },
  "isSuccess": true,
  "errors": []
}
```

### Success Response With List

```json
{
  "value": [
    {
      "id": "uuid"
    }
  ],
  "isSuccess": true,
  "errors": []
}
```

### Error Response

```json
{
  "value": null,
  "isSuccess": false,
  "errors": [
    {
      "message": "Validation failed",
      "code": "VALIDATION_ERROR",
      "property": "name",
      "tags": {
        "maxLength": "255"
      }
    }
  ]
}
```

---

## Usage Rules

1. Каждый API-метод должен возвращать данные в общем `result wrapper`.
2. Поле `errors` должно присутствовать всегда, даже если массив пустой.
3. Если метод возвращает список с пагинацией, поле `value` должно содержать `PaginatedResult`.
4. При ошибке `value` может быть `null`.
5. Для ошибок валидации нужно заполнять `property`, если ошибка относится к конкретному полю.

---

## Related Artifacts

- [`wiki/api/booking/Booking API summary.md`](../booking/Booking%20API%20summary.md)
- [`source/results/2026-05-05 - Шаблон результата пагинации API.md`](../../../source/results/2026-05-05%20-%20%D0%A8%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%20%D1%80%D0%B5%D0%B7%D1%83%D0%BB%D1%8C%D1%82%D0%B0%D1%82%D0%B0%20%D0%BF%D0%B0%D0%B3%D0%B8%D0%BD%D0%B0%D1%86%D0%B8%D0%B8%20API.md)
