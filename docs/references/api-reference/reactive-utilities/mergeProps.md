---
description: Используется для установки свойств по умолчанию для компонентов в случае, если вызывающая сторона их не предоставила
---

# mergeProps

```ts
function mergeProps(...sources: any): any;
```

Метод **merge** реактивного объекта. Используется для установки свойств по умолчанию для компонентов в случае, если вызывающая сторона их не предоставила. Или клонировать объект свойств, включая реактивные свойства.

Этот метод работает за счет использования прокси и разрешения свойств в обратном порядке. Это позволяет динамически отслеживать свойства, которые не присутствуют при первом объединении объекта props.

```ts
// default props
props = mergeProps({ name: 'Smith' }, props);

// clone props
newProps = mergeProps(props);

// merge props
props = mergeProps(props, otherProps);
```

## Ссылки

-   [mergeProps](https://docs.solidjs.com/references/api-reference/reactive-utilities/mergeProps)
