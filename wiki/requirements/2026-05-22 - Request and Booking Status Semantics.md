# Request and Booking Status Semantics

**Created:** 2026-05-22  
**Last updated:** 2026-05-22  

---

| Entity | Statuses | Closure reasons | Semantics |
|---|---|---|---|
| `Request` | `Draft`, `Submitted`, `InProgress`, `Closed` | `Cancelled`, `Completed` | `Draft` - черновик заявки. `Submitted` - заявка отправлена, есть активные брони, но ни одна еще не была в `InProgress`. `InProgress` - хотя бы одна бронь заявки в `InProgress`. `Closed` - единый terminal lifecycle status. |
| `Request` | `Closed` | `Cancelled` | Используется для заявки, закрытой до перехода в `InProgress`: отмененный draft, либо submitted-заявка, в которой ни одна бронь не дошла до фактического выполнения. |
| `Request` | `Closed` | `Completed` | Используется только для заявки, в которой хотя бы одна бронь уже была в `InProgress`, а затем активных броней не осталось. |
| `Booking` | `Draft`, `Submitted`, `Confirmed`, `InProgress`, `Closed` | `Cancelled`, `Declined`, `Revoked`, `Terminated`, `Completed` | `Draft` - черновик item-а. `Submitted` - item отправлен и ждет обязательных согласований/решения. `Confirmed` - все обязательные согласования пройдены, item готов к старту. `InProgress` - фактическое выполнение началось. `Closed` - единый terminal lifecycle status. |
| `Booking` | `Closed` | `Cancelled` | Draft booking item закрыт вместе с отменой родительской draft-заявки. |
| `Booking` | `Closed` | `Declined` | Бронь отклонена согласующим. |
| `Booking` | `Closed` | `Revoked` | Бронь отозвана requestor-ом после submit, но до необратимого исполнения. |
| `Booking` | `Closed` | `Terminated` | Бронь остановлена/прекращена после подтверждения или в ходе исполнения. |
| `Booking` | `Closed` | `Completed` | Бронь успешно завершена обычным close-сценарием. |
