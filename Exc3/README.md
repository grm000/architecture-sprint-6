# Задание 3
## Список проблем и рисков
1. Синхронное обращение к внешним API и сохранение результата в синхронном запросе. 
Проблемы:
    - задержки и блокировка при запросах и агрегации данных
    - низкая отказоустойчивость, сбой вызова к одному из API ломает весь процесс агрегации данных.
2. Рассогласованность потоков обработки данных. 
Сбои в синхронных запросах агрегации данных могут повлиять на процесс взаиморасчетов что.
Так же процесс взаиморасчетов приводит к пиковому потреблению ресурсов и может привести к непредсказуемой деградации сервиса.
3. Синхронное взаимодействие между сервисами и распределенное состояние всей системы сильно ограничивают возможности горизонтального масштабирования. Так же в дальнейшем возникнут проблемы с расширением функциональности системы. 
```
Данные проблемы проявляются при потоке данных для 5 страховых компаний. Увеличение количества страховых компаний только усугубят проблемы.
```
Для решения вышеуказанных проблем заменим синхронное взаимодействие между `core-app`, `ins-product-aggregator` и `ins-comp-settlement` на асинхронное. Первичным будет процесс синхронизации с внешними поставщиками. Для этого по расписанию с настраиваемой скважностью (15 мин по умолчанию) запускаем параллельные задания выгрузки данных их внешних сервисов. При этом в случае сбоя одного из запросов или на синхронизацию, данные "догонят" за счет завершения не исполненных событий, после восстановления сервиса. Взаиморасчеты переведем с временной на событийную модель. При этом гарантированность исполнения взаиморасчетов критична поэтому события обрабатываем, используя `Transactional Outbox` 
