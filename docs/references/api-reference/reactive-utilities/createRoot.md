---
description: Создает новую не отслеживаемую область видимости владельца, которая не подвергается автоудалению
---

# createRoot

```ts
function createRoot<T>(fn: (dispose: () => void) => T): T;
```

Создает новую не отслеживаемую область видимости владельца, которая не подвергается автоудалению. Это полезно для вложенных реактивных областей, которые вы не хотите освобождать при повторном вычислении родительской области.

Весь код Solid должен быть обернут в одну из таких областей верхнего уровня, поскольку они гарантируют освобождение всей памяти/вычислений. Обычно об этом не нужно беспокоиться, поскольку createRoot встроен во все функции входа в рендеринг.

## Ссылки

-   [createRoot](https://docs.solidjs.com/references/api-reference/reactive-utilities/createRoot)
