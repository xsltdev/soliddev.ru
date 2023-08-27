---
description: Компонент - это функция, определяющая часть пользовательского интерфейса. Они являются строительными блоками приложения Solid
---

# Построение UI с помощью компонентов

## Что такое компоненты?

**Компонент** - это функция, определяющая часть пользовательского интерфейса. Они являются строительными блоками приложения Solid.

Давайте создадим наш первый компонент. Этот компонент будет отображать `Hello World!` внутри элемента `<div>`. Для этого в папке `src` необходимо создать файл `HelloWorld.tsx`. Затем скопируйте в него следующее содержимое:

```ts
export function HelloWorld() {
    return <div>Hello World!</div>;
}
```

В нашем первом компоненте есть несколько моментов, на которые следует обратить внимание:

-   Имя нашего компонента, `HelloWorld`, имеет заглавную букву (иногда ее называют _паскалевской_), причем _*H*ello_ и _*W*orld_ написаны с большой буквы.
    Это помогает нам отличать компоненты от других функций JavaScript.
-   Наш компонент возвращает нечто, очень похожее на HTML, но это не так &mdash; это _JSX_.

???react "React"

    Хотя компоненты Solid похожи на компоненты React, они отличаются друг от друга по нескольким признакам. Функции компонентов Solid запускаются только один раз для настройки и больше не запускаются. Если вы привыкли помещать вычисления в тело функции компонента React и ожидать, что они будут повторно вычисляться при рендеринге компонента, то в Solid вам, возможно, придется использовать другой подход. Не волнуйтесь &mdash; мы рассмотрим все лучшие практики в этом руководстве.

    Как и в React, в файл можно включать сколько угодно компонентов. Можно даже написать компонент _внутри_ другого!

???vue "Vue"

    В Vue компоненты имеют _жизненный цикл_, который важен для работы с ними. В Solid компоненты - это просто функции, которые группируют ваш код. У них нет жизненных циклов, а система рендеринга Solid основана на _реактивных_ функциях, которые вы используете&mdash; мы еще вернемся к этому!

    В Solid можно включать в один файл сколько угодно компонентов. Можно даже написать компонент _внутри_ другого!

???svelte "Svelte"

    В Svelte компоненты имеют _жизненные циклы_, которые важны для работы с ними. В Solid компоненты - это просто функции, которые группируют ваш код. У них нет жизненных циклов, а система рендеринга Solid основана на _реактивных_ функциях, которые вы используете &mdash; мы еще вернемся к этому!

    В Svelte используются однофайловые компоненты, а в Solid - нет. В один файл можно включать сколько угодно компонентов. Можно даже написать компонент _внутри_ другого!

???angular "Angular"

    В Angular компоненты имеют _жизненный цикл_, который важен для работы с ними. В Solid компоненты - это просто функции, которые группируют ваш код. У них нет жизненных циклов, а система рендеринга Solid основана не на границах компонентов, а на реактивных функциях, которые вы используете &mdash; мы еще вернемся к этому! Еще один момент: если в одном файле можно написать несколько компонентов Angular, используя свойство-декоратор компонента `template`, то в Solid это реализовано на другом уровне.

    В Solid можно не только включать сколько угодно компонентов в один файл, но даже писать компонент _вместе_ с другим!

### Использование компонента

Давайте выведем этот компонент на экран. Экспортируем его из файла `HelloWorld.tsx`, заменив `function` на `export function`.

Затем заменим шаблон в файле `App.tsx` на код, который импортирует и отображает наш компонент:

=== "HelloWorld.tsx"

    ```ts
    export function HelloWorld() {
    	return <div>Hello World!</div>;
    }
    ```

=== "App.tsx"

    ```ts
    import { HelloWorld } from './HelloWorld';
    function App() {
    	return (
    		<div>
    			<h1>Welcome</h1>
    			<HelloWorld />
    		</div>
    	);
    }
    export default App;
    ```

Мы использовали наш компонент так же, как и любой другой элемент: импортировав функцию `HelloWorld`, мы теперь можем использовать `<HelloWorld />`.

!!!note ""

    Построение с использованием компонентов позволяет разделить код и использовать его повторно. Компоненты позволяют нам создавать строительные блоки, которые мы можем соединять вместе для создания новых частей.

## Что такое JSX?

