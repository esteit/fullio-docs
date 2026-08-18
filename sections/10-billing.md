# 10. Биллинг

Биллинг показывает текущий период, начисления, счета и оплаты. Юридические файлы счетов и актов в этот контракт не входят.

## 10.1. Методы

| Метод             | Назначение                              |
|-------------------|-----------------------------------------|
| `billing.summary` | Сводка текущего периода и задолженности |
| `charge.list`     | Список начислений                       |
| `charge.get`      | Детализация начисления                  |
| `charge.dispute`  | Открыть спор по начислению              |
| `invoice.list`    | Список счетов                           |
| `invoice.get`     | Счёт и его строки                       |
| `payment.list`    | История оплат                           |

## 10.2. Деньги

Все суммы передаются целым количеством копеек:

```json
{
  "amount": 125050,
  "currency": "RUB"
}
```

Нельзя использовать числа с плавающей точкой.

## 10.3. `billing.summary`

```json
{
  "method": "billing.summary",
  "params": {}
}
```

```json
{
  "result": {
    "billing": {
      "current_period": {
        "date_from": "2026-08-01",
        "date_to": "2026-08-31",
        "accrued": {
          "amount": 8500000,
          "currency": "RUB"
        },
        "disputed": {
          "amount": 120000,
          "currency": "RUB"
        }
      },
      "receivable": {
        "amount": 4300000,
        "currency": "RUB"
      },
      "overdue": {
        "amount": 1500000,
        "currency": "RUB"
      },
      "unpaid_invoices": 2,
      "overdue_invoices": 1,
      "generated_at": "2026-08-18T10:30:00Z"
    }
  }
}
```

## 10.4. Начисления

Типы услуг:

- `receiving`;
- `storage`;
- `order_picking`;
- `order_line`;
- `packing`;
- `packing_material`;
- `marking`;
- `return_processing`;
- `photo_video`;
- `manual_service`;
- `minimum_payment`;
- `adjustment`.

Статусы: `draft`, `billable`, `invoiced`, `disputed`, `cancelled`.

## 10.5. `charge.list`

Фильтры: `status[]`, `service_type[]`, `date_from`, `date_to`, `related_entity.type`, `related_entity.id`, `invoice_id`.

Сортировка: `service_date`, `amount`, `created_at`.

Результат содержит `charges[]`, `totals` и `pagination`.

```json
{
  "result": {
    "charges": [
      {
        "id": 6001,
        "service_date": "2026-08-18",
        "service_type": "receiving",
        "description": "Приёмка поставки SUPPLY-9001",
        "quantity": 50,
        "unit": "item",
        "rate": {
          "amount": 1500,
          "currency": "RUB"
        },
        "amount": {
          "amount": 75000,
          "currency": "RUB"
        },
        "status": "billable",
        "related_entity": {
          "type": "inbound",
          "id": 701
        },
        "invoice": null,
        "available_actions": ["dispute"],
        "created_at": "2026-08-18T09:30:00Z"
      }
    ],
    "totals": {
      "amount": 75000,
      "currency": "RUB"
    },
    "pagination": {
      "next_cursor": null,
      "has_more": false
    }
  }
}
```

## 10.6. `charge.get`

```json
{
  "method": "charge.get",
  "params": {
    "charge": {
      "id": 6001
    }
  }
}
```

Результат `result.charge` дополнительно содержит параметры расчёта, тарифное правило, связанные операции и историю статусов.

## 10.7. `charge.dispute`

```json
{
  "method": "charge.dispute",
  "params": {
    "charge": {
      "id": 6001,
      "reason": "Количество не соответствует акту приёмки",
      "files": [
        {
          "id": 902
        }
      ]
    }
  }
}
```

Метод возвращает обновлённый `result.charge` и созданный `result.problem`. Повторный открытый спор по одному начислению не создаётся.

## 10.8. Счета

Статусы: `issued`, `partially_paid`, `paid`, `overdue`, `cancelled`. Черновые счета клиенту не возвращаются.

`invoice.list` поддерживает `status[]`, `period_from`, `period_to`, `issued_from`, `issued_to`, `payment_due_from`, `payment_due_to`.

Результат содержит `invoices[]` и `pagination`.

## 10.9. `invoice.get`

```json
{
  "method": "invoice.get",
  "params": {
    "invoice": {
      "id": 7001
    }
  }
}
```

`result.invoice` содержит номер, период, даты выставления и оплаты, сумму, оплаченную сумму, задолженность, статус и строки. Каждая строка показывает тип услуги, количество, ставку, сумму и связанные начисления.

Файлы счёта и акта намеренно не входят в текущий контракт.

## 10.10. `payment.list`

Фильтры: `invoice_id`, `paid_from`, `paid_to`. Результат содержит `payments[]` и `pagination`.

Оплата содержит ID, дату, сумму, внешний платёжный номер, назначение и связанный счёт. Создание и изменение оплаты через клиентский API запрещено.
