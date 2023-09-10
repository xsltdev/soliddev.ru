---
description: Используется для пакетной обработки async-обновлений в транзакции, откладывая фиксацию до завершения всех async-процессов
---

# useTransition

```ts
function useTransition(): [
    pending: () => boolean,
    startTransition: (fn: () => void) => Promise<void>
];
```

Используется для пакетной обработки async-обновлений в транзакции, откладывая фиксацию до завершения всех async-процессов. Эта функция привязана к `Suspense` и отслеживает только ресурсы, прочитанные в границах `Suspense`.

```ts
const [isPending, start] = useTransition();

// check if transitioning
isPending();

// wrap in transition
start(() => setSignal(newValue), () => /* transition is done */)
```

## Ссылки

-   [useTransition](https://docs.solidjs.com/references/api-reference/reactive-utilities/useTransition)
