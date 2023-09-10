---
description: Этот метод принимает сигнал и выдает наблюдаемое значение
---

# observable

```ts
function observable<T>(input: () => T): Observable<T>;
```

Этот метод принимает сигнал и выдает наблюдаемое значение. Вы можете использовать его из другой библиотеки наблюдаемых значений по вашему выбору, обычно с помощью оператора `from`.

```ts
// How to integrate rxjs with a Solid signal
import { observable } from 'solid-js';
import { from } from 'rxjs';

const [s, set] = createSignal(0);

const obsv$ = from(observable(s));

obsv$.subscribe((v) => console.log(v));
```

Вы также можете использовать `from` без `rxjs`; посмотрите эту [страницу](from.md).

## Ссылки

-   [observable](https://docs.solidjs.com/references/api-reference/reactive-utilities/observable)
