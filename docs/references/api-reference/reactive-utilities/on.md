---
description: on предназначен для передачи в вычисление, чтобы сделать его зависимости явными
---

# on

```ts
function on<T extends Array<() => any> | (() => any), U>(
    deps: T,
    fn: (input: T, prevInput: T, prevValue?: U) => U,
    options: { defer?: boolean } = {}
): (prevValue?: U) => U | undefined;
```

`on` предназначен для передачи в вычисление, чтобы сделать его зависимости явными. Если передается массив зависимостей, то `input` и `prevInput` являются массивами.

```ts
createEffect(on(a, (v) => console.log(v, b())));

// is equivalent to:
createEffect(() => {
    const v = a();
    untrack(() => console.log(v, b()));
});
```

Вы также можете не запускать вычисления сразу, а сделать так, чтобы они выполнялись только при изменении, установив опцию `defer` в `true`.

```ts
// doesn't run immediately
createEffect(on(a, (v) => console.log(v), { defer: true }));

setA('new'); // now it runs
```

## Использование `on` с хранилищами

!!!note ""

    Обратите внимание, что для магазинов и mutable добавление или удаление свойства из родительского объекта вызывает эффект. См. раздел [createMutable](../stores/store-utilities.md).

```ts
const [state, setState] = createStore({ a: 1, b: 2 });

// this will not work
createEffect(on(state.a, (v) => console.log(v)));

setState({ a: 3 }); // logs nothing

// instead, use an arrow function
createEffect(
    on(
        () => state.a,
        (v) => console.log(v)
    )
);

setState({ a: 4 }); // logs 4
```

## Аргументы и опции

| Аргумент  | Тип                                            | Описание                                                     |
| :-------- | :--------------------------------------------- | :----------------------------------------------------------- |
| `deps`    | `T`                                            | Зависимости, за которыми нужно следить.                      |
| `fn`      | `(input: T, prevInput: T, prevValue?: U) => U` | Функция, которую нужно запускать при изменении зависимостей. |
| `options` | `{ defer?: boolean }`                          | Параметры для настройки эффекта.                             |

## Ссылки

-   [on](https://docs.solidjs.com/references/api-reference/reactive-utilities/on)
