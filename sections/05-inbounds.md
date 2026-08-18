# 5. Поставки

## 5.1. Методы

| Метод            | Назначение               |
|------------------|--------------------------|
| `inbound.list`   | Список поставок          |
| `inbound.get`    | Карточка поставки        |
| `inbound.create` | Создать черновик         |
| `inbound.update` | Изменить поставку        |
| `inbound.submit` | Передать поставку складу |
| `inbound.cancel` | Отменить поставку        |

## 5.2. Статусы

| Статус      | Значение                                               |
|-------------|--------------------------------------------------------|
| `draft`     | Черновик клиента; поставку можно редактировать         |
| `expected`  | Поставка передана Fullio и ожидается на складе         |
| `receiving` | Склад выполняет приёмку                                |
| `problem`   | К поставке заведена активная проблема                  |
| `completed` | Приёмка завершена, принятый товар поставлен на остаток |
| `cancelled` | Поставка отменена до начала приёмки                    |

При создании проблемы поставка переходит из `receiving` в `problem`. После решения
она возвращается в `receiving`, если приёмка продолжается, либо переходит в
`completed`, если приёмка завершена.

## 5.3. Состав карточки

### 5.3.1. Объект `inbound`

| Поле                       | Тип         | Описание                                                           |
|----------------------------|-------------|--------------------------------------------------------------------|
| `id`                       | integer     | ID Fullio; только в ответе                                         |
| `shipment_number`          | string      | Номер поставки клиента                                             |
| `status`                   | string      | Текущий статус; только в ответе                                    |
| `expected_arrival_date`    | date        | Ожидаемая дата поступления                                         |
| `comment`                  | string/null | Комментарий клиента                                                |
| `quantities`               | object      | Сводные количества; только в ответе                                |
| `lines`                    | array       | Строки `inbound_line`                                              |
| `problems`                 | array       | Краткие объекты [`problem`](07-problems.md); только в ответе       |
| `files`                    | array       | Объекты [`file`](08-files.md); в запросе передаются ссылки по `id` |
| `charges_summary`          | object      | Краткие начисления за приёмку; только в ответе                     |
| `timeline`                 | array       | История статусов; только в ответе                                  |
| `available_actions`        | array       | Доступные действия; только в ответе                                |
| `created_at`, `updated_at` | datetime    | Время создания и изменения; только в ответе                        |

### 5.3.2. Объект `inbound_quantities`

| Поле       | Тип     | Описание                                            |
|------------|---------|-----------------------------------------------------|
| `expected` | integer | Заявлено клиентом                                   |
| `received` | integer | Фактически принято, включая повреждённый товар      |
| `accepted` | integer | Поставлено на доступный или заблокированный остаток |
| `damaged`  | integer | Принято с повреждениями                             |

Значения рассчитываются Fullio как суммы по строкам и не принимаются в запросах.

### 5.3.3. Объект `inbound_line`

| Поле                | Тип         | Описание                                                    |
|---------------------|-------------|-------------------------------------------------------------|
| `id`                | integer     | ID строки; только в ответе                                  |
| `product_id`        | integer     | ID товара; только в `create` и `update`                     |
| `product`           | object      | Товар: `id`, `sku`, `name`; только в ответе                 |
| `expected_quantity` | integer     | Ожидаемое количество                                        |
| `received_quantity` | integer     | Фактически принято; только в ответе                         |
| `accepted_quantity` | integer     | Поставлено на остаток; только в ответе                      |
| `damaged_quantity`  | integer     | Принято с повреждениями; только в ответе                    |
| `lots`              | array       | Созданные при приёмке партии `inbound_lot`; только в ответе |
| `comment`           | string/null | Комментарий клиента к строке                                |

### 5.3.4. Объект `inbound_lot`

`inbound_lot` — часть фактически принятого товара, которую Fullio дальше учитывает
на складе как отдельную партию. Партия связывает будущий остаток с конкретной
поставкой и приёмкой.

У одной строки может быть несколько партий, например если товар пришёл с разными
номерами партии производителя или принимался отдельными заходами. Даже если номер
производителя не указан, Fullio присваивает партии собственный `lot_number`.

| Поле                      | Тип         | Описание                                |
|---------------------------|-------------|-----------------------------------------|
| `id`                      | integer     | ID партии Fullio                        |
| `lot_number`              | string      | Внутренний номер партии Fullio          |
| `manufacturer_lot_number` | string/null | Номер партии, нанесённый производителем |
| `received_quantity`       | integer     | Количество в партии                     |
| `damaged_quantity`        | integer     | Повреждённое количество в партии        |
| `received_at`             | datetime    | Время фактической приёмки               |

Клиент не передаёт `inbound_lot` в `inbound.create` или `inbound.update`: партии
создаются Fullio по факту приёмки и появляются в `inbound.get`. До начала приёмки
`inbound_line.lots` является пустым массивом.

