https://drive.google.com/drive/folders/1bKN5V22Pi_i0-_WgL9vaYtPAOOBj0IuA?usp=drive_link

# Задание 1. Анализ, идентификация проблем и решений, планирование

https://docs.google.com/document/d/19I-g_HGoQtsCeytaSAqCylKRLzWDdGoekocEaAJKfdw/edit?usp=sharing

## Исходная информация

MES (от англ. manufacturing execution system), система управления производственными процессами — специализированное
прикладное программное обеспечение, предназначенное для решения задач синхронизации, координации, анализа и оптимизации
выпуска продукции в рамках какого-либо производства. MES-системы относятся к классу систем управления уровня цеха, но
могут использоваться и для интегрированного управления производством на предприятии в целом.

CRM (CRM-система, сокращение от англ. Customer Relationship Management) — Система управления взаимоотношениями с
клиентами - прикладное программное обеспечение для
организаций, предназначенное для автоматизации стратегий
взаимодействия с заказчиками (клиентами), в частности, для повышения уровня продаж, оптимизации маркетинга и улучшения
обслуживания клиентов путём сохранения информации о клиентах и истории взаимоотношений с ними, установления и улучшения
бизнес-процессов и последующего анализа результатов.

[Текущая архитектура](./jewerly_c4_model.drawio)

Акторы:

1. Покупатель (Customer)
2. Продавец (Seller)
3. Пользователь API (API User)

Статусная модель заказа:
INITIATED [онлайн-магазин] — пользователь завёл новый заказ или положил товары в пустую корзину.
FILE_UPLOADED [онлайн-магазин] — пользователь загрузил файл с 3D-моделью или создал его с помощью конструктора.
SUBMITTED [онлайн-магазин] — пользователь нажал на кнопку «Сделать заказ».
PRICE_CALCULATED [MES] — система посчитала стоимость заказа.
MANUFACTURING_APPROVED [CRM] — заказ подтверждён, его можно отдавать в производство.
MANUFACTURING_STARTED [MES] — оператор взял заказ в работу.
MANUFACTURING_COMPLETED [MES] — оператор выполнил заказ.
PACKAGING [MES] — оператор начал упаковывать заказ.
SHIPPED [MES] — заказ отправлен покупателю.
CLOSED [CRM] — заказ завершён. Он закрывается после получения сообщения от транспортной компании или вручную.

Расчёт стоимости в среднем занимает 2-3 минуты (до 30 минут).

## Проблемы

1. Система долго прогружается, когда операторы заходят на первую страницу MES.
2. Время отображение списка заказов велико, в том числе и при применении фильтрации по статусам.
3. Возникают неидентифицированные проблемы при использовании выставленного API
4. После открытия API MES резко возросло количество заказов
5. Время исполнения заказов значительно увеличилось
6. Возросло количество багов уровня high или highest, что приводит к величению срока выпуска нового функционала
7. Shop API и CRM используют одну БД Shop DB

## Инициативы

1. Повысить наблюдаемость системы через внедрение мониторинга.
2. Установить причину проблем с заказами в распределенной системе через внедрение трейсинга.
3. Повысить информативность о работе системы, в частности при возникновении ошибок, через внедрение логирования
4. Повысить отзывчивать системы через применение кешированя

## План на полгода

1. Постоение мониторинга методом RED сервисов Internet Shop, Shop API, CRM, CRM API, MES API, MES и баз данных Shop
   DB и MES db, очередей Messages Queue
2. Кеширование информации (по заказам) в Shop API. Обосновние: решает проблему долгой загрузки страницы, несложная
   задача.

3. Логирование в Internet Shop, Shop API, CRM, CRM API, MES API, MES на стеке ELK
4. Трейсинг

# Задание 2. Мониторинг

## Мотивация

Миниторинг повышает наблюдаемость системы, что дает следующие преимущества:

1. Повышение безопасности системы. Получая полное представление о работе системы, проще обеспечить безопасность.
2. Повышение надёжности системы. Мониторинг позволяет Быстрее определить корень проблемы, следовательно, так её проще 
   устранить и ее предотвратить в будущем.
3. Повышение быстродействия системы. Выявление узких мест помогает оптимизировать производительность.
4. Ускорение рабочего процесса разработки и DevOps. Позволяет оптимизировать процессы и сокращает количества ошибок, 
   что помогает ускорить процесс разработки и улучшить взаимодействие между командами.
5. Получение ценных бизнес-инсайтов. При наблюдении данные накапливаются и они могут привести к бизнес-инсайтам 
   помогать принимать стратегические решения более обоснованно.