Как мы узнали выше, компоненты Solid возвращают JSX. JSX - это HTML-подобный синтаксис, используемый в JavaScript-коде Solid для представления части объектной модели документа (DOM). В нашем примере `HelloWorld.tsx`, приведенном выше, мы вернули следующий JSX из нашего компонента:

```ts
<div>Hello World!</div>
```

Этот JSX представляет собой элемент `<div>` с текстом `Hello World!` внутри. Это так же, как если бы мы написали его в HTML-документе. Чтобы продемонстрировать возможности и гибкость Solid, давайте добавим в наш элемент `<div>` несколько встроенных стилей:

```ts
<div style="background-color: #2c4f7c; color: #FFF;">
    Hello world!
</div>
```

В результате будет создан `div` с синим цветом фона и белым текстом:

<div style="background-color: #2c4f7c; color: #FFF;">Hello world!</div>

Попытка создать стилизованный `div` с помощью JavaScript _без_ JSX будет более сложной:

```ts
const div = document.createElement('div');
div.style['background-color'] = '#2c4f7c';
div.style.color = '#FFF';
```

???vue "vue"

    Vue имеет свой собственный "язык шаблонов" в специальном типе файлов под названием `.vue`. `.jsx` выполняет аналогичную роль в Solid, но не интегрирован непосредственно в JavaScript. Мы можем написать JSX-элемент в любом месте, где мы могли бы написать выражение JavaScript, и нет никакого принудительного разделения между шаблоном и данными.

???svelte "svelte"

    В Svelte имеется собственный "язык шаблонов" в специальном типе файлов `.svelte`.`.jsx` выполняет аналогичную роль в Solid, но не интегрирован непосредственно в JavaScript. Мы можем написать JSX-элемент в любом месте, где мы могли бы написать выражение JavaScript, и нет никакого принудительного разделения между шаблоном и данными.

???angular "angular"

    Вместо JSX в Angular имеется собственный "компилятор шаблонов", который принимает строки шаблонов и компилирует их в инструкции JavaScript. Он выполняет ту же роль, что и JSX, но не интегрирован непосредственно в JavaScript. В Solid можно написать JSX-элемент в любом месте, где можно написать выражение JavaScript, и нет принудительного разделения между шаблоном и данными.

### JSX + JavaScript

JSX становится еще более мощным, если использовать его вместе с выражениями JavaScript. Например, мы можем использовать переменные для представления некоторого текстового содержимого в нашем `<div>`:

```ts
const name = 'Solid';

<div style="background-color: #2c4f7c; color: #FFF;">
    Hello {name}!
</div>;
```

Обратите внимание, как мы можем добавить `{name}` туда, где раньше было написано `World`. Это выражение JavaScript позволяет нам изменять содержимое нашего `<div>`:

<div style="background-color: #2c4f7c; color: #FFF;">Hello Solid!</div>

Мы можем пойти еще дальше и задать атрибут `{style}` нашего div как объект JavaScript:

```ts
const name = 'Solid';
const style = {
    'background-color': '#2c4f7c',
    color: '#FFF',
};

<div style={style}>Hello {name}!</div>;
```

Благодаря JSX мы получаем код, позволяющий использовать динамическую природу JavaScript в сочетании с декларативным HTML-подобным синтаксисом.

По мере изучения этих уроков мы узнаем, как реактивная система Solid в сочетании с JSX улучшает работу по созданию пользовательских интерфейсов.

???vue "vue"

    Этот синтаксис связывания HTML и значений атрибутов - два разных синтаксиса в Vue. В Vue это может выглядеть следующим образом:

    ```html
    <div :style="style">Hello {{name}}!</div>
    ```

Давайте обновим наш исходный компонент `HelloWorld` с учетом того, что мы узнали в ходе изучения JSX:

```ts
export function HelloWorld() {
    const name = 'Solid';
    const style = {
        'background-color': '#2c4f7c',
        color: '#FFF',
    };
    return <div style={style}>Hello {name}!</div>;
}
```

### Использование фрагментов

Одно из правил при использовании JSX заключается в том, что компонентная функция должна возвращать один элемент. Например, следующий пример является правильным JSX, поскольку в нем есть единственный элемент `<div />`, который содержит остальную часть JSX:

```ts
<div>
    <h1>Welcome</h1>
    <HelloWorld />
</div>
```

Но следующий вариант - это _не_ корректный JSX, поскольку в нем нет ни одного содержащего элемента:

