---
description: В этом уроке мы узнаем, как добавить _состояние_ в наше приложение. Затем мы применим эти знания к нашему приложению Bookshelf, чтобы воплотить его в жизнь
---

# Добавление интерактивности с помощью состояния

В предыдущем уроке мы узнали, как создать приложение Solid с использованием компонентов и JSX. Затем мы создали приложение Bookshelf, используя наши новые знания, но оно не совсем закончено. В этом уроке мы узнаем, как добавить _состояние_ в наше приложение. Затем мы применим эти знания к нашему приложению Bookshelf, чтобы воплотить его в жизнь.

## Замечание о примитивах

По мере изучения этих уроков мы начнем слышать о _примитивах_ Solid. В то время как компоненты являются строительными блоками _видов_ в приложениях Solid, примитивы являются строительными блоками _взаимодействия_. Первый примитив, с которым мы познакомимся, - это _сигнал_.

## Управление базовым состоянием с помощью сигналов

В Solid самым основным способом управления состоянием нашего приложения является использование _сигнала_. Для создания сигнала Solid предоставляет функцию `createSignal`:

```ts
import { createSignal } from 'solid-js';

const [count, setCount] = createSignal(0);
```

Здесь происходит много всего: сначала мы вызываем `createSignal` с начальным значением state. В данном случае `count` будет начинаться с `0`. Функция `createSignal` возвращает двухэлементный массив, и мы используем JavaScript _деструктурирующее присваивание_ для распаковки этого массива. В данном случае мы присваиваем первый элемент переменной `count`, а второй элемент - переменной `setCount`.

Первый элемент, `count`, представляет собой _акцессорную_ функцию (также называемую _геттером_), которая возвращает текущее значение состояния.

Важно отметить, что это именно _функция получения текущего значения_, а не само значение. Попутно этот вызов функции сообщает Solid, что мы получили доступ к сигналу.

```ts
import { createSignal } from 'solid-js';

const [count, setCount] = createSignal(0);

console.log(count()); // 0
```

???svelte "svelte"

    Это эквивалентно `let count = 0` в Svelte.

    В Svelte вы можете использовать реактивное значение, не вызывая его как функцию. Это связано с тем, что реактивность Svelte использует компилятор, в то время как реактивность Solid является частью библиотеки. Поскольку мы работаем в рамках ограничений JavaScript, нам необходимо запускать код при обращении к значению, чтобы сообщить реактивной системе: "Это значение используется здесь!".

???react "react"

    Это эквивалентно `const [count, setCount] = useState(0)` в React.

    В React можно использовать реактивное значение, не вызывая его как функцию. Это связано с тем, что система рендеринга React будет перезапускать весь компонент при любом изменении состояния - нет ничего особенного в настройке отдельных частей состояния, поэтому нет необходимости запускать какой-либо код при использовании реактивного значения.

???vue "vue"

    Это эквивалентно `const count = ref(0)` в Vue.

    Вместо того чтобы разделять count на функцию-геттер и функцию-сеттер, Vue предлагает использовать `count.value` как для получения, так и для установки значения. Реактивная система Vue похожа на систему Solid: нам необходимо запускать код за кулисами при каждом обращении к `ref`.

    В Vue этот код находится внутри функции `.value` [getter function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get).

