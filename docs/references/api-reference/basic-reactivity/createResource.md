---
description: При этом создается сигнал, возвращающий результат выполнения асинхронного запроса
---

# createResource

При этом создается сигнал, возвращающий результат выполнения асинхронного запроса.

```ts
type ResourceReturn<T> = [
    {
        (): T | undefined;
        state:
            | 'unresolved'
            | 'pending'
            | 'ready'
            | 'refreshing'
            | 'errored';
        loading: boolean;
        error: any;
        latest: T | undefined;
    },
    {
        mutate: (v: T | undefined) => T | undefined;
        refetch: (info: unknown) => Promise<T> | T;
    }
];

export type ResourceOptions<T, S = unknown> = {
    initialValue?: T;
    name?: string;
    deferStream?: boolean;
    ssrLoadFrom?: 'initial' | 'server';
    storage?: (
        init: T | undefined
    ) => [Accessor<T | undefined>, Setter<T | undefined>];
    onHydrated?: (
        k: S | undefined,
        info: { value: T | undefined }
    ) => void;
};

function createResource<T, U = true>(
    fetcher: (
        k: U,
        info: {
            value: T | undefined;
            refetching: boolean | unknown;
        }
    ) => T | Promise<T>,
    options?: ResourceOptions<T, U>
): ResourceReturn<T>;

function createResource<T, U>(
    source: U | false | null | (() => U | false | null),
    fetcher: (
        k: U,
        info: {
            value: T | undefined;
            refetching: boolean | unknown;
        }
    ) => T | Promise<T>,
    options?: ResourceOptions<T, U>
): ResourceReturn<T>;
```

Функция `createResource` принимает асинхронную функцию fetcher и возвращает сигнал, который по завершении работы fetcher обновляется результирующими данными.

Существует два варианта использования `createResource`: можно передать функцию fetcher в качестве единственного аргумента, а можно дополнительно передать в качестве первого аргумента исходный сигнал. Сигнал источника будет перезапускать fetcher при каждом его изменении, а его значение будет передаваться в fetcher.

```ts
const [data, { mutate, refetch }] =
    createResource(fetchData);
```

---

```ts
const [data, { mutate, refetch }] = createResource(
    source,
    fetchData
);
```

В этих фрагментах считывателем является функция `fetchData`, а `data()` остается неопределенной до тех пор, пока `fetchData` не завершит разрешение. В первом случае `fetchData` будет вызвана немедленно. Во втором случае `fetchData` будет вызвана, как только `sourceSignal` примет любое значение, отличное от false, null или undefined. Она будет вызываться каждый раз, когда значение `sourceSignal` изменится, и это значение всегда будет передаваться в `fetchData` в качестве первого аргумента.

Вы можете вызвать `mutate` для прямого обновления сигнала `data` (это работает как любой другой установщик сигнала). Также можно вызвать функцию refetch для повторного запуска фетчера напрямую и передать дополнительный аргумент для предоставления дополнительной информации фетчеру, например `refetch(info)`.

`data` работает как обычный геттер сигнала: используйте `data()` для чтения последнего возвращенного значения `fetchData`. Но у него есть и дополнительные реактивные свойства: `data.loading` сообщает, был ли вызван фетчер, но не вернулся, а `data.error` сообщает, был ли запрос ошибочным; если да, то в нем содержится ошибка, выброшенная фетчером. (Примечание: если вы ожидаете ошибок, то, возможно, захотите обернуть `createResource` в [ErrorBoundary](../control-flow/ErrorBoundary.md)).

Начиная с **v1.4.0**, `data.latest` возвращает последнее возвращенное значение и не вызывает [Suspense](../control-flow/Suspense.md) и переходов; если значение еще не было возвращено, `data.latest` действует так же, как `data()`. Это может быть полезно, если необходимо показать устаревшие данные, пока загружаются новые.

`loading`, `error` и `latest` являются реактивными геттерами и могут быть отслежены.

## Фетчер

Функция `fetcher` - это асинхронная функция, которую вы предоставляете `createResource` для получения данных. Ей передаются два аргумента: значение исходного сигнала (если оно задано) и объект info с двумя свойствами: `value` и `refetching`. Свойство `value` сообщает о ранее полученном значении. Свойство `refetching` равно true, если `fetcher` был вызван с помощью функции refetch, и false в противном случае. Если функция `refetch` была вызвана с аргументом (`refetch(info)`), то свойство refetching устанавливается на этот аргумент.

```ts
async function fetchData(source, { value, refetching }) {
    // Fetch the data and return a value.
    //`source` tells you the current value of the source signal;
    //`value` tells you the last returned value of the fetcher;
    //`refetching` is true when the fetcher is triggered by calling `refetch()`,
    // or equal to the optional data passed: `refetch(info)`
}

const [data, { mutate, refetch }] = createResource(
    getQuery,
    fetchData
);

// read value
data();

// check if loading
data.loading;

// check if errored
data.error;

// directly set value without creating promise
mutate(optimisticValue);

// refetch the last request explicitly
refetch();
```

