# Установка и настройка

Здесь мы приведем краткое руководство по настройке и установке приложения Solid. Мы также затронем такие вопросы, как настройка переменных окружения, Typescript и линтеров.

## Установка

Solid имеет несколько [vite-шаблонов](https://github.com/solidjs/templates) для начала работы. Чтобы воспользоваться ими, просто выполните следующие команды:

=== "Javascript"

    ```bash
    > npx degit solidjs/templates/js my-app
    > cd my-app
    > npm install # yarn or pnpm install
    # To run the server
    > npm run dev
    ```

=== "Typescript"

    ```bash
    > npx degit solidjs/templates/ts my-app
    > cd my-app
    > npm install # yarn or pnpm install
    # To run the server
    > npm run dev
    ```

### Typescript с TailwindCSS, WindiCSS, SASS или UNOCSS

В приведенной ниже команде можно заменить `tailwindcss` на `windicss`, `sass` или `unocss`, чтобы получить шаблон, в котором уже настроен один из них.

```bash
> npx degit solidjs/templates/ts-tailwindcss my-app
> cd my-app
> npm install # yarn or pnpm install
# To run the server
> npm run dev
```

!!!note ""

    Для получения дополнительных шаблонов посетите наш репозиторий [vite templates GitHub repository](https://github.com/solidjs/templates).

## Установка Typescript в уже существующие проекты Solid Javascript

Здесь мы расскажем о том, как установить и настроить Typescript в уже существующем проекте Solid Javascript.

**Шаг 1:** Установите `typescript` в наш проект

```bash
> npm install --save-dev typescript
```

**Шаг 2:** Инициализация Typescript с помощью файла `tsconfig.json`.

Вы можете создать файл `tsconfig.json` или выполнить приведенную ниже команду, чтобы он был сгенерирован автоматически

```bash
> tsc --init
```

**Шаг 3:** Отредактируйте файл `tsconfig.json` в соответствии с конфигурацией Solid.

Скопируйте приведенный ниже код в файл `tsconfig.json`, так он должен выглядеть.

```json
{
    "compilerOptions": {
        "strict": true,
        "target": "ESNext",
        "module": "ESNext",
        "moduleResolution": "node",
        "allowSyntheticDefaultImports": true,
        "esModuleInterop": true,
        "jsx": "preserve",
        "jsxImportSource": "solid-js",
        "types": ["vite/client"],
        "noEmit": true,
        "isolatedModules": true
    }
}
```

Если вы использовали `tsc --init`, то ваш `tsconfig.json` должен содержать много шаблонов, поэтому вы можете избавиться от них и просто скопировать приведенный выше фрагмент.

**Заключительный шаг:** Создайте файл Typescript или `.tsx`, чтобы проверить, все ли работает так, как нужно.

!!!warning "Примечание"

    Если вы хотите заменить файл `index.jsx` на `index.tsx`, то сначала необходимо сделать две вещи. Необходимо убедиться, что вы изменили атрибут `src` для тега `script` в файле `index.html`.

```diff
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <link rel="shortcut icon" type="image/ico" href="/src/assets/favicon.ico" />
    <title>Solid App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>

+   <script src="/src/index.tsx" type="module"></script>
  </body>
</html>
```

```ts
// Mytscomponent.tsx

import { Component } from 'solid-js';

function MyTsComponent(): Component {
    return (
        <div>
            <h1> This is a typescript component </h1>
        </div>
    );
}

export default MyTsComponent;
```

Давайте используем наш компонент Typescript в нашем компоненте Javascript

```js
// MyJsComponent.jsx

import MyTsComponent from './MyTsComponent';

function MyJsComponent() {
    return (
        <>
            ...
            <MyTsComponent />
        </>
    );
}
```

## Настройка переменных окружения

Solid использует [Vite](https://vitejs.dev/), поэтому использование переменных окружения здесь будет сильно отличаться от использования переменных окружения в других фреймворках, использующих другие встроенные средства.

**Шаг 1:** Создадим файл `.env` и объявим в нем значение `VITE_VARIABLE_NAME`.

```
VITE_VARIABLE_NAME="I am a vite environment variable"
```

**Шаг 2:** Используйте переменную окружения в приложении Solid.

Попробуем использовать переменную окружения, которую мы только что создали, в одном из компонентов Solid

```js
function MyComponent() {
    return (
        <div>
            <h2>
                Component with environment variable used{' '}
                {import.meta.env.VITE_VARIABLE_NAME}
            </h2>
        </div>
    );
}

export default MyComponent;
```

Вот и все, всего два шага, и вы готовы к работе с переменными `.env`.

Обратите внимание, как мы использовали переменную окружения в нашем компоненте Solid, вместо привычного для многих разработчиков `process.env` мы используем

```js
import.meta.env;
```

что характерно для Vite.

!!!note "Примечание"

    Префикс `VITE` используется на стороне клиента, поэтому переменные окружения, специфичные для бэкенда, не должны использовать этот префикс, поскольку они не связаны с `VITE` для использования на стороне клиента.

Более подробную информацию о переменных окружения в Vite и о том, как использовать intellisense для переменных окружения в Typescript, можно найти в [Vite Documentation](https://vitejs.dev/guide/env-and-mode.html#env-files).

## Ссылки

-   [Installation & Setup](https://docs.solidjs.com/guides/how-to-guides/get-ready-for-solid/installation-and-setup)
