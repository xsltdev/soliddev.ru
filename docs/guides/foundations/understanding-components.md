---
description: Компоненты представляют собой относительно небольшие фрагменты кода, которые можно повторно использовать и комбинировать для создания более крупных и сложных приложений
---

# Понимание компонентов

## Что такое компоненты?

**Компоненты** - это строительные блоки большинства фронтенд-приложений. Они представляют собой относительно небольшие фрагменты кода, которые можно повторно использовать и комбинировать для создания более крупных и сложных приложений.

В JSX-фреймворках, таких как Solid, Preact, React и им подобных, компоненты определяются с помощью функции, которая возвращает JSX-элемент. Вот пример простого компонента:

```js
function MyComponent() {
    return <div>Hello World</div>;
}
```

Затем этот компонент может быть использован в других компонентах или в основном компоненте приложения. Например, мы можем использовать только что созданный компонент в главном компоненте приложения следующим образом:

```js
function App() {
    return (
        <div>
            <MyComponent />
        </div>
    );
}
```

## Что такое дерево компонентов?

Дерево компонентов - это древовидная структура, представляющая компоненты вашего приложения. Корнем дерева является главный компонент приложения. Каждый компонент в дереве может иметь ноль или более дочерних компонентов. Дочерними компонентами компонента являются компоненты, которые отображаются внутри него.

Например, дерево компонентов для приложения, которое мы только что определили, будет выглядеть следующим образом:

```
App
└── MyComponent
```

Деревья компонентов могут быть очень сложными и большими. Например, дерево компонентов для большого приложения может выглядеть следующим образом:

```
App
├── Header
├── Sidebar
├── Content
│   ├── Post
│   │   ├── PostHeader
│   │   ├── PostContent
│   │   └── PostFooter
│   ├── Post
│   │   ├── PostHeader
│   │   ├── PostContent
│   │   └── PostFooter
|   └── Post
│       ├── ...
└── Footer
```

Из приведенного выше рисунка видно, что компоненты могут иметь дочерние компоненты, которые также являются компонентами. Именно это делает компоненты настолько мощными. Они позволяют нам разбивать наши приложения на более мелкие и управляемые части.

## Почему компоненты отличаются от компонентов в Solid?

В Solid компоненты определяются с помощью функции, которая возвращает JSX-элемент. Однако жизненный цикл функции заканчивается после ее первого запуска. Это означает, что функция не будет повторно запущена при изменении состояния приложения. Это отличается от других фреймворков, таких как React, где функции определяются и запускаются каждый раз при изменении состояния приложения.

Этот механизм, используемый Solid, можно назвать функциями настройки, в отличие от функций рендеринга других фреймворков. Это связано с тем, что функция запускается только один раз и используется для настройки всего, что должно отслеживаться реактивной системой. Это очень важное различие, которое необходимо учитывать при изучении Solid.

Вот пример того, что все это значит:

```js
function MyComponent() {
    const [count, setCount] = createSignal(0);

    // This will only be run once and will not be
    // re-run when the application state changes
    console.log(count());

    return (
        <div>
            <p>Count: {count()}</p>
            <button onClick={() => setCount(count() + 1)}>
                Increment
            </button>
        </div>
    );
}
```

Как видно из приведенного выше кода, в нем присутствует оператор `console.log(count())`. Оно будет выполнено только один раз и не будет повторно выполняться при изменении состояния приложения. Это связано с тем, что функция выполняется только один раз и используется для настройки всего, что необходимо отслеживать реактивной системе.

Для того чтобы оператор `console.log(count())` выполнялся каждый раз при изменении состояния count, необходимо использовать примитив `createEffect`. Этот примитив будет запускать функцию при каждом изменении состояния count. Вот пример того, как это можно сделать:

```js
function MyComponent() {
    const [count, setCount] = createSignal(0);

    // This will be run every time the count state changes
    createEffect(() => {
        console.log(count());
    });

    return (
        <div>
            <p>Count: {count()}</p>
            <button onClick={() => setCount(count() + 1)}>
                Increment
            </button>
        </div>
    );
}
```

Именно такие мелкие различия, как правило, сбивают людей с толку, когда они только начинают использовать Solid. Однако, как только вы привыкнете к нему, это окажется очень мощным и гибким механизмом.

Вот еще один пример того, как компоненты Solid отличаются от компонентов React:

