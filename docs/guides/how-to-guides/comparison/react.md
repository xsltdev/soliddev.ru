---
description: React оказал большое влияние на Solid. Его однонаправленный поток и явное разделение чтения и записи в API хуков легли в основу API Solid
---

# Сравнение с React

React оказал большое влияние на Solid. Его однонаправленный поток и явное разделение чтения и записи в API хуков легли в основу API Solid. В большей степени, чем задача быть просто "библиотекой рендеринга", а не фреймворком.

Solid имеет свое мнение о том, как следует подходить к управлению данными при разработке приложений, но не стремится ограничивать их выполнение.

Однако, несмотря на то, что Solid соответствует философии дизайна React, он работает принципиально иначе. Такое сходство API с другой моделью исполнения обычно отталкивает разработчиков React, которые ожидают, что определенные паттерны React можно будет перенести в Solid. Данное руководство призвано помочь указать на эти различия и способы их преодоления.

## Компоненты как функции рендеринга против функций настройки

### React

Если говорить упрощенно, то компоненты React работают как способ организации кода и обработки обновлений представления, вытекающих из изменений состояния или свойств. Это означает, что при каждом обновлении React будет перезапускать компонент с последним состоянием/свойствами, чтобы отразить его в представлении. Рассмотрим следующий пример:

```js
import { useState } from 'react';

const App = () => {
    const [count, setCount] = useState(0);

    return (
        <>
            <p>Count is: {count}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );
};
```

Каждый раз, когда мы нажимаем на кнопку, происходит обновление состояния, приводящее к повторному запуску нашего компонента.

