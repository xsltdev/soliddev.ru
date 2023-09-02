---
description: В этом разделе мы поговорим о генерации статических сайтов и о том, как ее можно реализовать в Solid
---

# Генерация статических сайтов (SSG)

!!!note ""

    Это низкоуровневый API, предназначенный для использования авторами библиотек. Если вы хотите использовать рендеринг на стороне сервера в своем приложении, мы предлагаем воспользоваться [Solid Start](https://start.solidjs.com/getting-started/what-is-solidstart), нашим метафреймворком, который позволяет очень просто добавить рендеринг на стороне сервера в ваше приложение.

В этом разделе мы поговорим о генерации статических сайтов и о том, как ее можно реализовать в Solid.

## Что такое статическая генерация сайта?

Генерация статических сайтов - это процесс создания статического (никогда не меняющегося) HTML-сайта на основе исходных данных с помощью таких фреймворков веб-разработки, как Solid.

Статические сайты состоят из HTML, который загружается каждый раз одинаково, в отличие от динамических сайтов, где данные на сайте могут меняться в зависимости от ввода пользователя или изменений в получаемых данных.

## Как Solid реализует SSG?

Solid использует функцию [`renderToStringAsync`](async-ssr.md) для загрузки всех асинхронных данных и генерации необходимого HTML.

Возможно, вы зададитесь вопросом, чем же это отличается от Async SSR? Ключевое отличие от Async SSR заключается в том, что в Async SSR сервер запускает приложение Solid, а в SSG приложение Solid компилируется и сохраняет результат заранее. Это означает, что код Solid больше не трогается, кроме как во время сборки и обновления.

Возможно, это звучит несколько туманно и сложно для понимания, поэтому вот пример того, как это выглядит в действии.

## Как использовать SSG в Solid?

!!!note ""

    Данный процесс реализации SSG в Solid является более сложным по сравнению с реализацией других форм SSR в Solid, поэтому внимательно следите за его выполнением

Для того чтобы использовать SSG в Solid, необходимо иметь компилятор, который помогает компилировать JS-код. В данном примере мы будем использовать Rollup.

```js
// rollup.config.js
import path from 'path';
import nodeResolve from '@rollup/plugin-node-resolve';
import babel from '@rollup/plugin-babel';
import json from '@rollup/plugin-json';
import copy from 'rollup-plugin-copy';
import { terser } from 'rollup-plugin-terser';
import manifest from 'rollup-route-manifest';

export default [
    {
        input: 'App.jsx',
        output: [
            {
                dir: 'public/js',
                format: 'esm',
            },
        ],
        preserveEntrySignatures: false,
        plugins: [
            nodeResolve({
                exportConditions: ['solid'],
                extensions: ['.js', '.jsx', '.ts', '.tsx'],
            }),
            babel({
                babelHelpers: 'bundled',
                presets: [
                    [
                        'solid',
                        {
                            generate: 'dom',
                            hydratable: true,
                        },
                    ],
                ],
            }),
            terser(),
        ],
    },
    {
        input: 'index.js',
        output: [
            {
                dir: 'lib',
                exports: 'auto',
                format: 'cjs',
            },
        ],
        external: [
            'solid-js',
            'solid-js/web',
            'node-fetch',
        ],
        plugins: [
            nodeResolve({
                preferBuiltins: true,
                exportConditions: ['solid'],
                extensions: ['.js', '.jsx', '.ts', '.tsx'],
            }),
            babel({
                babelHelpers: 'bundled',
                presets: [
                    [
                        'solid',
                        {
                            generate: 'ssr',
                            hydratable: true,
                            async: true,
                        },
                    ],
                ],
            }),
            json(),
        ],
    },
];
```

В приведенном выше коде мы настраиваем rollup для помощи в компиляции нашего кода, превращая все имеющиеся у нас jsx в скомпилированный javascript, который впоследствии мы сможем запустить с помощью `node`.

Далее создадим наш `App.jsx`.

```jsx
// App.jsx
export default function App() {
    return (
        <div>
            <h1>This is a header</h1>
            <p>This is a simple description</p>
        </div>
    );
}
```

Выше представлен простой компонент. При желании можно использовать асинхронные данные, так как мы будем использовать функцию `renderToStringAsync`, все асинхронные данные будут получены и помещены на место до создания сайта.

Теперь мы создадим файл `index.js`, который rollup скомпилирует и превратит в файл CommonJS со всем jsx- и tsx-кодом, преобразованным, скомпилированным и готовым к запуску.

```js
// index.js
import fetch from 'node-fetch';
import { renderToStringAsync } from 'solid-js/web';
import App from './App';

globalThis.fetch = fetch;

// entry point for server render
export default (req) => {
    return renderToStringAsync(() => <App />);
};
```

В приведенном выше коде мы принимаем наш компонент Solid и преобразуем его в строку с помощью функции `renderToStringAsync`.

В заключение мы создадим скрипт, который поможет нам генерировать и организовывать наши статические файлы.

```js
// generate.js
const path = require('path');
const renderStatic = require('solid-ssr/static');

const PAGES = ['index'];
const pathToServer = path.resolve(
    __dirname,
    'lib/index.js'
);
const pathToPublic = path.resolve(__dirname, 'public');

renderStatic(
    PAGES.map((p) => ({
        entry: pathToServer,
        output: path.join(pathToPublic, `${p}.html`),
        url: p === 'index' ? `/` : `/${p}`,
    }))
);
```

Это поможет нам сгенерировать статические файлы с помощью функции `renderStatic`, предоставляемой пакетом `solid-ssr`.

Теперь осталось скомпилировать наш код, запустить скрипт и обслужить наш статический сайт.

```bash
npx rollup -c rollup.config.js && node generate.js #compile and generate the site
npx serve public -l 8080 #serve the site
```

!!!note ""

    Более подробную информацию о SSG и SSR в Solid можно найти в репо [solid-ssr-workbench](https://github.com/ryansolid/solid-ssr-workbench) и репо [solid-ssr](https://github.com/solidjs/solid/tree/main/packages/solid-ssr).

## Ограничения и преимущества SSG

### Ограничения

Основным ограничением SSG в целом является то, что он полностью статичен, т.е. взаимодействия с пользователем практически не будет, а обновление данных сайта возможно только при его полной перестройке. Поэтому рекомендуется использовать статическую генерацию сайта только в тех случаях, когда пользователь не будет взаимодействовать с сайтом или его взаимодействие будет минимальным.

### Преимущества

Основное преимущество SSG заключается в том, что статические сайты загружаются значительно быстрее, чем динамические, благодаря тому, что на них не используется javascript, а есть только HTML и CSS. Другим преимуществом является то, что вам не нужно беспокоиться о наполнении и индексации поисковыми системами, поскольку нет необходимости запускать javascript, и ваш сайт каждый раз отображается одинаково, с одними и теми же данными, что делает его чрезвычайно удобным для блогов.

## Ссылки

-   [Static Site Generation (SSG)](https://docs.solidjs.com/references/concepts/ssr/ssg)
