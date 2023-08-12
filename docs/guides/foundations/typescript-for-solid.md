# Typescript для Solid

В этом разделе мы рассмотрим Typescript и то, как он используется при создании Solid. Эта часть может показаться немного странной для Javascript-разработчиков, не знакомых с Typescript, поэтому мы постараемся максимально разложить ее по полочкам.

Solid спроектирован таким образом, чтобы его было легко использовать с TypeScript: использование стандартного JSX делает код в значительной степени понятным для TypeScript, а также предоставляет сложные встроенные типы для своего API. В этом руководстве рассмотрены некоторые полезные советы по работе с TypeScript и типизации кода Solid.

## Настройка TypeScript

Шаблоны [Solid starter templates](https://github.com/solidjs/templates/) предлагают хорошие отправные точки для [`tsconfig.json`](https://github.com/solidjs/templates/blob/master/ts/tsconfig.json).

!!!note ""

    Чтобы настроить уже существующий проект Solid Javascript на использование Typescript, следуйте этому [руководству](/guides/how-to-guides/get-ready-for-solid/installation-and-setup#setup-typescript-in-pre-existing-solidjs-javascript-projects).

Самое главное, для использования TypeScript с компилятором Solid JSX необходимо настроить TypeScript так, чтобы он оставлял JSX-конструкции в покое через [`"jsx": "preserve"`](https://www.typescriptlang.org/tsconfig#jsx), а также указать TypeScript, откуда берутся JSX-типы через [`"jsxImportSource": "solid-js"`](https://www.typescriptlang.org/tsconfig#jsxImportSource). Таким образом, минимальный `tsconfig.json` будет выглядеть следующим образом:

```json
{
    "compilerOptions": {
        "jsx": "preserve",
        "jsxImportSource": "solid-js"
    }
}
```

Если в вашей кодовой базе используется смесь JSX-типов (например, некоторые файлы - React, а другие - Solid), вы можете установить значение по умолчанию `jsxImportSource` в файле `tsconfig.json` для большинства вашего кода, а затем [переопределить опцию `jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource) в определенных файлах `.tsx` с помощью следующей прагмы:

```ts
/** @jsxImportSource solid-js */
```

или

```ts
/** @jsxImportSource react */
```

Для того чтобы воспользоваться последним, необходимо убедиться, что в проекте установлены `react` и его созависимости, а также что проект правильно настроен для использования JSX-файлов react.

## Типы API

Solid написан на TypeScript, поэтому все в нем типизировано. В документации [API](/references/api-reference) подробно описаны типы для всех вызовов API, а также приведено несколько полезных определений типов, чтобы было проще обращаться к понятиям Solid, когда нужно указать явные типы. Здесь мы рассмотрим результирующие типы при использовании нескольких основных примитивов.

### Сигналы

`createSignal<T>` параметризуется типом `T` объекта, хранящегося в сигнале. Например:

```ts
const [count, setCount] = createSignal<number>();
```

Приведенная выше `createSignal` имеет возвращаемый тип `Signal<number>`, соответствующий типу, который мы ей передали. Это кортеж из геттера и сеттера, каждый из которых имеет тип [generic](https://www.typescriptlang.org/docs/handbook/2/generics.html):

```ts
import type { Signal, Accessor, Setter } from 'solid-js';
type Signal<T> = [get: Accessor<T>, set: Setter<T>];
```

!!!note ""

    В TypeScript 3.8 добавлен новый синтаксис для импорта и экспорта только типов. В `import type` импортируются только декларации, которые будут использоваться для аннотаций и деклараций типов. Они будут полностью удалены после компиляции и не будут включены в выдаваемый JavaScript. Подробнее о них можно прочитать [здесь](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html)

В данном случае геттер сигнала `count` имеет тип `Accessor<number | undefined>`. `Accessor<T>` - это определение типа, предоставляемое Solid, в данном случае эквивалентное `() => число | undefined`. В данном примере `| undefined` добавлено потому, что мы не указали значение по умолчанию для `createSignal`, поэтому значение сигнала действительно начинается как `undefined`.

Установщик сигнала `setCount` имеет тип `Setter<number>`, что является более сложным определением типа, примерно соответствующим `(value?: number | ((prev?: number) => number)) => number`, представляющим две возможности для передаваемого аргумента: вы можете вызвать `setCount` либо с `number`, либо с функцией, принимающей предыдущее значение (если оно было) и возвращающей `number`. Заметим, что и параметр `number`, и параметр `number` для функции являются необязательными, поскольку начальное значение сигнала было `неопределенным`.

В действительности тип `Setter` сложнее, поскольку нам необходимо различать передачу функции-установщика и передачу функции в качестве значения, на которое мы хотим установить сигнал. Если при вызове `setCount(value)` возникает ошибка TypeScript "Argument ... is not assignable to parameter", то попробуйте обернуть аргумент сеттера как в `setCount(() => value)`, чтобы убедиться, что `value` не будет вызван.

**По умолчанию**.

Мы можем избежать необходимости явно указывать тип сигнала при вызове `createSignal` и избежать `| undefined` части типа, предоставив значение по умолчанию для `createSignal`:

```ts
const [count, setCount] = createSignal(0);
const [name, setName] = createSignal('');
```

В этом случае TypeScript считает, что типы сигналов - `number` и `string` соответственно. Таким образом, например, `count` получает тип `Accessor<number>`, а `name` - тип `Accessor<string>` (без `| undefined`).

### Контекст

Аналогично сигналам, функция [`createContext<T>`](https://www.solidjs.com/docs/latest/api#createcontext) параметризуется типом `T` значения контекста. Мы можем указать этот тип в явном виде:

```ts
type Data = { count: number; name: string };
const dataContext = createContext<Data>();
```

В данном случае `dataContext` имеет тип `Context<Data | undefined>`, в результате чего `useContext(dataContext)` будет иметь соответствующий возвращаемый тип `Data | undefined`. Причина `| undefined` заключается в том, что контекст может быть не указан в предках текущего компонента, и тогда `useContext` возвращает `undefined`.

Если вместо этого мы предоставляем значение по умолчанию для `createContext`, мы избегаем `| undefined` части типа, а также часто избегаем необходимости явного указания типа `createContext`:

```ts
const dataContext = createContext({ count: 0, name: '' });
```

В данном случае TypeScript считает, что `dataContext` имеет тип `Context<{count: number, name: string}>`, что эквивалентно `Context<Data>` (без `| undefined`).

Другой распространенной схемой является определение фабричной функции, которая производит значение для контекста. Затем мы можем получить возвращаемый тип этой функции с помощью помощника типа TypeScript [`ReturnType`](https://www.typescriptlang.org/docs/handbook/utility-types.html#returntypetype) и использовать его для ввода контекста:

```ts
export const makeCountNameContext = (
    initialCount = 0,
    initialName = ''
) => {
    const [count, setCount] = createSignal(initialCount);
    const [name, setName] = createSignal(initialName);
    return [
        { count, name },
        { setCount, setName },
    ] as const;
    // `as const` forces tuple type inference
};
type CountNameContextType = ReturnType<
    typeof makeCountNameContext
>;
export const CountNameContext =
    createContext<CountNameContextType>();
export const useCountNameContext = () =>
    useContext(CountNameContext);
```

В данном примере `CountNameContextType` соответствует возвращаемому значению `makeCountNameContext`:

```ts
[
    { count: Accessor<number>, name: Accessor<string> },
    { setCount: Setter<number>, setName: Setter<string> },
];
```

а `useCountNameContext` имеет тип `() => CountNameContextType | undefined`.

Если вы хотите избежать возможности `undefined`, вы можете утверждать, что контекст всегда предоставляется при использовании:

```ts
export const useCountNameContext = () =>
    useContext(CountNameContext)!;
```

Это опасное предположение; безопаснее было бы действительно предоставлять аргумент по умолчанию для `createContext`, чтобы контекст всегда был определен.

## Типы компонентов

```ts
import type { JSX, Component } from 'solid-js';
type Component<P = {}> = (props: P) => JSX.Element;
```

Для типизации базовой компонентной функции используйте тип `Component<P>`, где `P` - тип аргумента `props`, который должен быть [object type](https://www.typescriptlang.org/docs/handbook/2/objects.html). Это обеспечит передачу правильно типизированных свойств в качестве атрибутов, а также то, что возвращаемое значение является чем-то, что может быть отображено Solid: `JSX.Element` может быть узлом DOM, массивом `JSX.Element`, функцией, возвращающей `JSX.Element`, булевым числом, `undefined`/`null` и т.д. Приведем несколько примеров:

```tsx
const Counter: Component = () => {
    const [count, setCount] = createSignal(0);
    return (
        <button onClick={() => setCount((c) => c + 1)}>
            {count()}
        </button>
    );
};
<Counter />; // good
<Counter initial={5} />; // type error: no initial prop
<Counter>hi</Counter>; // type error: no children prop
const InitCounter: Component<{ initial: number }> = (
    props
) => {
    const [count, setCount] = createSignal(props.initial);
    return (
        <button onClick={() => setCount((c) => c + 1)}>
            {count()}
        </button>
    );
};
<InitCounter initial={5} />; // good
```

Если вы хотите, чтобы ваш компонент принимал дочерние элементы JSX, вы можете либо явно добавить тип для `children` в `P`, либо использовать тип `ParentComponent`, который автоматически добавляет `children? JSX.Element`. В качестве альтернативы, если вы хотите объявить свой компонент с `function` вместо `const`, вы можете использовать помощник `ParentProps` для типа `props`. Некоторые примеры:

```tsx
import {
    JSX,
    ParentComponent,
    ParentProps,
} from 'solid-js';
type ParentProps<P = {}> = P & { children?: JSX.Element };
type ParentComponent<P = {}> = Component<ParentProps<P>>;
// Equivalent typings:
//const CustomCounter: Component<{children?: JSX.Element}> = ...
//function CustomCounter(props: ParentProps): JSX.Element { ...
const CustomCounter: ParentComponent = (props) => {
    const [count, setCount] = createSignal(0);
    return (
        <button onClick={() => setCount((c) => c + 1)}>
            {count()}
            {props.children}
        </button>
    );
};
// Equivalent typings:
//const CustomInitCounter: Component<{initial: number, children?: JSX.Element}> = ...
//function CustomInitCounter(props: ParentProps<{initial: number}>): JSX.Element { ...
const CustomInitCounter: ParentComponent<{
    initial: number;
}> = (props) => {
    const [count, setCount] = createSignal(props.initial);
    return (
        <button onClick={() => setCount((c) => c + 1)}>
            {count()}
            {props.children}
        </button>
    );
};
```

В последнем примере параметр `props` автоматически приобретает вид `props: ParentProps<{initial: number}>`, что эквивалентно `props: {initial: number, children? JSX.Element}`. (Заметим, что до версии Solid 1.4 `Component` был эквивалентен `ParentComponent`).

Solid предоставляет еще два подтипа `Component` для работы с `children`:

```ts
import {
    JSX,
    FlowComponent,
    FlowProps,
    VoidComponent,
    VoidProps,
} from 'solid-js';
type FlowProps<P = {}, C = JSX.Element> = P & {
    children: C;
};
type FlowComponent<P = {}, C = JSX.Element> = Component<
    FlowProps<P, C>
>;
type VoidProps<P = {}> = P & { children?: never };
type VoidComponent<P = {}> = Component<VoidProps<P>>;
```

`VoidComponent` предназначен для компонентов, которые определенно не поддерживают `children`. `VoidComponent<P>` эквивалентен `Component<P>`, когда `P` не предоставляет тип для `children`.

`FlowComponent` предназначен для компонентов "потока управления", таких как `<Show>` и `<For>` в Solid. Такие компоненты обычно требуют наличия `children` для того, чтобы иметь смысл, и иногда имеют специфические типы для `children`, например, требуют, чтобы это была одна функция. Например:

```tsx
const CallMeMaybe: FlowComponent<
    { when: boolean },
    () => void
> = (props) => {
    createEffect(() => {
        if (props.when) props.children();
    });
    return <>{props.when ? 'Calling' : 'Not Calling'}</>;
};
<CallMeMaybe when={true} />; // type error: missing children
<CallMeMaybe when={true}>hi</CallMeMaybe>; // type error: children
<CallMeMaybe when={true}>
    {() => console.log("Here's my number")}
</CallMeMaybe>; // good
```

## Обработчики событий

Пространство имен `JSX` предлагает набор полезных типов, в частности, для работы с HTML DOM. Все предоставляемые типы см. в [определении JSX в dom-expressions](https://github.com/ryansolid/dom-expressions/blob/main/packages/dom-expressions/src/jsx.d.ts).

Одним из полезных вспомогательных типов, предоставляемых пространством имен `JSX`, является `JSX.EventHandler<T, E>`, который представляет собой одноаргументный обработчик событий для элемента DOM типа `T` и события типа `E`. Вы можете использовать его для ввода любых обработчиков событий, которые вы определяете вне JSX. Например:

```tsx
import type { JSX } from 'solid-js';
const onInput: JSX.EventHandler<
    HTMLInputElement,
    InputEvent
> = (event) => {
    console.log(
        'input changed to',
        event.currentTarget.value
    );
};
<input onInput={onInput} />;
```

Обработчики, определенные inline внутри [`on___` JSX-атрибутов](https://www.solidjs.com/docs/latest/api#on___) (со встроенными типами событий), автоматически типизируются как соответствующие `JSX.EventHandler`:

```tsx
<input
    onInput={(event) => {
        console.log(
            'input changed to',
            event.currentTarget.value
        );
    }}
/>
```

Обратите внимание, что `JSX.EventHandler<T>` ограничивает атрибут события [`currentTarget`](https://developer.mozilla.org/en-US/docs/Web/API/Event/currentTarget) типом `T` (в примере `event.currentTarget` типизирован как `HTMLInputEvent`, поэтому имеет атрибут `value`). Однако атрибут события [`target`](https://developer.mozilla.org/en-US/docs/Web/API/Event/target) может быть любым `DOMElement`.

Это связано с тем, что `currentTarget` - это элемент, к которому был прикреплен обработчик события, поэтому он имеет известный тип, а `target` - это то, с чем взаимодействовал пользователь, что вызвало появление события или его перехват обработчиком события, который может быть любым элементом DOM. Исключение составляют события ввода и фокусировки, когда они прикрепляются непосредственно к элементам `input`, в качестве `цели` будет указан HTMLInputElement.

## Атрибут ref

Когда мы используем атрибут `ref` с переменной, мы говорим Solid, что нужно присвоить переменной элемент DOM после того, как этот элемент будет отрисован. Без TypeScript это выглядит следующим образом:

```jsx
let divRef;
console.log(divRef); // undefined
onMount(() => {
    console.log(divRef); // <div> element
});
return <div ref={divRef} />;
```

Это создает проблему при вводе этой переменной: следует ли вводить `divRef` как `HTMLDivElement`, даже если она будет установлена как таковая только после рендеринга? (Здесь мы предполагаем, что режим `strictNullChecks` в TypeScript включен; в противном случае TypeScript игнорирует потенциально `неопределенные` переменные).

Наиболее безопасная схема в TypeScript - признать, что `divRef` является `неопределенной` на некоторое время, и проверять ее при использовании:

```tsx
let divRef: HTMLDivElement | undefined;
divRef.focus(); // correctly reported as an error at compile time
onMount(() => {
    if (!divRef) return;
    divRef.focus(); // correctly allowed
});
return <div ref={divRef}>...</div>;
```

В качестве альтернативы, поскольку мы знаем, что `onMount` вызывается только после рендеринга элемента `<div>`, мы могли бы использовать [nonnull assertion (`!`)](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-) при обращении к `divRef` внутри `onMount`:

```tsx
onMount(() => {
    divRef!.focus();
});
```

Другая достаточно безопасная схема - опустить `undefined` из типа `divRef` и использовать [definite assignment assertion (`!`)](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-7.html#definite-assignment-assertions) в атрибуте `ref`:

```tsx
let divRef: HTMLDivElement;
divRef.focus(); // correctly reported as an error at compile time
onMount(() => {
    divRef.focus(); // correctly allowed
});
return <div ref={divRef!}>...</div>;
```

Мы должны использовать `ref={divRef!}`, поскольку TypeScript предполагает, что атрибут `ref` устанавливается на переменную `divRef`, а значит, `divRef` уже должен быть присвоен. В Solid все наоборот: `divRef` присваивается атрибутом `ref`. Определенное утверждение присваивания `divRef!` эффективно убеждает TypeScript в том, что все происходит именно так: TypeScript поймет, что `divRef` был присвоен после этой строки.

При использовании этого шаблона TypeScript будет корректно отмечать любые случайные использования ссылок внутри тела функции (до блока JSX, в котором они определяются). Однако в настоящее время TypeScript не отмечает использование потенциально неопределенных переменных внутри вложенных функций. В контексте Solid необходимо следить за тем, чтобы не использовать рефссылки внутри `createMemo`, `createRenderEffect` и `createComputed` (до блока JSX, в котором определяются рефссылки), поскольку эти функции вызываются сразу, и рефссылки еще не будут определены (однако TypeScript не отметит это как ошибку). Напротив, в предыдущем шаблоне эти ошибки будут отловлены.

Другой распространенный, но менее безопасный способ - поместить утверждение об определенном присваивании в точку объявления переменной.

```tsx
let divRef!: HTMLDivElement;
divRef.focus(); // allowed despite causing an error
onMount(() => {
    divRef.focus(); // correctly allowed
});
return <div ref={divRef}>...</div>;
```

Такой подход фактически отключает проверку присваивания для этой переменной, что является простым обходным путем, но требует дополнительной осторожности. В частности, в отличие от предыдущего паттерна, он некорректно допускает преждевременное использование переменной даже вне вложенных функций.

## Сужение потока управления

Распространенным паттерном является использование [`<Show>`](https://www.solidjs.com/docs/latest/api#%3Cshow%3E) для отображения данных только тогда, когда эти данные определены:

```tsx
const [name, setName] = createSignal<string>();
return (
    <Show when={name()}>
        Hello {name().replace(/\s+/g, '\xa0')}!
    </Show>
);
```

В этом случае TypeScript не может определить, что два вызова `name()` вернут одно и то же значение и что второй вызов произойдет только в том случае, если первый вызов вернет истинное значение. Поэтому при попытке вызвать `.replace()` он будет жаловаться, что `name()` может быть `undefined`.

Вот два варианта решения этой проблемы:

1.  Вы можете вручную утверждать, что `name()` будет не-нулевым во втором вызове, используя оператор TypeScript [non-null assertion operator `!`](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#non-null-assertion-operator-postfix-):

    ```tsx
    return (
        <Show when={name()}>
            Hello {name()!.replace(/\s+/g, '\xa0')}!
        </Show>
    );
    ```

2.  Можно использовать форму обратного вызова `<Show>`, которая передает значение свойства `when`, когда оно истинно:

    ```tsx
    return (
        <Show when={name()}>
            {(n) => (
                <>Hello {n().replace(/\s+/g, '\xa0')}!</>
            )}
        </Show>
    );
    ```

    В данном случае типизация компонента `Show` достаточно умна, чтобы сообщить TypeScript, что `n` является истиной, поэтому не может быть `undefined` (или `null`, или `false`). Помните, что форма с утверждением null будет выброшена, если к ней обратиться, когда условие уже не будет истинным.

## Специальные атрибуты и директивы JSX

### `on:___`/`oncapture:___`

При использовании пользовательских обработчиков событий через атрибуты Solid [`on:___`/`oncapture:___`](https://www.solidjs.com/docs/latest/api#on%3A___%2Foncapture%3A___) необходимо определить соответствующие типы для получаемых объектов `Event`, переопределив интерфейсы `CustomEvents` и `CustomCaptureEvents` в пространстве имен модуля `"solid-js"` ` `JSX`, например, так:

```tsx
class NameEvent extends CustomEvent {
    type: 'Name';
    detail: { name: string };
    constructor(name: string) {
        super('Name', { detail: { name } });
    }
}
declare module 'solid-js' {
    namespace JSX {
        interface CustomEvents {
            // on:Name
            Name: NameEvent;
        }
        interface CustomCaptureEvents {
            // oncapture:Name
            Name: NameEvent;
        }
    }
}
<div
    on:Name={(event) =>
        console.log('name is', event.detail.name)
    }
/>;
```

### `prop:___`/`attr:___`

Если вы используете принудительные свойства через атрибуты Solid [`prop:___`](https://www.solidjs.com/docs/latest/api#prop%3A___) или пользовательские атрибуты через Solid [`attr:___` attributes](https://www.solidjs.com/docs/latest/api#attr%3A___), вы можете определить их типы в интерфейсах `ExplicitProperties` и `ExplicitAttributes` соответственно:

```tsx
declare module "solid-js" {
  namespace JSX {
    interface ExplicitProperties { // prop:___
      count: number;
      name: string;
    }
    interface ExplicitAttributes { // attr:___
      count: number;
      name: string;
    }
  }
}

<Input prop:name={name()} prop:count={count()} />
<my-web-component attr:name={name()} attr:count={count()} />
```

### `use:___`

Если вы определяете пользовательские директивы для атрибутов Solid [`use:___`](https://www.solidjs.com/docs/latest/api#use%3A___), вы можете ввести их в интерфейс `Directives`, например, так:

```tsx
function model(
    element: HTMLInputElement,
    value: Accessor<Signal<string>>
) {
    const [field, setField] = value();
    createRenderEffect(() => (element.value = field()));
    element.addEventListener('input', (e) => {
        const value = (e.target as HTMLInputElement).value;
        setField(value);
    });
}
declare module 'solid-js' {
    namespace JSX {
        interface Directives {
            // use:model
            model: Signal<string>;
        }
    }
}
let [name, setName] = createSignal('');
<input type="text" use:model={[name, setName]} />;
```

Если вы импортируете директиву `d` из другого файла/модуля, а `d` используется только как директива `use:d`, то TypeScript (точнее, [`babel-preset-typescript`](https://babeljs.io/docs/en/babel-preset-typescript)) по умолчанию удалит `импорт` `d` (опасаясь, что `d` - это тип, поскольку TypeScript не понимает `use:d` как ссылку на `d`). Обойти эту проблему можно двумя способами:

1.  Использовать опцию конфигурации [`babel-preset-typescript` `onlyRemoveTypeImports: true`](https://babeljs.io/docs/en/babel-preset-typescript#onlyremovetypeimports), которая не позволяет удалять любые импорты, кроме `import type ...`. Если вы используете `vite-plugin-solid`, то можете указать эту опцию через `solidPlugin({ typescript: { onlyRemoveTypeImports: true } })` в `vite.config.ts`.

    Обратите внимание, что эта опция может быть проблематичной, если вы не используете бдительно `export type` и `import type` во всей своей кодовой базе.

2.  Добавьте фальшивый доступ типа `false && d;` к каждому модулю, импортирующему директиву `d`. Это не позволит TypeScript удалить `импорт` модуля `d`, и, если вы используете древовидный метод, например [Terser](https://terser.org/), этот код будет опущен из вашего конечного пакета кода.

    Более простой фальшивый доступ `d;` также не позволит удалить `импорт`, но, как правило, не будет подвергаться древовидности, поэтому окажется в конечном коде.

## Ссылки

-   [Typescript for Solid](https://docs.solidjs.com/guides/foundations/typescript-for-solid#special-jsx-attributes-and-directives)