6. Повышение удовлетворённости пользователей. Своевременное решение проблем, которые связаны с производительностью и
   функциональностью приложения, улучшает пользовательский опыт.

## Выбор подхода к мониторингу

Метрики RED измеряют показатели, которые важны для конечных пользователей ваших сервисов. Этот подход можно использовать
для мониторинга веб-сервисов, запросов к базам данных, очередей асинхронного взаимодействия. При этом подходе 
измеряются такие типы метрик как частота (Requests Rate) и длительность исполнения (Duration) запросов, а также 
ошибки (Errors), возникающие при обработке запросов.

Метод USE более применим к ресурсам, производительность которых снижается при интенсивном использовании. К ним 
относятся серверы и инструменты. При использовании этого метода измеряются такие типы метрик как уровень использовани 
(Utilization) ресурсов, насыщенность (Saturation) в работе сервиса, которая приводит к накоплению очереди обработки, и 
возникновение ошибок (Errors).

Какие метрики измерять для элементов системы указано в таблице 1. Метрики сгруппированы по методу, что будет 
использовано для формирования дашбордов и настройке алертинга.

Таблица 1. Метрики контейнеров

| Контейнер        | Метрика                         | Метка | Тип           | Метод | Пояснение                                                                                     |
|------------------|---------------------------------|-------|---------------|-------|-----------------------------------------------------------------------------------------------|
| Messages Queue   | Number of "dead" letters        |       | Errors        | USE   | Измеряет количество повторно публикуемых сообщений. Накопление ошибок.                        |
| Messages Queue   | Number of message in flight     |       | Saturation    | USE   | Измеряет количество сообщений в пути. Насыщенность очереди                                    |
| Messages Queue   | CPU %                           |       | Utilization   | USE   | Процент использования процессора. Хватает ли вычислительных мощностей                         |
| Messages Queue   | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
|                  |                                 |       |               |       |
| Shop API         | RPS                             |       | Requests Rate | RED   | Скорость запросов. Определяем как часто происходят запросы                                    |
| Shop API         | Response time                   |       | Duration      | RED   | Длительность исполнения запроса. Насколько быстро отрабатывают запросы                        |
| Shop API         | Number of HTTP 4xx              |       | Errors        | RED   | Количество запросов с необработанной ошибкой. Насколько часто возникает необработанная ошибка |
|                  |                                 |       |               |       |                                                                                               |
| Shop API         | CPU %                           |       | Utilization   | USE   | Процент использования процессора. Хватает ли вычислительных мощностей                         |
| Shop API         | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
| Shop API         | Number of HTTP 2xx              |       | Traffic       | USE   | Количество успешных запросов. Насколько активно используется в штатном режиме                 |
| Shop API         | Number of simultaneous sessions |       | Saturation    | USE   | Количество одновременных подключений                                                          |
| Shop API         | Kb transferred (received)       |       | Traffic       | USE   | Объем получаемых данных. Определяет изменение объема данных от клиентов                       |
| Shop API         | Kb provided (sent)              |       | Traffic       | USE   | Объем отдаваемых данных. Определяет изменение объема данных для клиентов                      |
| Shop API         | Number of HTTP 5xx              |       | Errors        | USE   | Количество некорректных клиентских запросов. Насколько часто возникает необработанная ошибка  |
|                  |                                 |       |               |       |                                                                                               |
| CRM API          | RPS                             |       | Requests Rate | RED   | Скорость запросов. Определяем как часто происходят запросы                                    |
| CRM API          | Response time                   |       | Duration      | RED   | Длительность исполнения запроса. Насколько быстро отрабатывают запросы                        |
| CRM API          | Number of HTTP 4xx              |       | Errors        | RED   | Количество запросов с необработанной ошибкой. Насколько часто возникает необработанная ошибка |
|                  |                                 |       |               |       |                                                                                               |
| CRM API          | CPU %                           |       | Utilization   | USE   | Процент использования процессора. Хватает ли вычислительных мощностей                         |
| CRM API          | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
| CRM API          | Number of HTTP 2xx              |       | Traffic       | USE   | Количество успешных запросов. Насколько активно используется в штатном режиме                 |
| CRM API          | Number of simultaneous sessions |       | Saturation    | USE   | Количество одновременных подключений                                                          |
| CRM API          | Kb transferred (received)       |       | Traffic       | USE   | Объем получаемых данных. Определяет изменение объема данных от клиентов                       |
| CRM API          | Kb provided (sent)              |       | Traffic       | USE   | Объем отдаваемых данных. Определяет изменение объема данных для клиентов                      |
| CRM API          | Number of HTTP 5xx              |       | Errors        | USE   | Количество некорректных клиентских запросов. Насколько часто возникает необработанная ошибка  |
|                  |                                 |       |               |       |                                                                                               |
| MES API          | RPS                             |       | Requests Rate | RED   | Скорость запросов. Определяем как часто происходят запросы                                    |
| MES API          | Response time                   |       | Duration      | RED   | Длительность исполнения запроса. Насколько быстро отрабатывают запросы                        |
| MES API          | Number of HTTP 4xx              |       | Errors        | RED   | Количество запросов с необработанной ошибкой. Насколько часто возникает необработанная ошибка |
|                  |                                 |       |               |       |                                                                                               |
| MES API          | CPU %                           |       | Utilization   | USE   | Процент использования процессора. Хватает ли вычислительных мощностей                         |
| MES API          | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
| MES API          | Number of HTTP 2xx              |       | Traffic       | USE   | Количество успешных запросов. Насколько активно используется в штатном режиме                 |
| MES API          | Number of simultaneous sessions |       | Saturation    | USE   | Количество одновременных подключений                                                          |
| MES API          | Kb transferred (received)       |       | Traffic       | USE   | Объем получаемых данных. Определяет изменение объема данных от клиентов                       |
| MES API          | Kb provided (sent)              |       | Traffic       | USE   | Объем отдаваемых данных. Определяет изменение объема данных для клиентов                      |
| MES API          | Number of HTTP 5xx              |       | Errors        | USE   | Количество некорректных клиентских запросов. Насколько часто возникает необработанная ошибка  |
|                  |                                 |       |               |       |                                                                                               |
| Shop DB          | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
| Shop DB          | Number of connections           |       | Saturation    | USE   | Количество одновременных подключений. Одновременное использование базы данных                 |
| Shop DB          | Size of instance                |       | Utilization   | USE   | Размер базы данных                                                                            |
| Shop DB          | xact_rollback                   |       | Errors        | USE   | Количество откатов и ошибок при транзакциях. Позволит обнаружить ошибки в запросах            |
|                  |                                 |       |               |       |                                                                                               |
| MES DB           | Memory Utilisation              |       | Utilization   | USE   | Потребление оперативной памяти. Достаточно ли оперативной памяти                              |
| MES DB           | Number of connections           |       | Saturation    | USE   | Количество одновременных подключений. Одновременное использование базы данных                 |
| MES DB           | Size of instance                |       | Utilization   | USE   | Размер базы данных                                                                            |
| MES DB           | xact_rollback                   |       | Errors        | USE   | Количество откатов и ошибок при транзакциях. Позволит обнаружить ошибки в запросах            |
|                  |                                 |       |               |       |                                                                                               |
| 3d files storage | Size of storage                 |       | Utilization   | USE   | Размер хранилища                                                                              |
|                  |                                 |       |               |       |                                                                                               |

