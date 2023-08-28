---
description: В динамических внешних приложениях обычно требуется отображать различные пользовательские интерфейсы, когда приложение находится в разных состояниях
---

# Условное отображение пользовательского интерфейса

В динамических внешних приложениях обычно требуется отображать различные пользовательские интерфейсы, когда приложение находится в разных состояниях. Примером может служить отображение приветственного сообщения для гостя или аутентифицированного пользователя.

Для гостя мы можем отобразить общее приветственное сообщение и форму регистрации:

```ts
<div>Welcome to the application. Please sign in to continue.</div>
<SignInForm />
```

Для аутентифицированного пользователя мы, возможно, захотим поприветствовать его по имени и предоставить ему его пользовательскую панель:

```ts
<div>Welcome back, Jessica!</div>
<Dashboard />
```

## Условный показ содержимого

В Solid мы можем использовать компонент `<Show />` для условного показа содержимого. Компонент `<Show />` принимает пропс `when` и необязательный пропс `fallback`.

-   Когда параметр `when` имеет значение `true`, JSX внутри компонента `<Show />` отображается
-   Если параметр `when` равен `false`, то отображается JSX внутри `fallback` (если он задан).

В следующем примере показано, как можно использовать компонент `<Show />` для условного отображения страницы входа в систему или приборной панели:

```ts
import { Show } from 'solid-js';
interface IHomeProps {
    isLoggedIn: boolean;
    firstName: string;
}
function Home(props: IHomeProps) {
    return (
        <Show
            when={props.isLoggedin}
            fallback={
                <>
                    <div>
                        Welcome to the application. Please
                        sign in to continue.
                    </div>
                    <SignInForm />
                </>
            }
        >
            <div>Welcome back, {props.firstName}!</div>
            <Dashboard />
        </Show>
    );
}
```

Когда значение `props.isLoggedIn` равно `true`, мы приветствуем вернувшегося пользователя и показываем `<Dashboard />`. Когда значение `props.isLoggedIn` равно `false`, мы отображаем содержимое `fallback`, которое представляет собой наше общее приветствие и форму `<SignInForm />`.

???react "react"

    В React это распространенный паттерн для обработки потока управления путем раннего возврата из функции компонента. Например, для выполнения условного отображения аутентификации можно было бы сделать следующее:

    ```js
    function Home(props) {
    	if (props.isLoggedIn) {
    		return (
    			<>
    				<div>Welcome back, {props.firstName}!</div>
    				<Dashboard />
    			</>
    		);
    	}

    	return (
    		<>
    			<div>
    				Welcome to the application. Please sign in
    				to continue.
    			</div>
    			<SignInForm />
    		</>
    	);
    }
    ```

    Однако **это не будет работать в Solid!**.

    В разделе [Building UI with Components](building-ui-with-components.md) этого руководства мы отметили, что функции компонентов _работают только один раз_ в Solid. Это означает, что JSX, возвращаемый при первоначальном возврате функции, является единственным JSX, который когда-либо будет возвращен функцией.

    В Solid, если мы хотим условно отобразить JSX в компоненте, нам нужно, чтобы это условие находилось в возвращаемом JSX. Хотя это требует некоторой адаптации при переходе с React, мы пришли к выводу, что тонкий контроль, предоставляемый реактивной системой Solid, стоит того, чтобы пойти на этот компромисс.

???angular "angular"

    Вместо `Show` в Angular может использоваться `ngIf`. Аналогично, вместо `fallback` в `ngIf` будет использоваться предложение `else`, а также `ng-template`.

    ```html
    <ng-container *ngIf="isLoggedin; else notLoggedIn">
    	<div>Welcome back, {{firstName}}!</div>
    	<dashboard></dashboard>
    </ng-container>
    <ng-template #notLoggedIn>
    	<div>
    		Welcome to the application. Please sign in to
    		continue.
    	</div>
    	<sign-in-form></sign-in-form>
    </ng-template>
    ```

???vue "vue"

    В Vue вышеописанное будет выглядеть следующим образом:

    ```html
    <template v-if="isLoggedIn">
    	<div>
    		Welcome to the application. Please sign in to
    		continue.
    	</div>
    	<SignInForm />
    </template>
    <template v-else>
    	<div>Welcome back, {{firstName}}!</div>
    	<Dashboard />
    </template>
    ```

## Итерация данных с помощью `<For />`

В пользовательских интерфейсах часто требуется отображать списки данных. Такие списки могут быть любой длины, и поэтому мы не можем просто жестко закодировать каждый элемент. Вместо этого Solid предоставляет нам компонент `<For />`. Если вы кодили по примеру приложения "Книжная полка", то заметили, что нам уже приходилось использовать этот компонент.

Компонент `<For />` принимает массив, по которому нужно выполнить цикл, в пропсе `each`:

```js
const books = ['Book 1', 'Book 2'];

<For each={books}>...</For>;
```

Внутри компонента `<For />` используется функция _callback_ для перебора элементов. В данном случае мы создаем новый элемент списка `<li>` для каждой книги в нашем массиве `books`:

```js
const books = ['Book 1', 'Book 2'];

<For each={books}>
    {(book) => {
        return <li>{book}</li>;
    }}
</For>;
```

??react "react"

    В React итерация по массивам в JSX осуществляется с помощью метода массива `map`:

    ```js
    <>
    	{books.map((book) => {
    		return <li key={book}>{book}</li>;
    	})}
    </>
    ```

    Хотя это и работает в Solid, но не является оптимальным. Можно рассматривать `<For />` как оптимизированную версию `map`. При использовании `<For />` Solid способен интеллектуально определить, какие элементы массива необходимо обновить. Именно поэтому нам не нужно использовать `key`, как в React.

## Revisiting the bookshelf

In the [Adding Interactivity with State](adding-interactivity-with-state.md) section of this tutorial, we found ourselves already needing the `<For />` component. This allowed us to iterate over any number of books in on our bookshelf:

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

=== "AddBook.tsx"

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

Теперь воспользуемся компонентом `<Show />`. Мы будем показывать нашу форму `AddBook` только в том случае, если пользователь хочет добавить книгу.

Мы создадим в компоненте `Bookshelf` булевый сигнал, который будет отслеживать, открыта форма или нет, и добавим кнопки для открытия и закрытия формы. Для условного отображения формы мы будем использовать компонент `<Show />`.

=== "App.tsx\*"

    ```ts
    import { createSignal, Show } from 'solid-js';
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
    	const [showForm, setShowForm] = createSignal(false);
    	const toggleForm = () => setShowForm(!showForm());
    	return (
    		<div>
    			<h1>{props.name}'s Bookshelf</h1>
    			<BookList books={books()} />
    			<Show
    				when={showForm()}
    				fallback={
    					<button onClick={toggleForm}>
    						Add a book
    					</button>
    				}
    			>
    				<AddBook setBooks={setBooks} />
    				<button onClick={toggleForm}>
    					Finished adding books
    				</button>
    			</Show>
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf name="Solid" />;
    }
    export default App;
    ```

=== "AddBook.tsx"

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

Когда `showForm()` имеет значение `true`, приложение отображает форму `<AddBook />` и кнопку, позволяющую снова скрыть форму. Когда `showForm()` имеет значение `false`, отображается компонент `fallback` &mdash; кнопка для показа формы `<AddBook />`.

## Ссылки

-   [Conditional User Interface Display](https://docs.solidjs.com/guides/tutorials/getting-started-with-solid/control-flow)
