---
description: Трекинг - это механизм, который Solid использует для "отслеживания" того, когда был получен доступ к сигналу. Так он узнает, какие эффекты следует повторно запустить при изменении сигнала
---

# Трекинг

**Вначале изучите эти темы:** _сигналы_, _эффекты_, _предметы_.

**Трекинг** - это механизм, который Solid использует для "отслеживания" того, когда был получен доступ к сигналу.
Так он узнает, какие эффекты следует повторно запустить при изменении сигнала.

## Что нужно знать

1.  Цель отслеживания - регистрация подписок. Когда эффект "подписывается" на сигнал, он будет повторно запускаться при изменении этого сигнала.
2.  Отслеживание происходит при доступе. При обращении к сигналу (или хранимому значению) внутри эффекта сигнал "отслеживается", и эффект подписывается на него.
3.  Отслеживание происходит не везде. Если обратиться к сигналу вне эффекта, то отслеживания не произойдет. Это связано с тем, что нет эффекта, который мог бы подписаться на этот сигнал.
4.  Эмпирическое правило **отслеживания**: Обращайтесь к реактивному значению в то же время, когда вы его _используете_.

Это относится не только к сигналам, созданным с помощью `createSignal` - свойства и хранилища работают аналогичным образом.

## Пример

```jsx
function Counter() {
    const [count, setCount] = createSignal(0);
    const increment = () => setCount(count() + 1);

    /* When count() is called, the "count" signal is tracked as a subscription of this effect,
    so this code reruns when (and only when) count changes */
    createEffect(() => {
        console.log('My effect says ' + count());
    });

    /* This code isn't in a "tracking scope", so count()
     doesn't do anything special and this code never reruns */
    console.log(count());

    return (
        /*
      JSX is a tracking scope (it uses effects behind the scenes), so this code
      registers count as a subscription when count() is called
    */
        <button type="button" onClick={increment}>
            {count()}
        </button>
    );
}
```

## Проблемы с трекингом

Рассмотрим несколько примеров, в которых мы можем применить **элемент отслеживания** для отладки кода.

### Производное состояние

В этом примере абзац не будет обновляться при изменении `count`, поскольку доступ к `count` был получен вне области отслеживания.

```jsx
function Counter() {
    const [count, setCount] = createSignal(0);
    const increment = () => setCount(count() + 1);

    const doubleCount = count() * 2;
    return (
        <>
            <p>Twice my count: {doubleCount}</p>
            <button type="button" onClick={increment}>
                {count()}
            </button>
        </>
    );
}
```

Решение заключается в том, чтобы превратить `doubleCount` в _функцию_. Таким образом, при использовании `doubleCount` в JSX будет вызываться `count()` и регистрироваться подписка.

```jsx
function Counter() {
    const [count, setCount] = createSignal(0);
    const increment = () => setCount(count() + 1);

    const doubleCount = () => count() * 2;
    return (
        <>
            <p>Twice my count: {doubleCount()}</p>
            <button type="button" onClick={increment}>
                {count()}
            </button>
        </>
    );
}
```

### Деструктуризация свойств

В этом примере абзац в `DoubleCountView` никогда не будет обновляться.

```jsx
function DoubleCountView(props) {
    const { value } = props;
    const doubleCount = () => value * 2;
    return <p>{doubleCount()}</p>;
}

function Counter() {
    const [count, setCount] = createSignal(0);
    const increment = () => setCount(count() + 1);

    return (
        <>
            <DoubleCountView value={count()} />
            <button type="button" onClick={increment}>
                {count()}
            </button>
        </>
    );
}
```

Здесь реактивным значением является свойство `value`, переданное от родителя. Где находится доступ к этому значению?

```jsx
const { value } = props;
```

Это деструктурирующее присваивание является единственным кодом, который действительно обращается к свойствам `props.value`! После этого `value` просто представляет собой статическое значение, к которому обращались в данный момент.

Чтобы исправить это, нам необходимо убедиться, что мы _доступаем_ к значению одновременно с тем, как _используем_ его. Мы можем вызвать `props.value` напрямую:

```jsx
function DoubleCountView(props) {
    const doubleCount = () => props.value * 2;
    return <p>{doubleCount()}</p>;
}
```

В качестве альтернативы можно деструктурировать свойства `props` одновременно с их использованием:

```jsx
function DoubleCountView(props) {
    const doubleCount = () => {
        const { value } = props;
        return value * 2;
    };
    return <p>{doubleCount()}</p>;
}
```

## Ссылки

-   [Tracking](https://docs.solidjs.com/references/concepts/reactivity/tracking)
