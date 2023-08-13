---
description: Tailwind CSS - это утилитарная CSS-библиотека, создаваемая по запросу. Tailwind CSS интегрируется с Solid как встроенный плагин PostCSS
---

# Tailwind CSS

**Tailwind CSS** - это утилитарная CSS-библиотека, создаваемая по запросу. Tailwind CSS интегрируется с Solid как встроенный плагин PostCSS.

## Установить Tailwind CSS

```sh
# Install (choose one)
npm i --save-dev tailwindcss postcss autoprefixer
pnpm i --save-dev tailwindcss postcss autoprefixer   # Using pnpm
yarn add --dev tailwindcss postcss autoprefixer # Using yarn

# Initialize
npx tailwindcss init -p
```

## Создание конфигурации

Tailwind CSS - это инструмент, основанный на конфигурации. Используйте команду initialize, приведенную выше, или создайте `tailwind.config.js` в корне каталога вашего проекта. Он должен выглядеть примерно так:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
    theme: {
        extend: {},
    },
    plugins: [],
};
```

Более подробную информацию о конфигурации можно найти в официальной документации: [Tailwind Official Documentation](https://tailwindcss.com/docs/guides/solidjs).

## Добавление директив Tailwind

Tailwind состоит из трех слоев: базового слоя, слоя компонентов и слоя утилит. Добавьте эти строки кода в ваш файл `src/index.css`:

```diff
# src/index.css

+ @tailwind base;
+ @tailwind components;
+ @tailwind utilities;
```

Это подсказка для PostCSS, что мы используем Tailwind, и передача Tailwind информации о том, какие директивы мы используем и каков их порядок. Если вы не знаете, что делаете, то, скорее всего, не стоит изменять этот код. Но вы все равно можете добавить пользовательский CSS ниже этих директив. Обратите внимание, что этот файл будет скомпилирован как PostCSS.

## Импорт Tailwind CSS

Убедитесь, что `index.css` импортирован в ваш корневой файл `index.jsx` или `index.tsx`. Если это не так, добавьте `import "./index.css"` в `index.jsx` или `index.tsx`:

```diff
# src/index.jsx

import { render } from 'solid-js/web'; import App from './App';
+ import "./index.css"

render(() => <App />, document.getElementById('root') as HTMLElement);
```

### Использование

Теперь, когда мы настроили TailwindCSS, мы можем избавиться от стилей в файле `Card.css` или просто избавиться от файла полностью.

```diff
/* src/components/Card.css */

- .grid { display: grid; }
- .grid.grid-center { place-items: center; }
- .screen { min-height: 100vh; }
-
- .card {
-   height: 160px;
-   aspect-ratio: 2;
-   border-radius: 16px;
-   background-color: white;
-   box-shadow: 0 0 0 4px hsl(0 0% 0% / 15%);
- }
```

Не забудьте удалить импорт `Card.css` из всех компонентов, в которые он может быть импортирован, и используйте классы стилей, предоставляемые TailwindCSS.

```diff
/* src/components/Card.jsx */

- import "./Card.css"

function Card() {
  return (
    <>
+     <div class="grid place-items-center min-h-screen">
+       <div class="h-[160px] aspect aspect-[2] rounded-[16px] shadow-[0_0_0_4px_hsl(0_0%_0%_/_15%)]">Hello, world!</div>
      </div>
    </>
  );
}
```

## Поддержка

Для получения дополнительной поддержки см. руководство [Taiwind CSS/Vite integration guide](https://tailwindcss.com/docs/guides/vite) или присоединяйтесь к [официальным каналам Tailwind CSS](https://discord.com/invite/7NF8GNe) и [Solid JS](https://discord.com/invite/solidjs) Discord. 👋

## Ссылки

-   [Tailwind CSS](https://docs.solidjs.com/guides/how-to-guides/styling-in-solid/tailwind-css)
