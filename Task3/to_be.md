@startuml
title RetailCore — TO-BE: поэтапная трансформация

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

  component "Инструменты поддержки\n\nНовая история для пилотной группы\nПрежний просмотр сохранён" as Admin #D5E8D4

  component "Java-монолит\n\nОформление • Цена и промо\nИзменение статусов\nОплата • Выбор доставки" as Monolith #DAE8FC

  database "Oracle\n\nЗаказы и история\nПравила и процедуры\nТаблицы обмена" as Oracle #DAE8FC

  component "Существующий обмен\nОчереди и задания" as Async
  component ".NET-сервис\nинтеграций" as DotNet
  component "Python-задания\nаналитики" as Analytics

  package "Первый этап — история заказа" {
    component "Сбор подтверждённых\nизменений заказа\n\nСпособ захвата уточняется" as Capture #D5E8D4

    component "Сервис истории\n\nСтатусы, причины,\nисточник и время" as History #D5E8D4

    database "Хранилище истории\nКопия для чтения" as HistoryDB #D5E8D4
  }

  package "Следующие этапы" {
    component "Подготовка данных\nвыбранного отчёта" as ReportJob #FFF2CC

    database "Отдельная витрина\nвыбранного отчёта" as ReportDB #FFF2CC

    component "Адаптер выбранного партнёра\n\nЖурнал отправок\nПроверка повторов" as Adapter #FFF2CC
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
Monolith --> Delivery : действующие сценарии
Monolith ..> Warehouses : маршрут обмена уточнить

Support --> Admin : разбор обращений
Admin --> Monolith : прежние ручные действия
Admin --> Oracle : прежний просмотр

Monolith --> Async : события / задания
Async --> DotNet : партнёрский обмен
DotNet --> Oracle : общие таблицы
DotNet --> Delivery : неперенесённые сценарии
Async --> Analytics : выгрузки
Analytics --> Oracle : неперенесённые выборки
BI --> Oracle : неперенесённые отчёты
Business --> BI : просмотр отчётов

Oracle -[#2E7D32]-> Capture : подтверждённые изменения
Capture -[#2E7D32]-> History : данные выбранных сценариев
History -[#2E7D32]-> HistoryDB : запись и чтение истории
Admin -[#2E7D32]-> History : запрос истории

Capture -[#B8860B,dashed]-> ReportJob : после проверки полноты
ReportJob -[#B8860B,dashed]-> ReportDB : расчёт показателей
BI -[#B8860B,dashed]-> ReportDB : выбранный отчёт

Monolith -[#B8860B,dashed]-> Adapter : после оформления заказа\nвыбранный сценарий
Adapter -[#B8860B,dashed]-> Delivery : заявки / подтверждения
Adapter -[#B8860B,dashed]-> Monolith : результат обработки

legend bottom
  Зелёный — первый этап: история заказа для поддержки.
  Жёлтый — следующие этапы после отдельных проверок.
  Голубой и серый — сохранённые компоненты.

  Сбор изменений должен учитывать все источники записи:
  монолит, процедуры, задания и ручные действия.
  Связь Oracle → сбор обозначает источник данных,
  а не выбранную технологию захвата.

  Новая история не изменяет заказы.
  Для перенесённого партнёрского сценария старую
  отправку отключаем: отправитель должен быть один.

  Чёрный пунктир — маршрут требует уточнения.
  Ответы на запросы отдельно не показаны.
endlegend

@enduml
