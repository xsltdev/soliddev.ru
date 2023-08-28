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

???svelte "svelte"

    Это эквивалентно `$: console.log(count)` в Svelte.

???react "react"

    В React зависимости объявляются в явном виде с помощью массива зависимостей:

    ```js
    useEffect(() => {
    	console.log(count);
    }, [count]);
    ```

    В противном случае эффект будет запускаться заново при изменении _любого_ состояния компонента. В Solid зависимости отслеживаются автоматически, и вам не нужно беспокоиться о лишних повторных запусках.

???vue "vue"

    Это эквивалентно `watchEffect(() => console.log(count.value))` в Vue.

???angular "angular"

    Хотя Angular не может сравниться 1:1 с `createEffect` в Solid, они по касательной похожи на [хуки жизненного цикла](https://angdev.ru/angular/lifecycle-hooks/) Angular.

    Однако `createEffect` имеет ряд основных преимуществ перед методами жизненного цикла:

    -   Они могут выполняться вне компонентов.
    -   Они имеют более консолидированный API в результате отделения от компонентов.
    -   Они являются "композитными" (эффект можно поместить внутрь другого эффекта).

## Рендеринг с помощью сигналов

Прежде чем мы вернемся к нашей книжной полке, давайте посмотрим пример использования этих _примитивов_ внутри _компонентов_.

```ts
import { createSignal } from 'solid-js';
function Counter() {
    const [count, setCount] = createSignal(0);
    return <div>Current count: {count()}</div>;
}
```

Мы видим, что, как и другие переменные, мы можем использовать сигналы в нашем JSX-коде, заключая их в фигурные скобки. Пока этот компонент не слишком интересен, поэтому давайте добавим возможность увеличивать счетчик. Для этого добавим элемент `<button>` и зададим ему обработчик _клик_ с помощью атрибута `onClick`. Этот обработчик щелчка будет увеличивать наш счетчик с помощью функции `setCount`:

```ts
import { createSignal } from 'solid-js';
function Counter() {
    const [count, setCount] = createSignal(0);
    const increment = () => {
        setCount(count() + 1);
    };
    return (
        <div>
            Current count: {count()}
            <button onClick={increment}>Increment</button>
        </div>
    );
}
```

И теперь у нас есть работающий счетчик! Примечательно, что наш текст обновляется каждый раз, когда увеличивается `count`. Не напоминает ли это вам эффект? При изменении сигнала код, управляющий этой частью DOM, запускается заново, подобно тому, как код в нашем эффекте запускается при изменении `count`.

За кулисами компилятор Solid создает эффекты на основе нашего JSX. Он видит, что мы используем `count()` в определенной части DOM, и создает эффект, который обновляет именно эту часть DOM при повторном выполнении сигнала.

Философия Solid заключается в том, что, рассматривая все как сигнал или эффект, мы можем лучше рассуждать о нашем приложении.

## Пересмотр книжной полки

Теперь у нас есть инструменты, необходимые для того, чтобы сделать наше приложение "Книжная полка" интерактивным. В качестве иллюстрации приведем текущее состояние приложения со следующими компонентами:

-   `BookList`, список книг на нашей Книжной полке
-   `AddBook` - форма, позволяющая добавлять книги на полку.
-   `Bookshelf`, наш основной компонент приложения, который содержит два других компонента

=== "App.tsx"

    ```ts
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

    ```ts
    export function AddBook() {
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit">Add book</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    export function BookList() {
    	return (
    		<ul>
    			<li>
    				Code Complete{' '}
    				<span style={{ 'font-style': 'italic' }}>
    					(Steve McConnell)
    				</span>
    			</li>
    			<li>
    				The Hobbit{' '}
    				<span style={{ 'font-style': 'italic' }}>
    					(J.R.R. Tolkien)
    				</span>
    			</li>
    		</ul>
    	);
    }
    ```

В качестве первого шага к добавлению интерактивности давайте добавим сигнал, который будет отслеживать список книг. Назовем его `books`, и он будет находиться в компоненте `BookList`. Каждая книга будет иметь `title` и `author`.

=== "App.tsx"

    ```ts
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

    ```ts
    export function AddBook() {
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit">Add book</button>
    		</form>
    	);
    }
    ```

=== "BookList\*.tsx"

    ```ts
    import { createSignal } from 'solid-js';
    type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    export function BookList() {
    	const [books, setBooks] = createSignal(initialBooks);
    	return (
    		<ul>
    			<li>
    				{books()[0].title}{' '}
    				<span style={{ 'font-style': 'italic' }}>
    					({books()[0].author})
    				</span>
    			</li>
    			<li>
    				{books()[1].title}{' '}
    				<span style={{ 'font-style': 'italic' }}>
    					({books()[1].author})
    				</span>
    			</li>
    		</ul>
    	);
    }
    ```

Здесь следует отметить несколько моментов:

Во-первых, хотя до сих пор мы использовали `createSignal` только для сохранения значения числа в состоянии, он может управлять всеми видами состояния. В нашем приложении "Книжная полка" сигналом является массив объектов.

Во-вторых, теперь мы используем `books` непосредственно в нашем JSX-коде. Мы вызываем `books()` для доступа к массиву сигналов, затем обращаемся к элементу с индексом `0` (ноль) этого массива в первом элементе списка и к элементу с индексом `1` этого массива во втором элементе списка. Это работает, но не является гибким: мы хотим обрабатывать динамическое количество книг.

## Перебор элементов

Лучшим способом перебора элементов в Solid является компонент `<For />`. Компонент `<For />` имеет пропс `each`, которому мы можем передать наш массив `books()`.

```tsx
<For each={books()}></For>
```

Внутри компонента `For` мы используем функцию _callback_, которая будет применяться к каждому элементу массива. В данном случае мы хотим, чтобы каждая `книга` отображалась внутри `<li>`.

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

???react "react"

    В React мы бы использовали `array.map`:

    ```jsx
    {
    	books.map((book) => (
    		<li key={book.title}>
    			{book.title} ({book.author})
    		</li>
    	));
    }
    ```

    Если бы мы использовали здесь `array.map` в Solid, то _каждый элемент_ внутри книги пришлось бы перерисовывать при каждом изменении сигнала `books`.

    Компонент `For` проверяет массив при его изменении и обновляет только необходимый элемент. Это та же самая проверка, которую выполняет система рендеринга VDOM в React, когда мы используем `.map`.

    Обратите внимание, что, в отличие от React, нам не нужно указывать `ключ` компоненту `For`: он сравнивает каждый элемент по ссылке.

???vue "vue"

    В Vue вышеописанное будет выглядеть следующим образом:

    ```html
    <li v-for="book of books" key="book.title">
    	{{book.title}} ({{book.author}})
    </li>
    ```

    Обратите внимание, что, в отличие от Vue, нам не нужно указывать `key` компоненту `For`: он сравнивает каждый элемент по ссылке.

???svelte "svelte"

    В Svelte вышеизложенное будет выглядеть следующим образом:

    ```svelte
    {#each books as book (book.title) }
    <li>{book.title} ({book.author})</li>
    {#each}
    ```

    Обратите внимание, что, в отличие от Svelte, нам не нужно предоставлять 'key' компоненту `For`: он сравнивает каждый элемент по ссылке.

Теперь наш компонент `BookList` выглядит следующим образом:

=== "App.tsx"

    ```ts
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

    ```ts
    export function AddBook() {
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit">Add book</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    import { createSignal, For } from 'solid-js';
    type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    export function BookList() {
    	const [books, setBooks] = createSignal(initialBooks);
    	return (
    		<ul>
    			<For each={books()}>
    				{(book) => {
    					return (
    						<li>
    							{book.title}
    							<span
    								style={{
    									'font-style': 'italic',
    								}}
    							>
    								{' '}
    								({book.author})
    							</span>
    						</li>
    					);
    				}}
    			</For>
    		</ul>
    	);
    }
    ```

## Производное состояние

Solid позволяет легко отслеживать _производное состояние_. Вы можете представить себе производное состояние как вычисления, основанные только на другой информации, которую вы уже отслеживаете в состоянии. В нашем приложении "Книжная полка" примером производного состояния может служить количество книг в списке: это длина массива `books` в любой момент времени.

В Solid для вычисления производного состояния достаточно создать _производный сигнал_: функцию, которая опирается на другой сигнал:

```tsx
const totalBooks = () => books().length;
```

Теперь при каждом вызове `totalBooks()` Solid будет регистрировать базовый сигнал (`books`) как зависимость, поэтому вычисляемое значение всегда будет актуальным.

=== "App.tsx"

    ```ts
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

    ```ts
    export function AddBook() {
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit">Add book</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    import { createSignal, For } from 'solid-js';
    type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    export function BookList() {
    	const [books, setBooks] = createSignal(initialBooks);
    	const totalBooks = () => books().length;
    	return (
    		<>
    			<h2>My books ({totalBooks()})</h2>
    			<ul>
    				<For each={books()}>
    					{(book) => {
    						return (
    							<li>
    								{book.title}
    								<span
    									style={{
    										'font-style':
    											'italic',
    									}}
    								>
    									{' '}
    									({book.author})
    								</span>
    							</li>
    						);
    					}}
    				</For>
    			</ul>
    		</>
    	);
    }
    ```

???vue "vue"

    В Vue можно написать:

    ```js
    const totalBooks = computed(() => books.length);
    ```

    При этом в памяти создается место для вычисленного значения, которое постоянно обновляется при изменении `books`. Таким образом, если вы дважды использовали `{{totalBooks}}` в своем шаблоне, то `books.length` будет вызван только один раз. Производные сигналы в Solid не создают место в памяти; при каждом вызове `totalBooks()` будет повторно выполняться код `books().length`.

    Для этого в Solid есть другая функция: `createMemo`, которая будет повторно выполнять вычисления только при изменении зависимости.

    ```js
    import { createMemo } from 'solid-js';

    const totalBooks = createMemo(() => books().length);
    ```

???svelte "svelte"

    В Svelte можно написать:

    ```js
    	$: totalBooks = books.length;
    ```

    При этом в памяти создается место для вычисленного значения, которое постоянно обновляется при изменении `books`. Таким образом, если вы дважды использовали `{totalBooks}` в своем шаблоне, то `books.length` будет вызвана только один раз.

    Производные сигналы в Solid не создают место в памяти; при каждом вызове `totalBooks()` будет повторно выполняться код `books().length`.

    Для этого в Solid есть другая функция: `createMemo`, которая будет повторно выполнять вычисления только при изменении зависимости.

    ```js
    import { createMemo } from 'solid-js';

    const totalBooks = createMemo(() => books().length);
    ```

## Поднятие состояния

Мы хотим добавить книгу в список с помощью компонента `AddBook`. Однако есть одна проблема: как сделать так, чтобы сеттер `setBooks` был доступен компоненту `AddBooks`?

Мы знаем, что родители могут передавать пропсы дочерним компонентам, но как _родственные_ компоненты могут передавать пропсы друг другу? Это распространенная проблема в Solid, и решение обычно заключается в том, чтобы _поднимать состояние вверх_ до общего родителя. В данном случае наш сигнал `books` может находиться в компоненте `Bookshelf`. Затем компоненту `BookList` можно передать данные из геттера.

Давайте начнем с того, что поднимем наш сигнал `books` в компонент `Bookshelf` и передадим его значение обратно в компонент `BookList`. Вы можете увидеть изменения, которые мы сделали в файлах `App.tsx` и `BookList.tsx`.

=== "App.tsx"

    ```ts
    import { createSignal } from 'solid-js';
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    export type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	const [books, setBooks] = createSignal(initialBooks);
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList books={books()} />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

    ```ts
    export function AddBook() {
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit">Add book</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    import { For } from 'solid-js';
    import { Book } from './App';
    interface BookListProps {
    	books: Book[];
    }
    export function BookList(props: BookListProps) {
    	const totalBooks = () => props.books.length;
    	return (
    		<>
    			<h2>My books ({totalBooks()})</h2>
    			<ul>
    				<For each={props.books}>
    					{(book) => {
    						return (
    							<li>
    								{book.title}
    								<span
    									style={{
    										'font-style':
    											'italic',
    									}}
    								>
    									{' '}
    									({book.author})
    								</span>
    							</li>
    						);
    					}}
    				</For>
    			</ul>
    		</>
    	);
    }
    ```

Теперь наш массив книг находится в компоненте `Bookshelf`. Затем мы передаем `books()` компоненту `BookList`. Теперь мы можем получить доступ к нашим книгам внутри компонента `BookList` с помощью пропса `props.books`.

!!!note ""

    Вы, наверное, заметили, что при передаче компоненту `BookList` сигнала `books()` мы вызвали `books()` &mdash; это не опечатка! В Solid принято вызывать аксессор сигнала при передаче его компоненту. В фоновом режиме Solid делает это _реактивным реквизитом_, и реактивность будет отслеживаться в JSX дочернего компонента. (_TODO: хорошее место для ссылки на обсуждение/руководство по пропсам и реактивности_).

## Добавление книг в список

Теперь, когда у нас есть поднятое состояние, мы можем добавить несколько книг в список. Передадим наш сеттер компоненту `AddBook` и вызовем `setBooks` при нажатии на кнопку `Add Book`. Эти изменения можно увидеть в файлах `App.tsx` и `AddBook.tsx`:

=== "App.tsx\*"

    ```ts
    import { createSignal } from 'solid-js';
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    export type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	const [books, setBooks] = createSignal(initialBooks);
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList books={books()} />
    			<AddBook setBooks={setBooks} />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="Solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx\*"

    ```ts
    import { Setter, JSX } from 'solid-js';
    import { Book } from './App';
    export interface AddBookProps {
    	setBooks: Setter<Book[]>;
    }
    export function AddBook(props: AddBookProps) {
    	const addBook: JSX.EventHandler<
    		HTMLButtonElement,
    		MouseEvent
    	> = (event) => {
    		event.preventDefault();
    		props.setBooks([]);
    	};
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input id="title" />
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input id="author" />
    			</div>
    			<button type="submit" onClick={addBook}>
    				Add book
    			</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    import { For } from 'solid-js';
    import { Book } from './App';
    interface BookListProps {
    	books: Book[];
    }
    export function BookList(props: BookListProps) {
    	const totalBooks = () => props.books.length;
    	return (
    		<>
    			<h2>My books ({totalBooks()})</h2>
    			<ul>
    				<For each={props.books}>
    					{(book) => {
    						return (
    							<li>
    								{book.title}
    								<span
    									style={{
    										'font-style':
    											'italic',
    									}}
    								>
    									{' '}
    									({book.author})
    								</span>
    							</li>
    						);
    					}}
    				</For>
    			</ul>
    		</>
    	);
    }
    ```

Внутри `AddBook` мы создали функцию `addBook`, которая используется в качестве обработчика _клик_ для кнопки нашей формы. Поскольку мы отправляем настоящую HTML-форму, мы используем `event.preventDefault()`, чтобы предотвратить стандартное поведение формы, заключающееся в выполнении post-запроса. Далее мы вызываем `props.setBooks`, но мы не совсем понимаем, что передать нашему сеттеру.

Мы знаем, что хотим сохранить существующие книги в списке, а затем добавить новую книгу, полученную из нашей формы. Чтобы получить существующие книги, мы могли бы использовать два различных подхода: мы могли бы _передать_ сигнал `books` нашему компоненту `AddBook`. Хотя это и сработает, стоит рассмотреть второй вариант: использование формы _callback function_ в сеттере. Мы его еще не использовали, а синтаксис выглядит следующим образом:

```ts
setCount((currentCount) => {
    return currentCount + 1;
});
```

Используя эту форму, наш сеттер получает доступ к текущему значению сигнала.

Такая форма для нашей функции `setBooks` решает первую проблему: наша функция `addBook` может быть записана следующим образом:

```js
const addBook = (event) => {
    event.preventDefault();
    props.setBooks((books) => {
        return books;
    });
};
```

Теперь нам необходимо добавить в этот список текст из вводимых форм. Для этого мы можем создать новый сигнал внутри компонента `AddBook`, который будет отслеживать значения вводимых данных. Для того чтобы этот сигнал всегда был равен значениям вводимых данных, мы используем его обработчик `onInput`. Кроме того, мы _привяжем_ сигнал `newBook()` к атрибуту `value` нашего `input`, чтобы убедиться, что наш `input` всегда отражает значение сигнала.

Наконец, мы хотим добавить `newBook` в список книг, а затем очистить поле ввода на случай, если пользователь захочет ввести еще несколько книг.

=== "App.tsx"

    ```ts
    import { createSignal } from 'solid-js';
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    export type Book = {
    	title: string;
    	author: string;
    };
    const initialBooks: Book[] = [
    	{ title: 'Code Complete', author: 'Steve McConnell' },
    	{ title: 'The Hobbit', author: 'J.R.R. Tolkien' },
    	{
    		title: 'Living a Feminist Life',
    		author: 'Sarah Ahmed',
    	},
    ];
    interface BookshelfProps {
    	name: string;
    }
    function Bookshelf(props: BookshelfProps) {
    	const [books, setBooks] = createSignal(initialBooks);
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList books={books()} />
    			<AddBook setBooks={setBooks} />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="Solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx\*"

    ```ts
    import { createSignal, Setter, JSX } from 'solid-js';
    import { Book } from './App';
    export interface AddBookProps {
    	setBooks: Setter<Book[]>;
    }
    const emptyBook: Book = { title: '', author: '' };
    export function AddBook(props: AddBookProps) {
    	const [newBook, setNewBook] = createSignal(emptyBook);
    	const addBook: JSX.EventHandler<
    		HTMLButtonElement,
    		MouseEvent
    	> = (event) => {
    		event.preventDefault();
    		props.setBooks((books) => [...books, newBook()]);
    		setNewBook(emptyBook);
    	};
    	return (
    		<form>
    			<div>
    				<label for="title">Book name</label>
    				<input
    					id="title"
    					value={newBook().title}
    					onInput={(e) => {
    						setNewBook({
    							...newBook(),
    							title: e.currentTarget.value,
    						});
    					}}
    				/>
    			</div>
    			<div>
    				<label for="author">Author</label>
    				<input
    					id="author"
    					value={newBook().author}
    					onInput={(e) => {
    						setNewBook({
    							...newBook(),
    							author: e.currentTarget.value,
    						});
    					}}
    				/>
    			</div>
    			<button type="submit" onClick={addBook}>
    				Add book
    			</button>
    		</form>
    	);
    }
    ```

=== "BookList.tsx"

    ```ts
    import { For } from 'solid-js';
    import { Book } from './App';
    interface BookListProps {
    	books: Book[];
    }
    export function BookList(props: BookListProps) {
    	const totalBooks = () => props.books.length;
    	return (
    		<>
    			<h2>My books ({totalBooks()})</h2>
    			<ul>
    				<For each={props.books}>
    					{(book) => {
    						return (
    							<li>
    								{book.title}
    								<span
    									style={{
    										'font-style':
    											'italic',
    									}}
    								>
    									{' '}
    									({book.author})
    								</span>
    							</li>
    						);
    					}}
    				</For>
    			</ul>
    		</>
    	);
    }
    ```

!!!note ""

    Мы использовали оператор spread для создания нового массива books внутри нашего сеттера books. Это общий паттерн для Solid, который позволяет убедиться в том, что мы создаем новый массив, а не обновляем (или _мутируем_) существующий массив сигналов. По умолчанию Solid использует проверку ссылочного равенства при определении обновления сигнала.

## Тестирование нашего приложения

Теперь у нас есть динамическое приложение "Книжная полка"! Попробуйте сами: вы должны иметь возможность добавлять книги с помощью компонента `AddBook` и видеть, как эти книги добавляются в список в компоненте `BookList`.

## Ссылки

-   [Adding Interactivity with State](https://docs.solidjs.com/guides/tutorials/getting-started-with-solid/adding-interactivity-with-state#rendering-with-signals)