???angular "angular"

    В Angular это может быть просто свойство класса, которое вы мутируете. Это одно из самых больших различий между Solid и Angular. В то время как Solid полагается на явное обновление свойств, Angular позволяет мутировать переменные, чтобы вызвать обнаружение изменений.

    Хотя это может показаться чисто умственным излишеством, у Solid есть два основных преимущества разделения чтения и записи:

    1.  Не требуется дополнительных затрат производительности на обнаружение изменений
    2.  Не требуется никакой дополнительной логики (например, [`runOutsideAngular`](https://angular.io/api/core/NgZone#runOutsideAngular)), чтобы избежать обнаружения изменений.

Второй элемент, `setCount`, представляет собой функцию _setter_. Если мы хотим увеличить `count`, мы можем передать `count() + 1` в `setCount`:

```ts
import { createSignal } from 'solid-js';

const [count, setCount] = createSignal(0);

setCount(count() + 1);

console.log(count()); // 1
```

Обратите внимание, что для того, чтобы увидеть новое значение `count`, мы добавили отчет `console.log` после использования `setCount`.

Ключевым моментом в реактивной системе Solid является то, что на самом деле нам не нужно этого делать. Вместо этого мы можем слушать &mdash; и мгновенно реагировать &mdash; на любые изменения сигнала, используя наш следующий примитив - _эффект_.

## Реакция на изменения с помощью эффектов

Возможность реагировать на изменения сигналов лежит в основе реактивной системы Solid. Самый простой способ сделать это - использовать _эффект_. Мы можем создать эффект с помощью хука `createEffect`:

```ts
import { createSignal, createEffect } from 'solid-js';

const [count, setCount] = createSignal(0);

createEffect(() => {
    console.log(count());
});

setCount(count() + 1);
```

Чтобы использовать `createEffect`, мы передаем ему функцию. При обновлении сигналов, используемых в этой функции, функция будет перезапущена.

В данном примере наш эффект зависит от `count`, поэтому он запускается при изменении `count`. Соответственно, мы, как и раньше, выводим в консоль сообщение `1`.

Автоматическое отслеживание зависимостей эффектов стало возможным благодаря тому, что `count` является функцией. Когда функция `count` вызывается внутри эффекта, этот эффект регистрируется как слушатель сигнала. Вот почему так важно, чтобы наши сигналы были функциями!

<FrameworkAside framework="svelte">
  This is equivalent to `$: console.log(count)` in Svelte.
</FrameworkAside>
<FrameworkAside framework="react">
  In React, you'd declare the dependencies explicitly using the dependency array:
  ```js
    useEffect(() => {
      console.log(count);
    }, [count])
  ```
    If you didn't, the effect would rerun whenever _any_ state in the component changes.
    In Solid, dependencies are tracked automatically, and you don't have to worry about extra reruns.
</FrameworkAside>
<FrameworkAside framework="vue">
  This is equivalent to `watchEffect(() => console.log(count.value))` in Vue.
</FrameworkAside>
<FrameworkAside framework="angular">
    While Angular doesn't have a 1:1 comparison to Solid's `createEffect`, they're tangentially similar to Angular's [lifecycle hooks](https://angular.io/guide/lifecycle-hooks).

    However, `createEffect` has some primary benefits over lifecycle methods:
    - They can run outside of components.
    - They have a more consolidated API as a result of their decoupling from components.
    - They're "composable" (You can put an effect inside another effect)

</FrameworkAside>

## Rendering with signals

Before we get back to our bookshelf, let's see an example of how these _primitives_ can be used inside _components_.

<CodeTabs
js={[{ name: "Counter.jsx", component: Counter1 }]}
ts={[{ name: "Counter.tsx", component: Counter1 }]}
/>

<div>Current count: 0</div>

We see that, much like other variables, we can use signals inside our JSX code by including them inside curly braces. This component is not too interesting yet; so let's add the ability to increment our count. We can do this by adding a `<button>` element and giving it a _click handler_ using the `onClick` attribute. This click handler will increment our count by using the `setCount` function:

<CodeTabs
js={[{ name: "Counter.jsx", component: Counter2 }]}
ts={[{ name: "Counter.tsx", component: Counter2 }]}
/>

<BasicCounter />

And now we have a functioning counter! Notably, our text updates whenever our `count` is incremented. Does this remind you of an effect? Whenever the signal changes, the code that controls that part of the DOM reruns, similar to how the code in our effect reran whenever `count` changed.

Behind the scenes, Solid's compiler creates effects based on our JSX. It sees that we're using `count()` in a specific part of the DOM, and it creates an effect that updates just that part of the DOM when the signal reruns.

A driving philosophy of Solid is that, by treating everything as a signal or an effect, we can better reason about our application.

## Revisting the bookshelf

We now have the tools necessary to make our Bookshelf application interactive. As a refresher, here's the current state of the app with the following components:

-   `BookList`, a list of books on our Bookshelf
-   `AddBook`, a form that will allow us to add more books to the shelf
-   `Bookshelf`, our main application component that contains the other two

<CodeTabs
js={[
{ name: "App.jsx", component: App2js },
{ name: "AddBook.jsx", component: AddBook1 },
{ name: "BookList.jsx", component: BookList1 },

]}ts={[
{ name: "App.tsx", component: App2ts },
{ name: "AddBook.tsx", component: AddBook1 },
{ name: "BookList.tsx", component: BookList1 },
]}
/>

As a first step to adding interactivity, let's add a signal that keeps track of our book list. We'll call it `books` and it will live in the `BookList` component. Each book will have a `title` and an `author`.

<CodeTabs
js={[
{ name: "App.jsx", component: App2js },
{ name: "AddBook.jsx", component: AddBook1 },
{ name: "BookList\*.jsx", component: BookList2js, default: true },

]}ts={[
{ name: "App.tsx", component: App2ts },
{ name: "AddBook.tsx", component: AddBook1 },
{ name: "BookList*.tsx", component: BookList2ts, default: true },
]}
/>

There are a couple of things to note here:

First, while we had only used `createSignal` to maintain the value of a number in state thus far, it can manage all kinds of state. In our Bookshelf application, our signal is an array of objects.

Second, we're now using `books` directly in our JSX code. We call `books()` to access the signal array, and then access the element at index 0 (zero) of that array in the first list item and the element at index 1 of that array in the second list item. This will work, but it's not flexible: we want to handle a dynamic number of books.

## Looping over items

The best way to loop over items in Solid is the `<For />` component. The `<For />` component has an `each` prop, to which we can pass our `books()` array.

```tsx
<For each={books()}></For>
```

Inside the `For` component, we use a _callback function_ that will be applied to _each_ element in the array. In this instance, we want each `book` to be rendered inside an `<li>`.

```tsx
<For each={books()}>
    {(book) => {
        return (
            <li>
                {book.title} ({book.author})
            </li>
        );
    }}
</For>
```

<FrameworkAside framework="react">
In React, we'd use `array.map`:
```jsx
{books.map(book => <li key={book.title}>{book.title} ({book.author}</li>)}
```

If we used `array.map` here in Solid, _every element_ inside the book would have to rerender whenever the `books` signal changes.
The `For` component checks the array when it changes, and only updates the necessary element. It's the same kind of checking that React's VDOM rendering system does for us when we use `.map`.

Note that, unlike in React, we don't need to provide a `key` to the `For` component: it compares each element by reference.
</FrameworkAside>

<FrameworkAside framework="vue">
The above would be written as the following in Vue:

```html
<li v-for="book of books" key="book.title">
    {{book.title}} ({{book.author}})
</li>
```

Note that, unlike in Vue, we don't need to provide a `key` to the `For` component: it compares each element by reference.
</FrameworkAside>

<FrameworkAside framework="svelte">
The above would be written as the following in Svelte:

```svelte
{#each books as book (book.title) }
  <li>{book.title} ({book.author})</li>
{#each}
```

Note that, unlike in Svelte, we don't need to provide a `key` to the `For` component: it compares each element by reference.
</FrameworkAside>

Our `BookList` component now looks like this:

<CodeTabs
js={[
{ name: "App.jsx", component: App2js },
{ name: "AddBook.jsx", component: AddBook1 },
{ name: "BookList.jsx", component: BookList3js, default: true },

]}ts={[
{ name: "App.tsx", component: App2ts },
{ name: "AddBook.tsx", component: AddBook1 },
{ name: "BookList.tsx", component: BookList3ts, default: true },
]}
/>

## Derived state

Solid makes it easy to track _derived state_. You can think of derived state as a computation based only on other information you're already tracking in state. In our Bookshelf application, an example of derived state would be the number of books on our list: it's the length of our `books` array at any point in time.

In Solid, all we have to do to compute derived state is to create a _derived signal_: a function that relies on another signal:

```tsx
const totalBooks = () => books().length;
```

Now, whenever we call `totalBooks()`, Solid will register the underlying signal (`books`) as a dependency, so the computed value will always stay up-to-date.

<CodeTabs
js={[
{ name: "App.jsx", component: App2js },
{ name: "AddBook.jsx", component: AddBook1 },
{ name: "BookList.jsx", component: BookList4js, default: true },

]}ts={[
{ name: "App.tsx", component: App2ts },
{ name: "AddBook.tsx", component: AddBook1 },
{ name: "BookList.tsx", component: BookList4ts, default: true },
]}
/>

<FrameworkAside framework="vue">
  In Vue, you might write:

```vue
const totalBooks = computed(() => books.length)
```

This creates a spot in memory for the computed value, and that memory is continually updated as `books` changes.
So, if you used `{{totalBooks}}` twice in your template, `books.length` would only be called once.
Derived signals in Solid don't create a spot in memory; every time `totalBooks()` is called, it will rerun the code `books().length`.
Solid has another feature for this: `createMemo`, which will only rerun the computation when the dependency changes.

```js
import { createMemo } from 'solid-js';

const totalBooks = createMemo(() => books().length);
```

</FrameworkAside>

<FrameworkAside framework="svelte">
In Svelte, you might write:
```svelte
	$: totalBooks = books.length;
```
  This creates a spot in memory for the computed value, and that memory is continually updated as `books` changes. So, if you used `{totalBooks}` twice in your template, `books.length` would only be called once.
  Derived signals in Solid don't create a spot in memory; every time `totalBooks()` is called, it will rerun the code `books().length`.
  Solid has another feature for this: `createMemo`, which will only rerun the computation when the dependency changes.

```js
import { createMemo } from 'solid-js';

const totalBooks = createMemo(() => books().length);
```

</FrameworkAside>

## Lifting state up

We want to add a book to the list using our `AddBook` component. There's one problem though: how do we make the `setBooks` setter available to the `AddBooks` component?

We know that parents can pass props to children, but how do _sibling_ components pass props to each other? This is a common problem in Solid and the solution is generally to _lift state up_ to a common parent. In this case, our `books` signal can live in the `Bookshelf` component. Then, the `BookList` component can be passed the data from the getter.

Let's start out by lifting our `books` signal up to `Bookshelf` and passing its value back down to the `BookList` component. You can see the changes we have made in both the `App.tsx` and `BookList.tsx` files.

<CodeTabs
js={[
{ name: "App.jsx", component: App3js },
{ name: "AddBook.jsx", component: AddBook1 },
{ name: "BookList.jsx", component: BookList5js },

]}ts={[
{ name: "App.tsx", component: App3ts },
{ name: "AddBook.tsx", component: AddBook1 },
{ name: "BookList.tsx", component: BookList5ts },
]}
/>

Our array of books now lives in the `Bookshelf` component. We then pass `books()` to the `BookList` component. We can now access our books within the `BookList` component by using `props.books`.

<Aside type="note">
 You may have noticed that we called `books()` when we passed it to the `BookList` component&mdash;this is not a typo! In Solid, it's a best practice to call a signal accessor when you pass it to a component. In the background, Solid makes this a _reactive prop_ and reactivity will be tracked in the child component's JSX. (_TODO: good place to link to a discussion/guide on props and reactivity_).
</Aside>

## Adding books to the list

Now that we have lifted state, we can add some books to the list. Let's pass our setter to the `AddBook` component and call `setBooks` when we click the `Add Book` button. You can see these changes in the `App.tsx` and `AddBook.tsx` files:

<CodeTabs
js={[
{ name: "App.jsx*", component: App4js },
{ name: "AddBook.jsx*", component: AddBook2js },
{ name: "BookList.jsx", component: BookList5js },

]}ts={[
{ name: "App.tsx*", component: App4ts },
{ name: "AddBook.tsx*", component: AddBook2ts },
{ name: "BookList.tsx", component: BookList5ts },
]}
/>

Inside `AddBook`, we created a function called `addBook` that is used as the _click handler_ for our form's button. Since we're submitting a real HTML form, we use `event.preventDefault()` to prevent the default form behavior of executing a post request. Next, we call `props.setBooks`, but we don't quite know what to pass to our setter.

We know we want to keep the existing books on the list and then add a new book that comes from our form input. To get the existing books, we could use two different approaches: we _could_ pass the `books` signal down to our `AddBook` component. While that would work, it's worth exploring the second option: using the _callback function_ form of the setter. We haven't used this yet, and the syntax is as follows:

```tsx
setCount((currentCount) => {
    return currentCount + 1;
});
```

By using this form, our setter has access to the current value of the signal.

This form for our `setBooks` function solves the first problem: our `addBook` function can be written as follows:

```jsx
const addBook = (event) => {
    event.preventDefault();
    props.setBooks((books) => {
        return books;
    });
};
```

Now, we need to append the text from our form inputs to this list. To do so, we can create a new signal inside the `AddBook` component to track the value of the inputs. We'll make sure this signal is always equal to the inputs' values by using its `onInput` handler. Additionally, we'll _bind_ the `newBook()` to the `value` attribute of our `input` to make sure our `input` always reflects the value of the signal.

Finally, we want to add the `newBook` to our books list and then clear the input field in case our user has more books to enter.

<CodeTabs
js={[
{ name: "App.jsx", component: App4js },
{ name: "AddBook.jsx\*", component: AddBook3js, default: true },
{ name: "BookList.jsx", component: BookList5js },

]}ts={[
{ name: "App.tsx", component: App4ts },
{ name: "AddBook.tsx*", component: AddBook3ts, default: true },
{ name: "BookList.tsx", component: BookList5ts },
]}
/>

<Aside type="note">
We used the spread operator to create an new books array inside our books setter. This is a common pattern in Solid and helps to make sure we create a new array rather than update (or _mutate_) the existing signal array. By default, Solid uses referential equality checks when determining if a signal has updated.
</Aside>

## Test-driving our app

We now have a dynamic Bookshelf application! Try it out yourself: you should be able to add books using the `AddBook` component and see those books added to the list in the `BookList` component.

<BasicBookshelf name="Solid" />