## План действий

1. Развернуть или инталлировать экпортер Prometheus для Shop API с метками из таблицы 1.
2. Развернуть или инталлировать экпортер Prometheus для MES API с метками из таблицы 1.
3. Развернуть или инталлировать экпортер Prometheus для CRM API с метками из таблицы 1.
4. Развернуть или инталлировать экпортер Prometheus для Shop DB с метками из таблицы 1.
5. Развернуть или инталлировать экпортер Prometheus для MES DB с метками из таблицы 1.
6. Развернуть или инталлировать экпортер Prometheus для 3d files storage с метками из таблицы 1.
7. Развернуть или инталлировать экпортер Prometheus для Messages Queue с метками из таблицы 1.
8. Развернуть инстанс Prometheus для получения временных рядов из экпортеров (пп 1-7)
9. Развернуть инстанс Grafana для визуализации временных рядов из Prometheus
10. Настроить дашборд для Shop API для метода USE (см таблицы 1)
11. Настроить дашборд для Shop API для метода RED (см таблицы 1)
12. Настроить дашборд для Shop MES для метода USE (см таблицы 1)
13. Настроить дашборд для Shop MES для метода RED (см таблицы 1)
14. Настроить дашборд для Shop CRM для метода USE (см таблицы 1)
15. Настроить дашборд для Shop CRM для метода RED (см таблицы 1)
16. Настроить дашборд для Shop DB (см таблицы 1)
17. Настроить дашборд для MES DB (см таблицы 1)
18. Настроить дашборд для 3d files storage (см таблицы 1)
19. Настроить дашборд для Messages Queue (см таблицы 1)

|Контейнер||

# Задание 3. Трейсинг

# Задание 4. Логирование

# Задание 5. Кеширование