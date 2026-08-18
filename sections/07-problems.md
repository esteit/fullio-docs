# 7. Проблемы

Проблема — единый клиентский процесс для нестандартной ситуации по товару, поставке, заказу, возврату или начислению.

## 7.1. Методы

| Метод             | Назначение                |
|-------------------|---------------------------|
| `problem.list`    | Список проблем            |
| `problem.get`     | Карточка проблемы         |
| `problem.comment` | Добавить комментарий      |
| `problem.respond` | Передать решение клиента  |
| `problem.close`   | Закрыть решённую проблему |

## 7.2. Статусы

`new`, `in_progress`, `waiting_for_client`, `waiting_for_fullio`, `resolved`, `closed`.

Проблема содержит:

- связанную сущность;
- сообщение сотрудника Fullio;
- ответственную сторону;
- требуемое действие;
- допустимые решения;
- комментарии и файлы;
- срок ответа;
- историю;
- `available_actions`.

При создании проблемы сотрудник Fullio заполняет только `message`. Связанная
сущность берётся из текущего контекста, а статус, ответственная сторона,
`required_action`, `allowed_decisions` и срок ответа определяются системой.

## 7.3. `problem.list`

Фильтры: `search`, `status[]`, `requires_client_action`, `related_entity.type`, `related_entity.id`, `created_from`, `created_to`. `search` ищет по сообщению и номеру связанной сущности.

Сортировка: `due_at`, `created_at`, `updated_at`.

Результат содержит `problems[]` и `pagination`.

## 7.4. `problem.get`

```json
{
  "method": "problem.get",
  "params": {
    "problem": {
      "id": 301
    }
  }
}
```

Пример результата:

```json
{
  "result": {
    "problem": {
      "id": 301,
      "status": "waiting_for_client",
      "message": "Не сходится количество, проверьте фактические данные.",
      "related_entity": {
        "type": "inbound",
        "id": 701
      },
      "requires_client_action": true,
      "required_action": "respond",
      "allowed_decisions": ["accept", "dispute", "provide_information"],
      "comments": [],
      "files": [],
      "due_at": "2026-08-20T12:00:00Z",
      "available_actions": ["comment", "respond"],
      "created_at": "2026-08-18T09:00:00Z",
      "updated_at": "2026-08-18T09:00:00Z"
    }
  }
}
```

## 7.5. `problem.comment`

```json
{
  "method": "problem.comment",
  "params": {
    "problem": {
      "id": 301,
      "comment": {
        "text": "Прикладываю копию накладной",
        "files": [
          {
            "id": 902
          }
        ]
      }
    }
  }
}
```

Результат: обновлённый `result.problem`.

## 7.6. `problem.respond`

```json
{
  "method": "problem.respond",
  "params": {
    "problem": {
      "id": 301,
      "decision": "dispute",
      "comment": "По нашим данным отгружено 50 единиц",
      "files": [
        {
          "id": 902
        }
      ]
    }
  }
}
```

`decision` должен присутствовать в `allowed_decisions` актуальной карточки. Ответ переводит проблему в `waiting_for_fullio` или `resolved` в зависимости от сценария.

## 7.7. `problem.close`

Клиент может закрыть только проблему с `available_actions: ["close"]`.

```json
{
  "method": "problem.close",
  "params": {
    "problem": {
      "id": 301
    }
  }
}
```

Закрытие не удаляет историю и связанные файлы.
