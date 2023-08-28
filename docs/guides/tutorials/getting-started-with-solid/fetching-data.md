---
description: Ресурсы - это специальные сигналы, разработанные специально для асинхронной загрузки и создаваемые с помощью функции createResource
---

# Получение данных

Наше приложение Bookshelf почти завершено, но мы живем в динамичном мире. Наши приложения практически никогда не бывают самодостаточными. Именно поэтому в Solid изначально заложена концепция _ресурсов_. Ресурсы - это специальные сигналы, разработанные специально для асинхронной загрузки и создаваемые с помощью функции `createResource`.

Ресурсы могут быть вызваны самими сигналами. Обычно в функцию `createResource` передаются два параметра: сигнал и функция извлечения данных, которая полагается на этот сигнал:

```tsx
const [data] = createResource(signal, dataFetchingFunction);
```

Сигнал _триггерит_ функцию `dataFetchingFunction`: всякий раз, когда он становится значением, отличным от `null`, `undefined` или `false`, вызывается функция `dataFetchingFunction` с этим значением в качестве первого аргумента.

Когда наша функция `dataFetchingFunction` завершится, она обновит значение `data()`, к которому мы можем получить доступ как к обычному сигналу. Кроме того, нам будут доступны свойства `data.loading` и `data.error`, чтобы мы могли реагировать на состояние выборки данных.

В дальнейшем, если значение `signal` снова изменится, функция `dataFetchingFunction` будет запущена заново (если это значение не `null`, `undefined` или `false`).

## Получение данных для книжной полки

В нашем приложении "Книжная полка" мы не собираемся заставлять пользователей набирать полное название и автора каждой книги: для поиска книг мы будем использовать API.

К счастью, [OpenLibrary.org](https://openlibrary.org/) предоставляет API, который мы можем использовать для поиска книг по их названиям. Например, `https://openlibrary.org/search.json?q=Lord%20of%20the%20Rings` выполнит поиск по книге "Властелин колец", и мы сможем получить официальное название книги и ее автора.

Давайте сначала напишем функцию поиска данных `searchBooks`, основанную на этом API. Функция будет принимать поисковый запрос и возвращать массив совпадений из API OpenLibrary.org:

=== "App.tsx"

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

=== "searchBooks.ts\*"

    ```ts
    type ResultItem = {
    	title: string;
    	author_name: string[];
    };
    export async function searchBooks(query: string) {
    	if (query.trim() === '') return [];
    	const response = await fetch(
    		`https://openlibrary.org/search.json?q=${encodeURI(
    			query
    		)}`
    	);
    	const results = await response.json();
    	const documents = results.docs as ResultItem[];
    	console.log(documents);
    	return documents
    		.slice(0, 10)
    		.map(({ title, author_name }) => ({
    			title,
    			author: author_name?.join(', '),
    		}));
    }
    ```

Этот код получает данные из API OpenLibrary.org, выбирает первые 10 результатов, а затем немного преобразует их, чтобы вернуть только свойства `title` и `author` для каждого результата.

Отложим на время эту функцию получения данных и займемся рефакторингом нашего компонента `AddBooks.tsx`. В настоящее время он состоит из формы, позволяющей пользователю ввести _и_ название, и автора книги, которую он хочет добавить на свою книжную полку.

Теперь, когда мы знаем, что будем использовать API OpenLibrary.org, мы можем немного упростить эту форму: мы просто попросим пользователя искать книгу по ее названию. Текст, который набирает пользователь, будет задан в сигнале `input`. После того как пользователь нажмет кнопку `Search`, текущее значение сигнала `input` будет сохранено в сигнале `query`. Это будет исходный сигнал для нашего ресурса.

=== "App.tsx"

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

=== "AddBook.tsx\*"

    ```ts
    import { createSignal, Setter, JSX } from 'solid-js';
    import { Book } from './App';
    export interface AddBookProps {
    	setBooks: Setter<Book[]>;
    }
    export function AddBook(props: AddBookProps) {
    	const [input, setInput] = createSignal('');
    	const [query, setQuery] = createSignal('');
    	return (
    		<form>
    			<div>
    				<label for="title">Search books</label>
    				<input
    					id="title"
    					value={input()}
    					onInput={(e) => {
    						setInput(e.currentTarget.value);
    					}}
    				/>
    			</div>
    			<button
    				type="submit"
    				onClick={(e) => {
    					e.preventDefault();
    					setQuery(input());
    				}}
    			>
    				Search
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

=== "searchBooks.ts"

    ```ts
    type ResultItem = {
    	title: string;
    	author_name: string[];
    };
    export async function searchBooks(query: string) {
    	if (query.trim() === '') return [];
    	const response = await fetch(
    		`https://openlibrary.org/search.json?q=${encodeURI(
    			query
    		)}`
    	);
    	const results = await response.json();
    	const documents = results.docs as ResultItem[];
    	console.log(documents);
    	return documents
    		.slice(0, 10)
    		.map(({ title, author_name }) => ({
    			title,
    			author: author_name?.join(', '),
    		}));
    }
    ```

Теперь, когда у нас есть исходный сигнал `query` и функция получения данных `searchBooks`, мы можем использовать `createResource` для запроса к API OpenLibrary.org! Напомним, что `createResource` принимает в качестве аргументов исходный сигнал и функцию поиска данных:

```ts
const [data] = createResource(query, searchBooks);
```

Зная, что в состоянии загрузки мы будем иметь доступ к `data.loading`, мы можем вывести на экран "Searching...", когда данные загружаются, или список результатов, когда данные загружены. Воспользуемся изученными функциями потока управления `<Show />` и `<For />`.

```ts
const [data] = createResource(query, searchBooks);

