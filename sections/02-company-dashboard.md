# 2. Dashboard

## 2.1. Методы

| Метод                 | Назначение                                   |
|-----------------------|----------------------------------------------|
| `dashboard.summary`   | Получить основные показатели                 |
| `dashboard.attention` | Получить объекты, требующие действия клиента |
| `dashboard.activity`  | Получить последние события компании          |

## 2.2. `dashboard.summary`

Параметров нет. Метод возвращает счётчики только тех разделов, которые входят в этот API.

```json
{
  "result": {
    "summary": {
      "inventory": {
        "total": 12540,
        "available": 10920,
        "reserved": 1240,
        "picked": 80,
        "blocked": 300,
        "damaged": 24
      },
      "products": {
        "total": 420,
        "active": 398,
        "draft": 18,
        "blocked": 4,
        "low_stock": 12
      },
      "inbounds": {
        "draft": 2,
        "expected": 3,
        "receiving": 1,
        "problem": 1
      },
      "orders": {
        "draft": 3,
        "new": 14,
        "in_progress": 37,
        "ready_to_ship": 8,
        "shipped_today": 19,
        "problem": 2
      },
      "returns": {
        "received": 4,
        "checking": 2,
        "requires_action": 1
      },
      "billing": {
        "current_period": {
          "amount": 8500000,
          "currency": "RUB"
        },
        "unpaid_invoices": 2,
        "overdue_invoices": 1
      },
      "requires_action": 6,
      "generated_at": "2026-08-18T10:30:00Z"
    }
  }
}
```

`damaged`, `returned` и `blocked` являются разными измерениями и могут пересекаться. Их нельзя складывать для получения `total`.

## 2.3. `dashboard.attention`

Поддерживает `filter.type`, `pagination`. Типы: `product`, `inbound`, `order`, `problem`, `return`, `invoice`.

```json
{
  "result": {
    "attention": [
      {
        "id": "problem:301",
        "type": "problem",
        "priority": "high",
        "title": "Проблема при приёмке поставки SUPPLY-9001",
        "description": "Недостача 4 единиц",
        "entity": {
          "type": "problem",
          "id": 301
        },
        "required_action": "respond",
        "due_at": "2026-08-20T12:00:00Z",
        "created_at": "2026-08-18T09:00:00Z"
      }
    ],
    "pagination": {
      "next_cursor": null,
      "has_more": false
    }
  }
}
```

`priority`: `low`, `normal`, `high`, `urgent`.

## 2.4. `dashboard.activity`

Возвращает клиентские события: создание и изменение статуса товара, поставки, заказа, возврата, проблемы, начисления или счёта. Внутренние складские детали и пользователи склада не раскрываются.

Поддерживает `filter.type`, `filter.created_from`, `filter.created_to`, `pagination`.

```json
{
  "result": {
    "activity": [
      {
        "id": 90001,
        "type": "inbound.status_changed",
        "title": "Поставка принята",
        "entity": {
          "type": "inbound",
          "id": 701
        },
        "created_at": "2026-08-18T09:15:00Z"
      }
    ],
    "pagination": {
      "next_cursor": null,
      "has_more": false
    }
  }
}
```