```json
{
  "lots": [
    {
      "id": 801,
      "lot_number": "FULLIO-2026-000801",
      "manufacturer_lot_number": "LOT-A",
      "received_quantity": 30,
      "damaged_quantity": 0,
      "received_at": "2026-08-24T09:15:00Z"
    },
    {
      "id": 802,
      "lot_number": "FULLIO-2026-000802",
      "manufacturer_lot_number": "LOT-B",
      "received_quantity": 20,
      "damaged_quantity": 2,
      "received_at": "2026-08-24T09:40:00Z"
    }
  ]
}
```

Сумма `received_quantity` по партиям равна `inbound_line.accepted_quantity`.
`damaged_quantity` входит в `received_quantity` партии и показывает принятую, но
заблокированную из-за повреждения часть.

### 5.3.5. Проблемы поставки

`inbound.problems` содержит краткое представление общей сущности `problem`:

| Поле                     | Тип         | Описание                        |
|--------------------------|-------------|---------------------------------|
| `id`                     | integer     | ID проблемы                     |
| `status`                 | string      | Статус общей сущности `problem` |
| `message`                | string      | Сообщение сотрудника Fullio     |
| `related_entity`         | object      | Поставка: `type: inbound`, `id` |
| `requires_client_action` | boolean     | Требуется ли действие клиента   |
| `required_action`        | string/null | Требуемое действие              |
| `allowed_decisions`      | array       | Допустимые решения клиента      |

Полная карточка и история доступны через `problem.get`, ответ клиента передаётся
через `problem.respond`. Параллельной сущности и отдельного набора статусов нет.

План, факт и повреждённое количество сравниваются непосредственно в
`inbound.lines`. Личный кабинет подсвечивает несовпадения автоматически. Проблема
не связывается с отдельной строкой и создаётся на всю поставку только тогда, когда
нужен отдельный процесс решения.

### 5.3.6. Объект `inbound_timeline_event`

| Поле          | Тип         | Описание                                                                            |
|---------------|-------------|-------------------------------------------------------------------------------------|
| `id`          | integer     | ID события                                                                          |
| `event`       | string      | `created`, `submitted`, `receiving_started`, `problem`, `completed` или `cancelled` |
| `from_status` | string/null | Предыдущий статус                                                                   |
| `to_status`   | string      | Новый статус                                                                        |
| `comment`     | string/null | Публичное пояснение события                                                         |
| `created_at`  | datetime    | Время события                                                                       |

### 5.3.7. Объект `charges_summary`

| Поле              | Тип     | Описание                        |
|-------------------|---------|---------------------------------|
| `count`           | integer | Количество связанных начислений |
| `amount`          | Money   | Общая сумма начислений          |
| `disputed_amount` | Money   | Сумма оспариваемых начислений   |

Полная детализация доступна через [`charge.list`](10-billing.md#105-chargelist) с
фильтром по связанной поставке.

## 5.4. `inbound.list`

Фильтры: `search`, `status[]`, `expected_arrival_from`, `expected_arrival_to`, `updated_since`.

Сортировка: `shipment_number`, `expected_arrival_date`, `created_at`, `updated_at`.

Результат содержит `inbounds[]` и `pagination`.

## 5.5. `inbound.get`

```json
{
  "method": "inbound.get",
  "params": {
    "inbound": {
      "id": 701
    }
  }
}
```

Результат: `result.inbound`.

## 5.6. `inbound.create`

```json
{
  "method": "inbound.create",
  "params": {
    "inbound": {
      "shipment_number": "SUPPLY-9001",
      "expected_arrival_date": "2026-08-24",
      "comment": "Доставка после 12:00",
      "lines": [
        {
          "product_id": 101,
          "expected_quantity": 50,
          "comment": null
        }
      ],
      "files": [
        {
          "id": 901
        }
      ]
    }
  }
}
```

Номер поставки уникален внутри компании. Один товар может присутствовать в поставке только одной строкой.

## 5.7. `inbound.update`

```json
{
  "method": "inbound.update",
  "params": {
    "inbound": {
      "id": 701,
      "patch": {
        "expected_arrival_date": "2026-08-25",
        "comment": "Новая дата согласована"
      }
    }
  }
}
```

Редактирование разрешено только согласно `available_actions`. Переданный `lines` или `files` является полным новым набором.

## 5.8. `inbound.submit`

Переводит корректный `draft` в `expected`.

```json
{
  "method": "inbound.submit",
  "params": {
    "inbound": {
      "id": 701
    }
  }
}
```

## 5.9. `inbound.cancel`

```json
{
  "method": "inbound.cancel",
  "params": {
    "inbound": {
      "id": 701,
      "reason": "Поставка перенесена на следующий месяц"
    }
  }
}
```

После начала приёмки отмена запрещена. Объект и история не удаляются.

## 5.10. Ответ на проблему при приёмке

Решения `accept`, `dispute` и `provide_information` передаются методом
[`problem.respond`](07-problems.md#76-problemrespond). Ответ клиента не исправляет
складской остаток напрямую: он переводит проблему в процесс проверки Fullio.
Обновлённая проблема затем возвращается в `inbound.problems` и через `problem.get`.
