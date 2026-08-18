# 4. Остатки и движения

## 4.1. Методы

| Метод                 | Назначение            |
|-----------------------|-----------------------|
| `inventory.list`      | Остатки по товарам    |
| `inventory.get`       | Остаток одного товара |
| `inventory.movements` | История движений      |

Остатки доступны только для чтения. Во внешнем API нет метода ручной установки количества.

## 4.2. Объект `inventory`

```json
{
  "product": {
    "id": 101,
    "sku": "TSHIRT-BLACK-M",
    "name": "Футболка чёрная, размер M"
  },
  "quantities": {
    "total": 18,
    "available": 12,
    "reserved": 4,
    "picked": 0,
    "blocked": 2,
    "damaged": 2,
    "incomplete": 0,
    "returned": 0
  },
  "min_stock_level": 20,
  "is_low_stock": true,
  "last_movement_at": "2026-08-18T09:15:00Z",
  "updated_at": "2026-08-18T09:15:00Z"
}
```

`total` — товар, который ещё числится на складском остатке Fullio. После завершения
упаковки товар исключается из `total`; также в него не входят списанные и утраченные
единицы. `damaged`, `incomplete` и `returned` могут пересекаться с `blocked` и
другими категориями.

`is_low_stock` равен `true`, когда `min_stock_level` больше нуля и
`quantities.available < min_stock_level`. Для расчёта используется доступный к
резервированию остаток по всем партиям товара. Заблокированный, повреждённый и уже
зарезервированный товар в него не входит.

## 4.3. `inventory.list`

Фильтры: `search`, `product_id`, `low_stock`, `has_available`, `has_problem`, `updated_since`.

Сортировка: `sku`, `name`, `available`, `total`, `updated_at`.

```json
{
  "method": "inventory.list",
  "params": {
    "filter": {
      "low_stock": true
    },
    "pagination": {
      "limit": 50
    }
  }
}
```

Результат содержит `inventory[]` и `pagination`.

## 4.4. `inventory.get`

```json
{
  "method": "inventory.get",
  "params": {
    "product": {
      "id": 101
    }
  }
}
```

Результат: `result.inventory`.

## 4.5. `inventory.movements`

Фильтры: `product_id`, `type[]`, `created_from`, `created_to`, `related_entity.type`, `related_entity.id`.

| Тип                   | Описание                                                           |
|-----------------------|--------------------------------------------------------------------|
| `receive`             | Приёмка новой поставки на склад Fullio                             |
| `reserve`             | Перевод доступного товара в резерв конкретного заказа              |
| `release_reservation` | Снятие резерва и возврат товара в доступный остаток                |
| `pack`                | Завершение упаковки заказа и списание товара со складского остатка |
| `return_receive`      | Физическое поступление клиентского возврата на склад               |
| `block`               | Перевод товара в недоступный для резерва остаток                   |
| `unblock`             | Возврат заблокированного товара в доступный остаток                |
| `damage`              | Фиксация повреждения и перевод в повреждённое состояние            |
| `write_off`           | Списание товара из физически учитываемого остатка                  |
| `lost`                | Фиксация утраты товара и уменьшение физического остатка            |
| `adjustment`          | Корректировка по результатам пересчёта или исправления ошибки      |

```json
{
  "result": {
    "movements": [
      {
        "id": 70001,
        "product": {
          "id": 101,
          "sku": "TSHIRT-BLACK-M",
          "name": "Футболка чёрная, размер M"
        },
        "type": "receive",
        "quantity": 50,
        "related_entity": {
          "type": "inbound",
          "id": 701
        },
        "comment": "Приёмка поставки SUPPLY-9001",
        "created_at": "2026-08-18T09:15:00Z"
      },
      {
        "id": 70002,
        "product": {
          "id": 101,
          "sku": "TSHIRT-BLACK-M",
          "name": "Футболка чёрная, размер M"
        },
        "type": "pack",
        "quantity": -2,
        "related_entity": {
          "type": "order",
          "id": 1001
        },
        "comment": "Упаковка заказа 45001 завершена",
        "created_at": "2026-08-20T12:30:00Z"
      }
    ],
    "pagination": {
      "next_cursor": null,
      "has_more": false
    }
  }
}
```

`quantity` — целое ненулевое число. Знак показывает изменение складского остатка
клиента. Для переходов между состояниями передаётся положительный объём операции:
общее количество товара при этом не меняется.
