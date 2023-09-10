---
description: Разделяет реактивный объект по ключам
---

# splitProps

```ts
function splitProps<T>(
    props: T,
    ...keys: Array<(keyof T)[]>
): [...parts: Partial<T>];
```

Разделяет реактивный объект по ключам.

Принимает реактивный объект и любое количество массивов ключей; для каждого массива ключей возвращает реактивный объект, содержащий только те свойства, которые были у исходного объекта. Последний реактивный объект в возвращаемом массиве будет иметь все оставшиеся свойства исходного объекта.

Это может быть полезно, если вы хотите потреблять некоторое количество свойств, а остальные передавать дочернему объекту.

```tsx
function MyComponent(props) {
    const [local, others] = splitProps(props, ['children']);

    return (
        <>
            <div>{local.children}</div>
            <Child {...others} />
        </>
    );
}
```

Поскольку `splitProps` принимает любое количество массивов, мы можем разделить объект реквизита на сколько угодно частей (например, если у нас есть несколько дочерних компонентов, каждому из которых требуется свое подмножество свойств).

Допустим, компоненту было передано шесть свойств:

```tsx
<MyComponent a={1} b={2} c={3} d={4} e={5} foo="bar" />;
// ...

function MyComponent(props) {
    console.log(props); // {a: 1, b: 2, c: 3, d: 4, e: 5, foo: "bar"}
    const [vowels, consonants, leftovers] = splitProps(
        props,
        ['a', 'e'],
        ['b', 'c', 'd']
    );
    console.log(vowels); // {a: 1, e: 5}
    console.log(consonants); // {b: 2, c: 3, d: 4}
    console.log(leftovers.foo); // bar
}
```

## Ссылки

-   [splitProps](https://docs.solidjs.com/references/api-reference/reactive-utilities/splitProps)