Также можно передать несколько сигналов в качестве источника, построив производный сигнал:

```ts
const [dataSignal, setDataSignal] = createSignal(1);
const [moreData, setMoreData] = createSignal('string_data');
const [data] = createResource(
    () => [dataSignal(), moreData()] as const,
    ([dataVal, moreDataVal]) => {
        // Reruns when either signal updates
    }
);
```

Обратите внимание на `as const`. Это необходимо для того, чтобы typescript правильно определил типы аргументов fetcher'а: `dataVal` и `moreDataVal`. В качестве альтернативы их можно явно типизировать при передаче в fetcher:

```ts
const [data] = createResource(
    () => [dataSignal(), moreData()],
    ([dataVal, moreDataVal]) =>
        fetcher(dataVal as number, moreDataVal as string)
);
```

## Версия 1.4.0 и более поздняя

### v1.4.0

Если вы используете `renderToStream`, то с помощью опции `deferStream` можно указать Solid на необходимость ожидания ресурса перед очисткой потока:

```ts
// fetches a user and streams content as soon as possible
const [user] = createResource(() => params.id, fetchUser);

// fetches a user but only streams content after this resource has loaded
const [user] = createResource(() => params.id, fetchUser, {
    deferStream: true,
});
```

### v1.5.0

1.  Мы добавили новое поле состояния, которое позволяет получить более подробную информацию о состоянии ресурса, чем `loading` и `error`. Теперь можно проверить, является ли ресурс `unresolved`, `pending`, `ready`, `refreshing` или `error`.

    | Состояние    | Значение разрешено | Загрузка | Есть ошибка |
    | ------------ | ------------------ | -------- | ----------- |
    | `unresolved` | No                 | No       | No          |
    | `pending`    | No                 | Yes      | No          |
    | `ready`      | Yes                | No       | No          |
    | `refreshing` | Yes                | Yes      | No          |
    | `error`      | No                 | No       | Yes         |

2.  При серверном рендеринге ресурсов, особенно при выборке при встраивании Solid в другие системы, которые выполняют выборку перед рендерингом, вы можете захотеть инициировать ресурс с этим предварительно полученным значением вместо повторной выборки и сериализации ресурса в его собственном состоянии. Для этого можно использовать новую опцию `ssrLoadFrom`. Вместо значения по умолчанию `server` можно передать `initial`, и ресурс будет использовать `initialValue`, как если бы это был результат первой выборки как для SSR, так и для гидратации.

    ```ts
    const [data, { mutate, refetch }] = createResource(
        () => params.id,
        fetchUser,
        {
            initialValue: preloadedData,
            ssrLoadFrom: 'initial',
        }
    );
    ```

3.  Ресурсы могут быть установлены с пользовательским хранилищем с той же сигнатурой, что и Сигнал, с помощью опции хранилища. Например, использование пользовательского хранилища сверки может быть выполнено следующим образом:

    ```ts
    function createDeepSignal<T>(value: T): Signal<T> {
        const [store, setStore] = createStore({
            value,
        });
        return [
            () => store.value,
            (v: T) => {
                const unwrapped = unwrap(store.value);
                typeof v === 'function' &&
                    (v = v(unwrapped));
                setStore('value', reconcile(v));
                return store.value;
            },
        ] as Signal<T>;
    }

    const [resource] = createResource(fetcher, {
        storage: createDeepSignal,
    });
    ```

Данная опция пока является экспериментальной и может быть изменена в будущем.

## Опции

Функция `createResource` принимает необязательный третий аргумент - объект options. Опциями являются:

| Имя            | Тип                                             | По умолчанию          | Описание                                                                                                                                                                  |
| -------------- | ----------------------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`         | `string`                                        | `undefined`           | Имя ресурса. Оно используется для отладки.                                                                                                                                |
| `deferStream`  | `boolean`                                       | `false`               | Если значение равно `true`, то Solid будет ждать разрешения ресурса перед очисткой потока.                                                                                |
| `initialValue` | `any`                                           | `undefined`           | Начальное значение ресурса.                                                                                                                                               |
| `onHydrated`   | `function`                                      | `undefined`           | Обратный вызов, который вызывается при гидратации ресурса.                                                                                                                |
| `ssrLoadFrom`  | <code>"server"</code> \| <code>"initial"</code> | <code>"server"</code> | Источник начального значения для SSR. Если установлено значение `"initial"`, то ресурс будет использовать опцию `initialValue` вместо значения, возвращаемого fetcher'ом. |
| `storage`      | `function`                                      | `createSignal`        | Функция, возвращающая сигнал. Это может быть использовано для создания пользовательского хранилища для ресурса. Это пока экспериментальная версия                         |

## Ссылки

-   [createResource](https://docs.solidjs.com/references/api-reference/basic-reactivity/createResource)
