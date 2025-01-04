# Архитектурное решение по логированию

Shop API - для всех операций записываем: OperatorID, OrderID, Timestamp
* INFO:
    * INITIATED
    * FILE_UPLOADED, FileId, file size
    * SUBMITTED
* WARN:
    * FILE_SIZE_IS_BIG, FileId, file size
    * FILE_SIZE_IS_SMALL, FileId, file size
* ERROR:
    * FILE_UPLOAD_ERROR, FileId, file size, file format
    * MAKE_ORDER_ERROR
* FATAL
    * UNKNOWN_ERROR, stack trace
    * NO_DB_CONNECTION, stack trace

MES API - для всех операций записываем: OperatorID, OrderID, Timestamp
* INFO:
    * PRICE_CALCULATED, price
    * MANUFACTURING_STARTED
    * MANUFACTURING_COMPLETED
    * PACKAGING
    * SHIPPED
* WARN:
    * PRICE_IS_HIGH, price
    * PRICE_IS_LOW, price
    * NO_OPERATOR_ASSIGNED, duration - заказ не взят в работу более 3х дней
    * NO_CONFIRMATION_MANUFACTURING_STARTED, duration - заказ не взят в работу более 3х дней
    * NO_CONFIRMATION_MANUFACTURING_COMPLETED, duration - заказ в работе более 15 рабочих дней
    * NO_CONFIRMATION_PACAGING, duration - товар на стадии упаковки более 1 дня
    * NO_CONFIRMATION_SHIPPED, duration - товар на стадии отправки более 1 дня
* ERROR:
    * FILE_DOWNLOAD_ERROR, FileID
    * FILE_FORMAT_ERROR, FileId
    * PRICE_CALCULATION_ERROR
* FATAL
    * UNKNOWN_ERROR, stack trace
    * NO_DB_CONNECTION, stack trace
    * NO_MQ_CONNECTION, stack trace

CRM API - для всех операций записываем: SellerID, OrderID, Timestamp
* INFO:
    * MANUFACTURING_APPROVED
    * CLOSED
* WARN:
    * NO_CONFIRMATION_LOGISTIC, duration - нет подтверждения от транспортной компании более 3х дней
    * NO_CONFIRMATION_MANUFACTURING, duration - нет подтверждения заказа более 3х дней
* FATAL
    * UNKNOWN_ERROR, stack trace
    * NO_DB_CONNECTION, stack trace
    * NO_MQ_CONNECTION, stack trace
  
## Мотивация
Расширенное логирование поможет командам (саапорт, разработка) анализировать состояние системы, расследовать инцинденты и выяснять их причины. 

## Предлагаемое решение
Для реализации логирования будем использовать использовать ELK. Для этого разворачиваем:
* Logstash - для агрегации логов и отправки их ElasticSearch
* OpenSearch - для хранения и поиска логов
* Kibana - для визуализации собранных логов

Для сбора логов с микросервисов будем использовать Logback. Логи перенаправляем в сервис Logstash путем конфигурирования logback.xml. При этом  для удобства дублируем лог в стандартный stdout.
* Shop API (SpringBoot)
* CRM API (SpringBoot)
* MES API (.NET)
* Message Queue (RabitMQ)

Для сбора логов с PostgreSQL используем Filebeat. В PostgreSQL включаем логирование в файлы, Filebeat настраиваем на директории логов и отправляем их в Logstash.
* Shop DB (PostgreSQL)
* MES DB (PostgreSQL)

Для сбора логов в S3 хранилище и публикации их в Logstash будем исходить из того какой именно S3 провайдер у нас используется. Так например, MinIO может из коробки перенаправлять логи в удаленный сервис. В случае с Ceph, логи будем вести в локальный файл, при этом настроив Filebeat на отслеживание изменения логов и отправки их в Logstash.