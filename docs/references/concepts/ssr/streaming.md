---
description: Потоковая передача - это техника рендеринга на стороне сервера, аналогичная Simple server-side rendering, только с использованием потоков
---

# Потоковая передача

!!!note ""

    Это низкоуровневый API, предназначенный для использования авторами библиотек. Если вы хотите использовать рендеринг на стороне сервера в своем приложении, мы предлагаем воспользоваться [Solid Start](https://start.solidjs.com/getting-started/what-is-solidstart), нашим метафреймворком, который позволяет очень просто добавить рендеринг на стороне сервера в ваше приложение.

## Что такое стриминг?

Потоковая передача - это техника рендеринга на стороне сервера, аналогичная [Simple server-side rendering](simple-client-fetching-ssr.md), только с использованием потоков. Данные загружаются на сервер и передаются по потоку, чтобы клиент мог их отрисовать.

В отличие от Async SSR в этой технике гидрируется только начальный HTML. Загрузка данных происходит несколько быстрее, что позволяет сократить время загрузки страницы. Это особенно заметно по сравнению с Async SSR при использовании на медленных серверах.

## Как работает `renderToStream`?

`renderToStream` работает, принимая компонент и возвращая HTML, однако для любого узла, где требуются асинхронные данные, вместо него отрисовывается заполнитель, который заменяется по мере загрузки потока. Таким образом, в потоковом режиме границы суспензий отображаются, а затем заменяются, в отличие от Async SSR, где границы суспензий не отображаются вообще.

```jsx
import { renderToStream } from 'solid-js/web';
import { createResource, Suspense } from 'solid-js';
import List from './MyListComponent';

const App = () => {
    const [userData] = createResource(getUserDataAsync);
    const [userPosts] = createResource(getUserPostsAsync);

    return (
        <div>
            <h1>Streaming</h1>
            <Suspense fallback={<p>Loading...</p>}>
                <h2>{userData().username}</h2>
                <Suspense
                    fallback={<p>Loading posts...</p>}
                >
                    <List posts={userPosts()} />
                </Suspense>
            </Suspense>
        </div>
    );
};

renderToStream(() => <App />);
```

В приведенном выше примере грубого кода сначала будут отрисовываться загрузочные плейсхолдеры, а затем, по мере загрузки потока, эти узлы будут заменяться. Вот пошаговое описание того, как это может выглядеть:

```html
<div>
    <h1>Streaming</h1>
    <p>Loading...</p>
</div>
```

---

```html
<div>
    <h1>Streaming</h1>
    <h2>johndoe</h2>
    <p>Loading posts...</p>
</div>
```

---

```html
<div>
    <h1>Streaming</h1>
    <h2>johndoe</h2>
    <ul>
        <li><a href="/post/1">first post</a></li>
        <li><a href="/post/2">second post</a></li>
        <li><a href="/post/3">third post</a></li>
        <li><a href="/post/4">fourth post</a></li>
    </ul>
</div>
```

## Как использовать `renderToStream`?

Для использования `renderToStream` вам потребуется сервер, на котором может выполняться Javascript-код. Вы можете использовать любой серверный фреймворк, но для данного примера мы будем использовать [Express](https://expressjs.com/).

```js
// server.js
import express from 'express';
import { renderToStringAsync } from 'solid-js/web';
import App from './App';

const app = express();

app.get('*', (res, req) => {
    return renderToStream(() => <App />).pipe(res);
});

app.listen(8080, () =>
    console.log('App is listening on port 8080')
);
```

---

```jsx
// App.jsx
import { createResource } from 'solid-js';

export default function App() {
    const [data] = createResource(() =>
        fetch(
            'https://jsonplaceholder.typicode.com/todos/1'
        ).then((res) => res.json())
    );

    return (
        <div>
            <h1>Async SSR</h1>
            <p>{data().title}</p>
        </div>
    );
}
```

!!!note ""

    Следует иметь в виду, что для сборки приложения потребуется пакетный компоновщик, например Vite, Webpack или Rollup. Вот полнофункциональный пример с использованием rollup и express. [solid-ssr-workbench](https://github.com/ryansolid/solid-ssr-workbench). Это репозиторий содержит примеры использования Solid со всеми тремя формами SSR.

## Ссылки

-   [Streaming](https://docs.solidjs.com/references/concepts/ssr/streaming)