```ts
<h1>Welcome</h1>
<HelloWorld />
```

Однако, возможно, мы не хотим, чтобы при выводе кода в DOM его окружало `div`. _JSX-фрагменты_ дают нам возможность писать корректный JSX и при этом не выводить в DOM содержащий его элемент:

```ts
<>
    <h1>Welcome</h1>
    <HelloWorld />
</>
```

???vue "vue"

    Аналогичным образом работал и Vue 2: компонент должен был иметь один корневой элемент.

???angular "angular"

    Angular имеет схожий с Fragments API с [`ng-container`](https://angular.io/api/core/ng-container).

### JSX компилируется

JSX представляет собой синтаксис поверх JavaScript и не является непосредственной частью какого-либо стандарта JavaScript. Поэтому JSX не может быть запущен непосредственно в браузере (или в большинстве других программ выполнения JavaScript). Solid и другие фреймворки, использующие JSX, требуют, чтобы их код выполнялся через компилятор. Если вы используете стартовый шаблон, то вам не нужно беспокоиться об этом, все будет работать из коробки.

!!!note ""

    Помните, мы говорили о том, что имена функций компонентов приводятся в заголовке? Это сделано для того, чтобы компилятор понимал разницу между собственными элементами DOM (например, `div`) и пользовательскими компонентами (например, `HelloWorld`).

???note "Подробнее о компиляторе"

    JSX-компилятор Solid не просто компилирует JSX в JavaScript, он также извлекает реактивные значения (о чем мы поговорим позже) и делает все более эффективным.

    Это более сложная задача, чем JSX-компилятор React, но гораздо менее сложная, чем, например, компилятор Svelte. Компилятор Solid не трогает JavaScript, а только JSX.

    Если вам интересно узнать, как Solid и другие фреймворки используют компиляцию, посмотрите статью [A Look at Compilation in JavaScript Frameworks](https://dev.to/this-is-learning/a-look-at-compilation-in-javascript-frameworks-3caj), написанную создателем Solid.

## Монтирование Solid в DOM

Как наш компонент `App` попадает на реальную HTML-страницу, которую мы показываем нашим пользователям?

Чтобы добавить приложение Solid в HTML-документ, мы сначала выделяем в HTML содержащий его элемент и присваиваем ему `id`. Обычно это выглядит следующим образом:

```html
<body>
    <div id="root"></div>
</body>
```

Затем мы вызываем функцию Solid `render`, передавая в нее наш компонент `App` и данный элемент:

```js
render(() => <App />, document.getElementById('root'));
```

???vue "vue"

    Это аналогично функции Vue [`app.mount`](https://vuejs.org/guide/essentials/application.html#mounting-the-app).

Посмотрите, как это выглядит в нашем проекте в файле `index.html`, который создает div, и в файле `src/index.tsx`, который вызывает эту функцию `render`:

=== "src/HelloWorld.tsx"

    ```ts
    export function HelloWorld() {
    	const name = 'Solid';
    	const style = {
    		'background-color': '#2c4f7c',
    		color: '#FFF',
    	};
    	return <div style={style}>Hello {name}!</div>;
    }
    ```

=== "src/App.tsx"

    ```ts
    import { HelloWorld } from './HelloWorld';
    function App() {
    	return (
    		<div>
    			<h1>Welcome</h1>
    			<HelloWorld />
    		</div>
    	);
    }
    export default App;
    ```

=== "src/index.tsx"

    ```ts
    import { render } from 'solid-js/web';
    import './index.css';
    import App from './App';
    render(
    	() => <App />,
    	document.getElementById('root') as HTMLElement
    );
    ```

=== "index.html"

    ```html
    <!DOCTYPE html>
    <html lang="en">
    	<head>
    		<meta charset="utf-8" />
    		<meta
    			name="viewport"
    			content="width=device-width, initial-scale=1"
    		/>
    		<meta name="theme-color" content="#000000" />
    		<link
    			rel="shortcut icon"
    			type="image/ico"
    			href="/src/assets/favicon.ico"
    		/>
    		<title>Solid App</title>
    	</head>
    	<body>
    		<noscript
    			>You need to enable JavaScript to run this
    			app.</noscript
    		>
    		<div id="root"></div>
    		<script src="/src/index.tsx" type="module"></script>
    	</body>
    </html>
    ```

!!!note ""

    Необходимо убедиться, что первым аргументом, передаваемым функции `render`, является функция, возвращающая компонент `() => <Component />`, а не просто сам компонент.

???react "react"

    В React по умолчанию в корень приложения добавлен компонент `StrictMode`, который [изменяет поведение рендеринга кода в зависимости от того, находитесь ли вы в режиме разработки или нет](https://beta.reactjs.org/learn/keeping-components-pure#side-effects-unintended-consequences:~:text=Detecting%20impure%20calculations%20with%20StrictMode).

    В Solid нет таких различий между рендерингом в режиме разработки и в режиме производства.

## Объединение нескольких компонентов

В большинстве случаев наши приложения достаточно велики, чтобы мы захотели разделить их на _множество_ компонентов. Давайте перейдем от примера с `HelloWorld` к созданию чего-то более масштабного &mdash; виртуальной книжной полки. Наша книжная полка позволит нам добавлять книги, отмечать их как "прочитанные" и, в конечном счете, получать названия книг из других сервисов в Интернете.

Учитывая интуитивную природу JSX, мы можем немного спланировать наше приложение. Наша главная `Bookshelf` будет состоять из двух компонентов верхнего уровня:

-   компонент `<BookList />`, в котором перечислены все книги, имеющиеся на нашей книжной полке
-   компонент `<AddBook />`, который позволит нам добавить книгу на книжную полку.

Хотя со временем мы захотим сделать нашу книжную полку интерактивной, давайте сначала создадим неинтерактивные версии каждого компонента. Во-первых, мы можем сделать компонент `BookList` внутренним и перечислить несколько книг вместе с их авторами:

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

Далее мы создаем компонент `AddBook`. Это будет форма, которая позволит нам добавить книгу в список `BookList`.

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

Finally, we can _compose_ our `BookList` and `AddBook` components together. Let's create a `Bookshelf` component within `App.tsx`.

=== "App.tsx"

    ```ts
    import { BookList } from './BookList';
    import { AddBook } from './AddBook';
    function Bookshelf() {
    	return (
    		<div>
    			<h1>My Bookshelf</h1>
    			<BookList />
    			<AddBook />
    		</div>
    	);
    }
    function App() {
    	return <Bookshelf />;
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

### Отношения между компонентами

Если компонент включает в себя другой компонент, мы называем его _родительским_ компонентом. В нашем примере `Bookshelf` компонент `Bookshelf` является _родительским_ для `BookList` и `AddBook`, поскольку эти два компонента находятся внутри `Bookshelf`. Соответственно, `BookList` и `AddBook` являются _дочерними_ компонентами `Bookshelf`. Возможно, вы уже знакомы с этой терминологией; так мы обычно обозначаем отношения между HTML-элементами в нашем DOM-дереве.

### Передача свойств между компонентами

Компоненты могут взаимодействовать со своими дочерними компонентами путем передачи свойств, часто сокращенно называемых `props`. Чтобы продемонстрировать возможности свойства `props`, давайте настроим книжную полку, добавив ей имя. Для этого добавьте в функцию bookshelf параметр `props`. Затем добавим атрибут `name` при использовании компонента:

=== "App.tsx\*"

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

Функции компонентов принимают только один аргумент, если таковой имеется: объект свойства. Свойства этого объекта устанавливаются при использовании компонента и предоставляют атрибуты.

???react "react"

    В React при обращении к свойствам внутри компонента принято использовать _деструктурирующее присваивание_. Например, наш компонент `Bookshelf` может быть написан в React следующим образом:

    ```ts
    function Bookshelf({ name }) {
    	return (
    		<div>
    			<h1>{name}'s Bookshelf</h1>
    			<Books />
    			<AddBook />
    		</div>
    	);
    }
    ```

    **Но деструктуризация свойств обычно является плохой идеей в Solid**. Под капотом Solid использует _прокси_ для хука объектов `свойств`, чтобы знать, когда к ним обращаются. Когда мы деструктурируем наш объект свойства в сигнатуре функции, мы немедленно получаем доступ к свойствам объекта и теряем реактивность.

### Прогресс!

Итак, мы познакомились с компонентами Solid, JSX и создали основу для удобного приложения "Книжная полка". Далее мы воплотим нашу книжную полку в жизнь, используя реактивную систему Solid!

## Ссылки

-   [Building User Interfaces with Components](https://docs.solidjs.com/guides/tutorials/getting-started-with-solid/building-ui-with-components)
