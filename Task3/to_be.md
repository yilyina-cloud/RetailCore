@startuml
title RetailCore — TO-BE после первого этапа

left to right direction

skinparam shadowing false
skinparam componentStyle rectangle
skinparam packageStyle rectangle
skinparam linetype spline
skinparam defaultFontName Arial
skinparam defaultFontSize 13
skinparam ArrowFontSize 11
skinparam nodesep 60
skinparam ranksep 80

skinparam component {
  BackgroundColor #F5F7FA
  BorderColor #637185
}

actor "Клиент" as Customer
actor "Поддержка" as Support
actor "Бизнес-\nпользователи" as Business

cloud "Маркетплейсы" as Marketplaces
cloud "Платёжные\nпровайдеры" as Payments
cloud "Службы доставки" as Delivery
cloud "Склады" as Warehouses
cloud "BI / отчёты" as BI

rectangle "RetailCore" {

  component "Сайт и мобильное\nприложение" as Channels

  component "Инструменты поддержки\n\nДобавлен просмотр новой истории\nдля пилотной группы" as Admin #D5E8D4

  component "Java-монолит\n\nОформление и обработка заказа\nЦена и промо • Статусы\nОплата • Выбор доставки" as Monolith #DAE8FC

  database "Oracle\n\nЗаказы и история\nПравила и процедуры\nТаблицы обмена" as Oracle #DAE8FC

  component "Существующий обмен\nОчереди и задания" as Async

  component ".NET-сервис\nинтеграций" as DotNet

  component "Python-задания\nаналитики" as Analytics

  package "Новая история заказа — только чтение" {

    component "Сбор подтверждённых\nизменений заказа" as Capture #D5E8D4

    component "Сервис истории\n\nСостояние и причина\nИсточник и время изменения" as History #D5E8D4

    database "Хранилище истории\nКопия данных для просмотра" as HistoryDB #D5E8D4
  }
}

rectangle "Размещение уточнить" {
  component "PromoService" as Promo
  component "Сервис наличия" as Stock
}

Customer --> Channels : заказ / статус
Channels --> Monolith : запросы
Marketplaces --> Monolith : заказы / обновления

Monolith --> Oracle : данные и правила
Monolith --> Promo : расчёт скидки
Monolith ..> Stock : проверка наличия
Monolith --> Payments : платёжная сессия
Monolith --> Delivery : резерв / доставка
Monolith ..> Warehouses : маршрут обмена уточнить

Support --> Admin : разбор обращений
Admin --> Monolith : прежние ручные действия
Admin --> Oracle : прежний просмотр данных

Monolith --> Async : события / задания
Async --> DotNet : партнёрский обмен
Async --> Analytics : выгрузки
DotNet --> Oracle : общие таблицы
DotNet --> Delivery : заявки / статусы
Analytics --> Oracle : прямые выборки
BI --> Oracle : отчётные запросы
Business --> BI : просмотр отчётов

Oracle -[#2E7D32]-> Capture : подтверждённые изменения
Capture -[#2E7D32]-> History : данные пилотных сценариев
History -[#2E7D32]-> HistoryDB : сохранение и чтение истории
Admin -[#2E7D32]-> History : запрос истории

legend bottom
  Зелёный — новые или изменённые части первого этапа.
  Голубой и серый — существующие части системы.

  Бизнес-логика сохраняется.
  Добавляется сбор изменений для истории заказа.
  История не изменяет заказы и не запускает операции.
  Прежний просмотр доступен для возврата и непилотных случаев.

  Oracle → сбор — логический поток данных.
  Способ захвата выбирается после проверки всех источников
  записи: монолита, процедур, заданий и ручных действий.

  В новой истории видны время обновления и неполнота данных.
  Чёрный пунктир — маршрут требует уточнения.
  Ответы на запросы отдельно не показаны.
endlegend

@enduml
