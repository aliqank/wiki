# Wiki Navigation

`wiki/` хранит только готовые к разработке артефакты аналитика.

Ключевые точки входа:
- [`brd/BRD.md`](brd/BRD.md) - опубликованный BRD для разработки (`DO NOT EDIT`)
- [`requirements/`](requirements/) - финальные требования
- [`requirements/usecases/Use Cases.md`](requirements/usecases/Use%20Cases.md) - сводка опубликованных use cases
- [`requirements/Aggregated Request Status Rules.md`](requirements/Aggregated%20Request%20Status%20Rules.md) - правила агрегации статусов заявок
- [`requirements/2026-05-22 - Request and Booking Status Semantics.md`](requirements/2026-05-22%20-%20Request%20and%20Booking%20Status%20Semantics.md) - семантика статусов Request / Booking
- [`api/booking/Booking API summary.md`](api/booking/Booking%20API%20summary.md) - сводка API booking-блока
- [`db/`](db/) - опубликованные схемы БД

Принцип работы:
- работа над документами ведется в других папках репозитория;
- в `wiki/` публикуются только финальные, согласованные материалы;
- черновики, заметки, транскрипты встреч и промежуточные версии в `wiki/` не размещаются.
- `source/` и другие не-`wiki` разделы используются как исторические/рабочие источники; их не редактируем для актуализации dev-ready требований.
- актуальные изменения требований, API и правил публикуем новыми или обновленными файлами в `wiki/`.

Разделы:
- [`glossary/`](glossary/) - утвержденный глоссарий терминов;
- [`requirements/`](requirements/) - финальные требования, готовые к передаче в разработку;
- [`requirements/usecases/`](requirements/usecases/) - use cases, готовые к использованию командой разработки;
- [`brd/`](brd/) - финальная версия BRD; [`wiki/brd/BRD.md`](brd/BRD.md) = DO NOT EDIT;
- [`api/`](api/) - готовые API-спецификации.
