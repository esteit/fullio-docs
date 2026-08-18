# Fullio Client API

Документация server-to-server API Fullio для сервиса, реализующего личный кабинет компании-клиента.

API работает через JSON-RPC 2.0. Компания определяется по API-ключу, который должен храниться только на backend сервиса личного кабинета.

## Статус

Версия документации: `0.2.0-draft`.

Документация описывает проектируемый контракт будущего рабочего API. Наличие метода в документации не означает, что он уже реализован в Fullio.

## Разделы

1. [Общие правила](sections/01-general.md)
2. [Компания и dashboard](sections/02-company-dashboard.md)
3. [Товары](sections/03-products.md)
4. [Остатки и движения](sections/04-inventory.md)
5. [Поставки](sections/05-inbounds.md)
6. [Заказы](sections/06-orders.md)
7. [Проблемы](sections/07-problems.md)
8. [Файлы](sections/08-files.md)
9. [Возвраты](sections/09-returns.md)
10. [Биллинг](sections/10-billing.md)
11. [Webhooks](sections/11-webhooks.md)

## Что не входит в контракт

- юридический раздел документов;
- управление интеграциями и API-ключами;
- настройка URL и секрета webhook через клиентский API;
- поддержка и обращения;
- настройки компании, пользователей и ролей;
- внутренние складские методы ТСД и backend Fullio.

## Endpoint

```text
POST /api/v1/jsonrpc
```

```http
Content-Type: application/json
X-Authorization-Token: fullio_...
```

## Первый запрос

```json
{
  "jsonrpc": "2.0",
  "id": "request-1",
  "method": "company.whoami",
  "params": {}
}
```

## Правило изменения документации

Каждый метод полностью описывается в одном предметном разделе: назначение, параметры, результат, ограничения, ошибки и законченный пример. Общие структуры и правила не дублируются, а берутся из раздела [«Общие правила»](sections/01-general.md).
