---
description: Обертывает tryFn обработчиком ошибок, который срабатывает при возникновении ошибки ниже этой точки
---

# catchError

!!!tip ""

    **Новое с версии v1.7.0**

```ts
import { catchError } from 'solid-js';

function catchError<T>(
    tryFn: () => T,
    onError: (err: any) => void
): T;
```

Обертывает `tryFn` обработчиком ошибок, который срабатывает при возникновении ошибки ниже этой точки. Выполняются только обработчики ошибок ближайшей области видимости. Сброс для запуска вверх по строке.

## Ссылки

-   [catchError](https://docs.solidjs.com/references/api-reference/reactive-utilities/catchError)