```js
function MyComponent() {
    const [count, setCount] = createSignal(0);

    if (count() > 5) {
        return <div>Count limit reached</div>;
    }

    return (
        <div>
            <p>Count: {count()}</p>
            <button onClick={() => setCount(count() + 1)}>
                Increment
            </button>
        </div>
    );
}
```

В приведенном выше примере компонент не вернет div `Count limit reached`, если счетчик превышает 5. Это происходит потому, что сам компонент не является реактивным и используется только один раз, поэтому он не знает, когда состояние `count` станет больше 5. Чтобы сделать `Count limit reached` видимым, необходимо добавить его в реактивную область видимости, например, в возвращаемый JSX, и использовать логику, заложенную в нем. Вот пример того, как это можно сделать:

```js
function MyComponent() {
    const [count, setCount] = createSignal(0);

    return (
        <div>
            {count() > 5 ? (
                <div>Count limit reached</div>
            ) : (
                <>
                    <p>Count: {count()}</p>
                    <button
                        onClick={() =>
                            setCount(count() + 1)
                        }
                    >
                        Increment
                    </button>
                </>
            )}
        </div>
    );
}
```

Приведенный выше код будет работать, как и ожидалось, и отображать div `Count limit reached`, когда счетчик будет больше 5. Но именно так должен выглядеть ваш код, если вы пишете на React, а не на Solid. В Solid следует использовать встроенные компоненты control-flow для достижения того же результата, но с лучшей реактивностью и оптимизацией. Вот пример того, как это можно сделать:

```js
function MyComponent() {
    const [count, setCount] = createSignal(0);

    return (
        <div>
            <Show
                when={count() > 5}
                fallback={
                    <>
                        <p>Count: {count()}</p>
                        <button
                            onClick={() =>
                                setCount(count() + 1)
                            }
                        >
                            Increment
                        </button>
                    </>
                }
            >
                <div>Count limit reached</div>
            </Show>
        </div>
    );
}
```

## Что такое свойства компонента?

Свойства компонента - это свойства, которые передаются компоненту. Они используются для передачи данных компоненту. Например, мы можем передать свойство name компоненту `MyComponent`, который мы определили ранее, следующим образом:

```js
function App() {
    return (
        <div>
            <MyComponent name="John Doe" />
        </div>
    );
}
```

Компонент `MyComponent` может получить доступ к свойству `name` следующим образом:

```js
function MyComponent(props) {
    return <div>Hello {props.name}</div>;
}
```

#### Почему деструктуризация свойств плоха в Solid?

Разработчики React, возможно, привыкли к деструктуризации свойств в аргументах функций, например, так:

```js
function MyComponent({ name }) {
    return <div>Hello {name}</div>;
}
```

Однако в Solid это очень плохая практика, поскольку можно предположить, что вы обращаетесь к реактивному значению свойства, но это не так, поскольку само значение не будет реактивным, а будет служить ссылкой на реактивное значение при начальной настройке компонента. Это означает, что если значение реквизита изменится вне компонента, то оно не будет обновлено внутри компонента.

Чтобы избежать этого, лучше всего не деструктурировать свойства в аргументах функции, а обращаться к ним с помощью объекта `props` внутри реактивной области видимости.

Приведем несколько примеров, поясняющих этот момент:

1.  Доступ к значениям свойств вне реактивной области видимости:

    ```js
    function MyComponent(props) {
        // This is a bad practice and will not update
        // within the component when the prop value changes
        const name = props.name;

        // This is a good practice and will update
        // within the component when the prop value changes
        // createSignal is a Solid primitive. More on this in the next page
        const [name, setName] = createSignal(props.name);

        return <div>Hello {name}</div>;
    }
    ```

2.  Доступ к значениям свойств за пределами реактивной области с помощью деструктуризации:

    ```js
    // This is a bad practice and will not update
    // within the component when the prop value changes
    function MyComponent(props) {
        const { name } = props;
        return <div>Hello {name}</div>;
    }

    // This is a good practice and will update
    // within the component when the prop value changes
    function MyComponent(props) {
        return <div>Hello {props.name}</div>;
    }
    ```

3.  Деструктуризация свойств в аргументах функции:

    ```js
    // This is a bad practice and will not update within
    // the component when the prop value changes
    function MyComponent({ name }) {
        return <div>Hello {name}</div>;
    }

    // This is a good practice and will update within
    // the component when the prop value changes
    function MyComponent(props) {
        return <div>Hello {props.name}</div>;
    }
    ```

## Ссылки

-   [Understanding Components](https://docs.solidjs.com/guides/foundations/understanding-components)
