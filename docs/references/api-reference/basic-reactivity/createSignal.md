---
description: Сигналы отслеживают одно значение (которым может быть любой объект JavaScript), изменяющееся с течением времени
---

# createSignal

**Сигналы** - это самый простой реактивный примитив. Они отслеживают одно значение (которым может быть любой объект JavaScript), изменяющееся с течением времени.

```ts
function createSignal<T>(
    initialValue: T,
    options?: {
        equals?: false | ((prev: T, next: T) => boolean);
        name?: string;
        internal?: boolean;
    }
): [get: () => T, set: (v: T) => T];

// available types for return value of createSignal:
type Signal<T> = [get: Accessor<T>, set: Setter<T>];
type Accessor<T> = () => T;
type Setter<T> = (v: T | ((prev?: T) => T)) => T;
```

Значение сигнала в начале равно переданному первому аргументу `initialValue` (или неопределено, если аргументов нет). Функция `createSignal` возвращает пару функций в виде двухэлементного массива: геттер (или аксессор) и сеттер. При обычном использовании этот массив можно деструктурировать в именованный сигнал следующим образом:

```ts
const [count, setCount] = createSignal(0);
const [ready, setReady] = createSignal(false);
```

Вызов геттера (например, `count()` или `ready()`) возвращает текущее значение Сигнала.

Важным для автоматического отслеживания зависимостей является то, что вызов геттера в области отслеживания приводит к тому, что вызывающая функция становится зависимой от этого Сигнала, поэтому при обновлении Сигнала эта функция будет запущена заново.

Вызов сеттера (например, `setCount(nextCount)` или `setReady(nextReady)`) устанавливает значение Сигнала и обновляет его (вызывая повторное выполнение зависимых функций), если значение действительно изменилось (подробнее см. ниже). В качестве единственного аргумента сеттер принимает либо новое значение сигнала, либо функцию, сопоставляющую последнее значение сигнала с новым значением. Обновленное значение также возвращается сеттером. В качестве примера:

```ts
// read signal's current value, and
// depend on signal if in a tracking scope
// (but nonreactive outside of a tracking scope):
const currentCount = count();

// or wrap any computation with a function,
// and this function can be used in a tracking scope:
const doubledCount = () => 2 * count();

// or build a tracking scope and depend on signal:
const countDisplay = <div>{count()}</div>;

// write signal by providing a value:
setReady(true);

// write signal by providing a function setter:
const newCount = setCount((prev) => prev + 1);
```

!!!note ""

    Если вы хотите сохранить функцию в Signal, вы должны использовать форму функции:

    ```ts
    setValue(() => myFunction);
    ```

    Однако функции не рассматриваются специально как аргумент `initialValue` в `createSignal`, поэтому можно передавать начальное значение функции как есть:

    ```ts
    const [func, setFunc] = createSignal(myFunction);
    ```

## Параметры

| Наименование | Тип                                                   | По умолчанию | Описание                                                                                                                                                                                                                                                                                 |
| ------------ | ----------------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `equals`     | <code>false \| ((prev: T, next: T) => boolean)</code> | `===`        | Функция, определяющая, изменилось ли значение Сигнала. Если функция возвращает true, то значение Сигнала не обновляется и зависимые элементы не запускаются повторно. Если функция возвращает `false`, то значение Сигнала будет обновлено и зависимые элементы будут запущены повторно. |
| `name`       | `string`                                              |              | Имя для сигнала. Это полезно для отладки.                                                                                                                                                                                                                                                |
| `internal`   | `boolean`                                             | `false`      | Если `true`, то Signal не будет доступен в devtools.                                                                                                                                                                                                                                     |

### `equals`

Опция `equals` может быть использована для настройки проверки равенства, используемой для определения того, изменилось ли значение Signal. По умолчанию используется строгая проверка равенства (`===`). Если необходимо использовать другую проверку равенства, то в качестве опции `equals` можно передать пользовательскую функцию. Пользовательская функция будет вызвана с предыдущим и следующим значениями Signal в качестве аргументов. Если функция вернет `true`, то значение Signal не будет обновлено и зависимые функции не будут перезапускаться. Если функция вернет `false`, то значение Сигнала будет обновлено, а зависимые элементы будут повторно запущены.

```ts
const [count, setCount] = createSignal(0, {
    equals: (prev, next) => prev === next,
});
```

Вот несколько примеров использования этого варианта:

```ts
// use { equals: false } to allow modifying object in-place;
// normally this wouldn't be seen as an update because the
// object has the same identity before and after change
const [object, setObject] = createSignal(
    { count: 0 },
    { equals: false }
);
setObject((current) => {
    current.count += 1;
    current.updated = new Date();
    return current;
});

// use { equals: false } signal as trigger without value:
const [depend, rerun] = createSignal(undefined, {
    equals: false,
});
// now calling depend() in a tracking scope
// makes that scope rerun whenever rerun() gets called

// define equality based on string length:
const [myString, setMyString] = createSignal('string', {
    equals: (oldVal, newVal) =>
        newVal.length === oldVal.length,
});

setMyString('string'); // considered equal to the last value and won't cause updates
setMyString('stranger'); // considered different and will cause updates
```

### `name`

Опция `name` может быть использована для присвоения сигналу имени. Это полезно для отладки. Имя будет отображаться в devtools.

```ts
const [count, setCount] = createSignal(0, {
    name: 'count',
});
```

### `internal`

Опция `internal` может быть использована для скрытия сигнала от devtools. Это полезно для сигналов, которые используются внутри компонента и не должны быть доступны пользователю.

```ts
const [count, setCount] = createSignal(0, {
    internal: true,
});
```

## Ссылки

-   [createSignal](https://docs.solidjs.com/references/api-reference/basic-reactivity/createSignal)
