# 8. Файлы

Файловый API используется для фото товаров, вложений поставок, фотографий повреждений, проблем, возвратов и лейблов доставки. Это технический файловый контур, а не исключённый юридический раздел документов.

Файлы не передаются в JSON-RPC как base64.

## 8.1. Методы

| Метод                 | Назначение                                     |
|-----------------------|------------------------------------------------|
| `file.prepareUpload`  | Создать загрузку и получить URL                |
| `file.completeUpload` | Подтвердить завершение загрузки                |
| `file.get`            | Получить метаданные и временный URL скачивания |

## 8.2. Статусы

`pending`, `uploaded`, `processing`, `ready`, `rejected`, `expired`.

## 8.3. `file.prepareUpload`

```json
{
  "method": "file.prepareUpload",
  "params": {
    "file": {
      "name": "product-101.jpg",
      "content_type": "image/jpeg",
      "size": 245810,
      "purpose": "product_photo"
    }
  }
}
```

`purpose`: `product_photo`, `inbound_attachment`, `problem_attachment`, `return_attachment`, `shipping_label`.

```json
{
  "result": {
    "file": {
      "id": 901,
      "name": "product-101.jpg",
      "content_type": "image/jpeg",
      "size": 245810,
      "purpose": "product_photo",
      "status": "pending",
      "upload": {
        "method": "PUT",
        "url": "https://upload.example.test/...",
        "headers": {
          "Content-Type": "image/jpeg"
        },
        "expires_at": "2026-08-18T10:45:00Z"
      }
    }
  }
}
```

## 8.4. Загрузка содержимого

Вызывающая система отправляет байты напрямую в `file.upload.url` указанным HTTP-методом и заголовками. JSON-RPC endpoint в передаче содержимого не участвует.

## 8.5. `file.completeUpload`

```json
{
  "method": "file.completeUpload",
  "params": {
    "file": {
      "id": 901
    }
  }
}
```

Fullio проверяет наличие, размер, MIME-тип и безопасность файла. Результат возвращает `result.file`. Использовать файл в сущности можно только после статуса `ready`.

## 8.6. `file.get`

```json
{
  "method": "file.get",
  "params": {
    "file": {
      "id": 901
    }
  }
}
```

Для `ready` возвращается временный `download.url` и `expires_at`. URL не следует сохранять; при истечении вызывается новый `file.get`.

Файл другой компании возвращается как `ENTITY_NOT_FOUND`.
