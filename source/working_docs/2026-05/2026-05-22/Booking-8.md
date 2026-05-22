По статусам брони:

Draft
Submitted, ConfirmedByFo, Confirmed
TransportConfirmed
InProgress
Declined, Revoked, Terminated, Closed
Extended, EquipmentChanged

Предлагаю разделить статусы:

1. жизненный цикл самой брони:

Draft
Submitted
Confirmed
InProgress
Closed
Declined
Revoked
Terminated (мб удалить кажется не нужен)

2. Approval:
  
вынести отдельно таблицу BookingApprovals:
bookingId
FoApproval / SupervisorApproval
status
comment
мб order (очередь кто первый подтверждает)
createdAt
...

Потом можно удалить из Bookings:

requiresSupervisorApproval
supervisorApprovedBy
supervisorApprovedAt
supervisorComment
transportBookingId
declineReason
terminateReason


3. Вынести транспортировку в отдельную таблицу BookingTransport тоже как approvals
и если нужно будет можем дополнить поля для БП транспортировки

нужно удалить transportBookingld