<Show when={!data.loading} fallback={<>Searching...</>}>
    <ul>
        <For each={data()}>
            {(book) => (
                <li>
                    {book.title} by {book.author}{' '}
                    <button>Add</button>
                </li>
            )}
        </For>
    </ul>
</Show>;
```

Наконец, мы хотим убедиться, что `<button />` в каждом элементе списка добавляет элемент в наш список книг.

```ts
const [data] = createResource(query, searchBooks);

<Show when={!data.loading} fallback={<>Searching...</>}>
    <ul>
        <For each={data()}>
            {(book) => (
                <li>
                    {book.title} by {book.author}{' '}
                    <button
                        aria-label={`Add ${book.title} by ${book.author} to the bookshelf`}
                        onClick={(e) => {
                            e.preventDefault();
                            props.setBooks((books) => [
                                ...books,
                                book,
                            ]);
                        }}
                    >
                        Add
                    </button>
                </li>
            )}
        </For>
    </ul>
</Show>;
```

!!!note ""

    Мы можем включить `aria-label` в нашу кнопку Add, чтобы сделать действие, выполняемое кнопкой, понятным для читателей экрана. Solid никогда не встанет на пути написания доступного кода, но он также не напишет его за вас!

## Завершение работы приложения

Подключим этот код получения и отображения ресурсов к нашему файлу `AddBook`, который завершит работу приложения:

=== "App.tsx"

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

=== "AddBook.tsx\*"

    ```ts
    import {
    	createSignal,
    	Setter,
    	JSX,
    	createResource,
    	For,
    	Show,
    } from 'solid-js';
    import { Book } from './App';
    import { searchBooks } from './searchBooks';
    export interface AddBookProps {
    	setBooks: Setter<Book[]>;
    }
    export function AddBook(props: AddBookProps) {
    	const [input, setInput] = createSignal('');
    	const [query, setQuery] = createSignal('');
    	const [data] = createResource<Book[], string>(
    		query,
    		searchBooks
    	);
    	return (
    		<>
    			<form>
    				<div>
    					<label for="title">Search books</label>
    					<input
    						id="title"
    						value={input()}
    						onInput={(e) => {
    							setInput(e.currentTarget.value);
    						}}
    					/>
    				</div>
    				<button
    					type="submit"
    					onClick={(e) => {
    						e.preventDefault();
    						setQuery(input());
    					}}
    				>
    					Search
    				</button>
    			</form>
    			<Show
    				when={!data.loading}
    				fallback={<>Searching...</>}
    			>
    				<ul>
    					<For each={data()}>
    						{(book) => (
    							<li>
    								{book.title} by{' '}
    								{book.author}{' '}
    								<button
    									aria-label={`Add ${book.title} by ${book.author} to the bookshelf`}
    									onClick={(e) => {
    										e.preventDefault();
    										props.setBooks(
    											(books) => [
    												...books,
    												book,
    											]
    										);
    									}}
    								>
    									Add
    								</button>
    							</li>
    						)}
    					</For>
    				</ul>
    			</Show>
    		</>
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

=== "searchBooks.ts"

    ```ts
    type ResultItem = {
    	title: string;
    	author_name: string[];
    };
    export async function searchBooks(query: string) {
    	if (query.trim() === '') return [];
    	const response = await fetch(
    		`https://openlibrary.org/search.json?q=${encodeURI(
    			query
    		)}`
    	);
    	const results = await response.json();
    	const documents = results.docs as ResultItem[];
    	console.log(documents);
    	return documents
    		.slice(0, 10)
    		.map(({ title, author_name }) => ({
    			title,
    			author: author_name?.join(', '),
    		}));
    }
    ```

## Поздравляем!

Вы только что создали свое первое приложение на Solid! Теперь вы знаете, как создавать пользовательские интерфейсы, управлять состоянием, условно отображать информацию и динамически получать содержимое с помощью Solid.

## Ссылки

-   [Fetching Data](https://docs.solidjs.com/guides/tutorials/getting-started-with-solid/fetching-data)
