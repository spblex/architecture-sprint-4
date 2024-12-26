# Планирование: анализ, идентификация проблем и поиск решений

## Проблемы
* Клиенты жалуются менеджерам по продажам, что они не получили заказ и над их изделием работают уже несколько месяцев, хотя обещали закончить за три недели.
* Жалобы со стороны пользователей API. Представители компаний ежедневно сообщают, что не получили свои заказы.
* Жалобы от операторов: когда они заходят на первую страницу MES, система долго прогружается.
* Операторам важно видеть самые новые заказы, есть фильтр по статусам и пагинация. 
  * Нет сортировки. 
  * Вероятно размер страницы пагинации подобран не оптимально.
  * Фронтенд написан на React, при этом в команде только один разработчик частично знаком с React.
* Два Java разработчика и один C#, при этом Тимлид бывший C# разработчик. При такой маленькой команде:
  * Нецелесообразно иметь микросервисы на разных технологиях: Java и C#, Vue и React
  * Тимлид бывший C# разработчик в команде из 2х Java инженеров и одного C# 
    
## Инициативы:
### MUST
* Нагрузочное Тестирование (НТ)
  * Развернуть стенд НТ
  * Разработать НФТ с учетом тенденции роста клиентов, определить SLA для операций
  * Обезличить данные с контура ПРОД для стенда НТ
  * Провести ряд НТ:
    * Профили от 100% до 200%
    * Поиск максимума 
    * Подбор оптимального набора ресурсов
  * Скорректировать hardware ресуры в среде ПРОД на базе результатов НТ
  * Горизонтально отмасштабировать микросервисы сервисы в среде ПРОД    
  * Выполнять НТ как обязательный пререквизит каждого релиза
* Выполнить ряд мер по идентификации проблемных мест:
  * Внедрить Мониторинг системы чтобы определить узкие места
  * Внедрить/расширить логирование для определения мест отказа системы
  * Внедрить трассировку системы
* Внедрить бэкенд и фронтенд кэширование

### SHULD
* Выровнять технологический стэк:
  * Перевести MES фронтенд на с React на Vue, т.к. в команде не хватает экспертизы в React
  * Перевести MES API с C# на Java, т.к. в команде 2 Java разработчика и 1 C# - подавляющее большинство сервисов на Java
    * Заменить C# разработчика на Java инженера
    * Оценить необходлимость замены Тимлида на профильного (бывшего Java-разработчика)
* Автоматизировать тест кейсы входящие в регрешен модель
  * Оценить необходимость нанять второго QA чтобы разгрузить на эту задачу имеющегося или нанять профильного AQA
* Собирать производственные метрики
  * % покрытия тест кейсами
  * % покрытия автотестами
  * code coverage
  * количество багов в релиз
  * размер тех долга и тенденция его сжигания

### NICE TO HAVE
* Внедрить DORA-метрики