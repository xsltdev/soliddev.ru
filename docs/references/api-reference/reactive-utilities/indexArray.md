---
description: Аналогичен mapArray, за исключением сопоставления по индексу. Элемент является сигналом, а индекс - константой
---

# indexArry

```ts
function indexArray<T, U>(
    list: () => readonly T[],
    mapFn: (v: () => T, i: number) => U
): () => U[];
```

Аналогичен `mapArray`, за исключением сопоставления по индексу. Элемент является сигналом, а индекс - константой.

Основной помощник для потока управления `<Index>`.

```ts
const mapped = indexArray(source, (model) => {
  return {
    get id() {
      return model().id
    }
    get firstInitial() {
      return model().firstName[0];
    },
    get fullName() {
      return `${model().firstName} ${model().lastName}`;
    },
  }
});
```

## Аргументы

| Имя     | Тип                           | Описание                |
| :------ | :---------------------------- | :---------------------- |
| `list`  | `() => readonly T[]`          | Список для отображения. |
| `mapFn` | `(v: () => T, i: число) => U` | Функция отображения.    |

## Ссылки

-   [indexArry](https://docs.solidjs.com/references/api-reference/reactive-utilities/indexArray)
