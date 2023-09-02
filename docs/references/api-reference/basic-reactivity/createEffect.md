---
description: Эффекты - это общий способ заставить произвольный код ("побочные эффекты") выполняться при изменении зависимостей, например, для ручной модификации DOM
---

# createEffect

```ts
function createEffect<T>(fn: (v: T) => T, value?: T): void;
```

**Эффекты** - это общий способ заставить произвольный код ("побочные эффекты") выполняться при изменении зависимостей, например, для ручной модификации DOM. `createEffect` создает новое вычисление, которое запускает заданную функцию в отслеживаемой области видимости, таким образом автоматически отслеживая ее зависимости, и автоматически перевыполняет функцию при обновлении зависимостей. Например:

```ts
const [a, setA] = createSignal(initialValue);

// effect that depends on signal `a`
createEffect(() => doSideEffect(a()));
```

Эффект будет запускаться всякий раз, когда `a` будет менять значение.

Эффект также будет запущен один раз, сразу после его создания, для инициализации DOM до нужного состояния. Это называется фазой "монтирования". Однако мы рекомендуем использовать вместо этого `onMount`, который является более явным способом выражения этого.

Обратный вызов эффекта может возвращать значение, которое будет передано в качестве аргумента `prev` при следующем вызове эффекта. Это полезно для запоминания значений, которые дорого вычислять. Например:

```ts
const [a, setA] = createSignal(initialValue);

// effect that depends on signal `a`
createEffect((prevA) => {
    // do something with `a` and `prevA`
    const sum = a() + b();
    if (sum !== prevA) console.log('sum changed to', sum);
    return sum;
}, 0);
// ^ the initial value of the effect is 0
```

Эффекты предназначены в основном для побочных эффектов, которые читают, но не пишут в реактивную систему: лучше избегать установки сигналов в эффектах, которые при отсутствии осторожности могут вызвать дополнительный рендеринг или даже бесконечные циклы эффектов. Вместо этого лучше использовать [createMemo](createMemo.md) для вычисления новых значений, зависящих от других реактивных значений, чтобы реактивная система знала, что от чего зависит, и могла оптимизировать соответствующим образом.

Первое выполнение функции эффекта происходит не сразу, а по расписанию после завершения текущей фазы рендеринга (например, после вызова функции, переданной в [render](../rendering/render.md), [createRoot](../reactive-utilities/createRoot.md) или [runWithOwner](../reactive-utilities/runWithOwner.md)). Если необходимо дождаться первого выполнения, используйте [queueMicrotask](https://developer.mozilla.org/docs/Web/API/queueMicrotask) (выполняется до рендеринга DOM браузером) или await Promise.resolve() или setTimeout(..., 0) (выполняется после рендеринга браузером).

```ts
// assume this code is in a component function, so is part of a rendering phase
const [count, setCount] = createSignal(0);

// this effect prints count at the beginning and when it changes
createEffect(() => console.log('count =', count()));
// effect won't run yet
console.log('hello');
setCount(1); // effect still won't run yet
setCount(2); // effect still won't run yet

queueMicrotask(() => {
    // now `count = 2` will print
    console.log('microtask');
    setCount(3); // immediately prints `count = 3`
    console.log('goodbye');
});

// --- overall output: ---
// hello
// count = 2
// microtask
// count = 3
// goodbye
```

Такая задержка первого выполнения полезна, поскольку означает, что эффект, определенный в области видимости компонента, запускается после того, как JSX, возвращаемый компонентом, будет добавлен в DOM. В частности, [refs](../special-jsx-attributes/ref.md) уже будет установлен. Таким образом, эффект можно использовать для ручного управления DOM, вызова библиотек vanilla JS или других побочных эффектов.

Обратите внимание, что первый запуск эффекта все равно выполняется до того, как браузер отрендерит DOM на экран (аналогично `useLayoutEffect` в React). Если необходимо дождаться окончания рендеринга (например, для измерения рендеринга), можно использовать `await Promise.resolve()` (или `Promise.resolve().then(...)`), но следует учитывать, что последующее использование реактивного состояния (например, сигналов) не вызовет повторного запуска эффекта, поскольку отслеживание невозможно после использования `await` в асинхронной функции. Таким образом, все зависимости следует использовать до промиса.

Если вы хотите, чтобы эффект выполнялся сразу, даже при первом запуске, используйте [createRenderEffect](../secondary-primitives/createRenderEffect.md) или [createComputed](../secondary-primitives/createComputed.md).

Очистку побочных эффектов между выполнениями функции эффекта можно производить вызовом [onCleanup](../lifecycles/onCleanup.md) внутри функции эффекта. Такая функция очистки вызывается как между выполнениями эффекта, так и при его утилизации (например, при размонтировании содержащего компонента). Например:

```ts
// listen to event dynamically given by eventName signal
createEffect(() => {
    const event = eventName();
    const callback = (e) => console.log(e);
    ref.addEventListener(event, callback);
    onCleanup(() =>
        ref.removeEventListener(event, callback)
    );
});
```

## Аргументы

-   `fn` - Функция для запуска в области отслеживания. Она может возвращать значение, которое будет передано в качестве аргумента `prev` при следующем вызове эффекта.
-   `value` - Начальное значение эффекта. Это полезно для запоминания значений, которые дорого вычислять.

## Ссылки

-   [createEffect](https://docs.solidjs.com/references/api-reference/basic-reactivity/createEffect)
