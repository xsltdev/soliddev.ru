---
description: Реактивный помощник карты, который кэширует каждый элемент по ссылке для уменьшения ненужного отображения при обновлении
---

# mapArray

```ts
function mapArray<T, U>(
    list: () => readonly T[],
    mapFn: (v: T, i: () => number) => U
): () => U[];
```

Реактивный помощник карты, который кэширует каждый элемент по ссылке для уменьшения ненужного отображения при обновлении. Функция отображения выполняется только один раз для каждого значения, а затем перемещается или удаляется по мере необходимости. Аргумент `index` является сигналом. Сама функция `map` не отслеживается.

Базовый помощник для потока управления `<For>`.

```ts
const mapped = mapArray(source, (model) => {
    const [name, setName] = createSignal(model.name);
    const [description, setDescription] = createSignal(
        model.description
    );

    return {
        id: model.id,
        get name() {
            return name();
        },
        get description() {
            return description();
        },
        setName,
        setDescription,
    };
});
```

## Аргументы

| Имя     | Тип                           | Описание                         |
| :------ | :---------------------------- | :------------------------------- |
| `list`  | `() => readonly T[]`          | Исходный массив для отображения. |
| `mapFn` | `(v: T, i: () => число) => U` | Функция отображения.             |

## Ссылки

-   [mapArray](https://docs.solidjs.com/references/api-reference/reactive-utilities/mapArray)