![Скриншот браузера файлов в Stackblitz, отображающий все файлы в шаблоне](react-simple-state-update-light.svg#only-light)
![Снимок экрана браузера файлов в Stackblitz, отображающего все файлы в шаблоне](react-simple-state-update-dark.svg#only-dark)

### Solid

Мы можем создать очень похожий код с помощью Solid, но принцип его работы под капотом будет отличаться. Компонент снова будет полезен как способ организации кода, но, в отличие от React, он не будет перезапускаться каждый раз, когда происходит обновление состояния или свойств.

Вместо этого компонент будет настраивать все, что необходимо отслеживать реактивной системе Solid, и уходить с дороги, когда код будет выполняться. Это означает, что **функция компонента будет запущена только один раз**, поскольку она не будет обрабатывать обновления состояния и свойств. Вот тот же пример, написанный на Solid:

```js
import { createSignal } from 'solid-js';

const App = () => {
    const [count, setCount] = createSignal(0);

    return (
        <>
            <p>Count is: {count()}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );
};
```

Когда мы нажмем на кнопку, реактивная система Solid будет выполнять только гранулярные обновления.

![Снимок экрана браузера файлов в Stackblitz, отображающего все файлы в шаблоне](solid-simple-state-update-light.svg#only-light)
![Снимок экрана браузера файлов в Stackblitz, отображающий все файлы в шаблоне](solid-simple-state-update-dark.svg#only-dark)

Это то, что мы подразумеваем в заголовке под _функциями рендеринга_ и _функциями настройки_. В React рендеринг и его обновления привязаны к компоненту. В Solid же компоненты существуют только в вашем коде и запускаются один раз для настройки всех элементов, необходимых для работы реактивной системы. Компоненты в Solid можно рассматривать как _исчезающие компоненты_, они исчезают после выполнения кода.

Это приводит к различиям в работе кода между двумя фреймворками, несмотря на их схожесть при написании кода. Например, мы можем вынести состояние за пределы компонента, и код Solid будет работать.

```js
import { createSignal } from 'solid-js';

const [count, setCount] = createSignal(0);

const App = () => {
    return (
        <>
            <p>Count is: {count()}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );
};
```

Опять же, это возможно благодаря тому, что реактивная система Solid живет вне компонентов. Если вы хотите почитать на эту тему, ознакомьтесь со статьей Райана Карниато (создателя Solid) [Components are Pure Overhead](https://dev.to/this-is-learning/components-are-pure-overhead-hpm)

## Ранние возвраты и использование `<Show>`

Поскольку компоненты в Solid выполняются только один раз, следующий код React **не** переносится в Solid

### В React

```js
import { useState } from 'react';

const App = () => {
    const [count, setCount] = useState(0);

    if (count > 5) {
        return <p>Count is too high!</p>;
    }

    return (
        <>
            <p>Count is: {count}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );
};
```

В приведенном примере, поскольку React выполняет рендеринг при каждом изменении состояния, наше условие оценивается каждый раз, когда мы нажимаем на кнопку. Как только `count` станет больше 5, наше условие будет истинным, и React отобразит наш элемент `<p`.

### В Solid

Мы можем написать аналогичный код, но модель выполнения снова будет другой

```js
import { createSignal } from 'solid-js';

const App = () => {
    const [count, setCount] = createSignal(0);

    if (count() > 5) {
        return <p>Count is too high!</p>;
    }

    return (
        <>
            <p>Count is: {count()}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );
};
```

Здесь, поскольку мы инициализируем счетчик в 0, а компонент запускается только один раз, условие будет оценено, окажется ложным и никогда не будет повторно оценено. Вы можете нажать на кнопку и перейти через 5, и все равно увидите счетчик и кнопку.

Это связано с тем, как работает гранулярная реактивность Solid. Проще говоря, в React реализован opt-out рендеринг, то есть React будет рендерить все заново, если не сказать иначе (вспомните `memo()`, `useMemo()`, `useCallback()`).

С другой стороны, Solid - это opt-in рендеринг, то есть он не будет перерисовывать ничего, что не отслеживается реактивной системой. Solid рассматривает рендеринг элементов в DOM как побочный эффект изменения состояния, поэтому способ "исправить" наш код заключается в проверке состояния внутри JSX, чтобы реактивная система отслеживала его и выводила DOM в результате изменения состояния. Solid предоставляет встроенный компонент, который поможет нам в этом: `<Show>`.

```js
import { createSignal, Show } from 'solid-js';

const App = () => {
    const [count, setCount] = createSignal(0);

    const fallback = (
        <>
            <p>Count is: {count()}</p>
            <button
                onClick={() =>
                    setCount((prevCount) => prevCount + 1)
                }
            >
                Increase count by 1
            </button>
        </>
    );

    return (
        <Show when={count() > 5} fallback={fallback}>
            <p>Count is too high!</p>
        </Show>
    );
};
```

## Перебор списков и использование `<For>`

Обычно итерация по спискам в React выполняется следующим образом

```js
import { useState } from 'react';

const App = () => {
    const [todos] = useState([
        { id: '1', name: 'Learn Solid' },
        { id: '2', name: 'Learn Solid Start' },
    ]);

    return (
        <>
            <h1>TO-DO</h1>
            <ul>
                {todos.map((todo) => (
                    <li key={todo.id}>{todo.name}</li>
                ))}
            </ul>
        </>
    );
};
```

Хотя это будет работать и на Solid, идиоматическим способом вывода списков является использование компонента `<For>`

```js
import { createSignal, For } from 'solid-js';

const App = () => {
    const [todos] = createSignal([
        { id: '1', name: 'Learn Solid' },
        { id: '2', name: 'Learn Solid Start' },
    ]);

    return (
        <>
            <h1>TO-DO</h1>
            <ul>
                <For each={todos()}>
                    {(todo) => <li>{todo.name}</li>}
                </For>
            </ul>
        </>
    );
};
```

Одним из преимуществ компонента является то, что отображаемые элементы по умолчанию имеют ключи, поэтому мы можем забыть о том, чтобы делать это самостоятельно.

Ключи по умолчанию позволяют выполнять гранулярные обновления, поскольку Solid будет применять обновления только к определенным элементам списка, а не пересоздавать список при каждом обновлении.

## Деструктуризация свойств

Деструктуризация свойств - обычное дело для React. Если мы нажмем кнопку "Add Todo" в приведенном ниже коде, React перерисует компонент `<App>` и все его дочерние элементы, отобразив новое значение в DOM.

```js
// This will update the DOM with the new todo
import { useState } from 'react';

const Todos = ({ todos }) => {
    return (
        <ul>
            {todos.map((todo) => (
                <li key={todo.id}>{todo.name}</li>
            ))}
        </ul>
    );
};

const App = () => {
    const [todos, setTodos] = useState([
        { id: '1', name: 'Learn Solid' },
        { id: '2', name: 'Learn Solid Start' },
    ]);
    return (
        <>
            <h1>TO-DO</h1>
            <Todos todos={todos} />
            <button
                onClick={() =>
                    setTodos((prev) => [
                        ...prev,
                        { id: '3', name: 'Learn Qwik' },
                    ])
                }
            >
                Add Todo
            </button>
        </>
    );
};
```

Аналогичный код для Solid и деструктуризация свойств компонента `<Todos>` не приведет к обновлению DOM.

```js
// This won't update the DOM with the new todo
import { createSignal, For } from 'solid-js';

const Todos = ({ todos }) => {
    return (
        <ul>
            <For each={todos}>
                {(todo) => <li>{todo.name}</li>}
            </For>
        </ul>
    );
};

const App = () => {
    const [todos, setTodos] = createSignal([
        { id: '1', name: 'Learn Solid' },
        { id: '2', name: 'Learn Solid Start' },
    ]);

    return (
        <>
            <h1>TO-DO</h1>
            <Todos todos={todos()} />
            <button
                onClick={() =>
                    setTodos((prev) => [
                        ...prev,
                        { id: '3', name: 'Learn Qwik' },
                    ])
                }
            >
                Add Todo
            </button>
        </>
    );
};
```

Причина этого, опять же, в том, что в Solid реализована реактивная модель "opt-in". Это означает, что все, что не находится в отслеживаемой области видимости (примитивы Solid и JSX), не вызовет обновления.

В нашем коде, приведенном выше, деструктурируя свойства, мы получаем к ним доступ в неотслеживаемой области видимости. Чтобы "исправить" это, необходимо обратиться к свойствам в отслеживаемой области видимости, в данном случае внутри JSX

```js
// This will update the DOM with the new todo
import { createSignal, For } from 'solid-js';

const Todos = (props) => {
    return (
        <ul>
            <For each={props.todos}>
                {(todo) => <li>{todo.name}</li>}
            </For>
        </ul>
    );
};

const App = () => {
    const [todos, setTodos] = createSignal([
        { id: '1', name: 'Learn Solid' },
        { id: '2', name: 'Learn Solid Start' },
    ]);

    return (
        <>
            <h1>TO-DO</h1>
            <Todos todos={todos()} />
            <button
                onClick={() =>
                    setTodos((prev) => [
                        ...prev,
                        { id: '3', name: 'Learn Qwik' },
                    ])
                }
            >
                Add Todo
            </button>
        </>
    );
};
```

## React vs. Solid

Приведем таблицу, различающую хуки в React и хуки в Solid.

| React (компоненты классов) | React (функциональные компоненты)         | Solid            |
| -------------------------- | ----------------------------------------- | ---------------- |
| `componentDidMount()`      | `useEffect(() => {}, [])`                 | `onMount()`      |
| `componentWillUnmount()`   | `useEffect(() => { return () => {}}, [])` | `onCleanup()`    |
| `this.state`               | `useState()`                              | `createSignal()` |
| `this.state`               | `useState()`                              | `createStore()`  |
| N/A                        | `useMemo()`                               | `createMemo()`   |
| `componentDidUpdate()`     | `useEffect(() => {}, [dependencies])`     | `createEffect()` |

## Ссылки

-   [Comparison with React](https://docs.solidjs.com/guides/how-to-guides/comparison/react)
