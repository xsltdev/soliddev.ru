---
description: Помощник, облегчающий взаимодействие с внешними производителями, такими как наблюдаемые значения RxJS или Svelte Stores
---

# from

```ts
function from<T>(
    producer:
        | ((setter: (v: T) => T) => () => void)
        | {
              subscribe: (
                  fn: (v: T) => void
              ) =>
                  | (() => void)
                  | { unsubscribe: () => void };
          }
): () => T | undefined;
```

Помощник, облегчающий взаимодействие с внешними производителями, такими как наблюдаемые значения RxJS или Svelte Stores. По сути, он превращает любой подписываемый объект (объект с методом `subscribe`) в Signal и управляет подпиской и отменой.

```ts
const signal = from(obsv$);
```

Также может быть использована пользовательская функция-производитель, которой передается функция-сеттер, возвращающая функцию-отписку:

```ts
const clock = from((set) => {
    const interval = setInterval(() => {
        set((v) => v + 1);
    }, 1000);

    return () => clearInterval(interval);
});
```

## Аргументы

| Имя        | Тип                                                                                                                                       | Описание                                    |
| :--------- | :---------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------ |
| `producer` | <code>((setter: (v: T) => T) => () => void) \| { subscribe: (fn: (v: T) => void) => (() => void) \| { unsubscribe: () => void }; }</code> | Функция `producer` или подписываемый объект |

## Ссылки

-   [from](https://docs.solidjs.com/references/api-reference/reactive-utilities/from)
