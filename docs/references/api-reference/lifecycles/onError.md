---
status: deprecated
---

# onError

!!!danger "Устарело"

    Утрачена в версии 1.7 для `catchError`.

```ts
import { onError } from 'solid-js';

function onError(fn: (err: any) => void): void;
```

Регистрирует метод-обработчик ошибок, который выполняется при ошибках дочерней области видимости. Выполняются только обработчики ошибок ближайшей области видимости. Выброс для запуска вверх по строке.

## Ссылки

-   [onError](https://docs.solidjs.com/references/api-reference/lifecycles/onError)
