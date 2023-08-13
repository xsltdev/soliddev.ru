---
description: WindiCSS - это утилитарная CSS-библиотека, создаваемая по требованию пользователя. WindiCSS интегрируется с Solid как плагин Vite
---

# WindiCSS

**WindiCSS** - это утилитарная CSS-библиотека, создаваемая по требованию пользователя. WindiCSS интегрируется с Solid как плагин Vite.

## Установить плагин Vite

```sh
# Install (choose one)
npm i --save-dev vite-plugin-windicss windicss
pnpm i --dev vite-plugin-windicss windicss   # Using pnpm
yarn add --dev vite-plugin-windicss windicss # Using yarn
```

## Создание конфигурации

WindiCSS - это инструмент, основанный на конфигурации. Создайте файл `.windi.config.ts` в корне каталога проекта. Он должен выглядеть примерно так:

```ts
// windi.config.ts
import { defineConfig } from 'windicss/helpers';
import formsPlugin from 'windicss/plugin/forms';

export default defineConfig({
    darkMode: 'class',
    safelist: 'p-3 p-4 p-5',
    theme: {
        extend: {
            colors: {
                teal: {
                    100: '#096',
                },
            },
        },
    },
    plugins: [formsPlugin],
});
```

## Импорт плагина Vite

После установки откройте файл `vite.config.js` или `vite.config.ts`. Стартовая конфигурация Solid Vite выглядит следующим образом:

```js
import { defineConfig } from 'vite';

import solidPlugin from 'vite-plugin-solid';

export default defineConfig({
    plugins: [solidPlugin()],
    server: {
        port: 3000,
    },
    build: {
        target: 'esnext',
    },
});
```

Теперь `import WindiCSS from "vite-plugin-windicss"` и вызываем его как функцию внутри плагинов:

```diff
import { defineConfig } from "vite"

+ import WindiCSS from "vite-plugin-windicss"
import solidPlugin from "vite-plugin-solid"

export default defineConfig({
	plugins: [
+		WindiCSS(),
		solidPlugin(),
	],
	server: {
		port: 3000,
	},
	build: {
		target: "esnext",
	},
})
```

Обратите внимание, что `WindiCSS` должен быть упорядочен перед `SolidPlugin`. Это предотвращает некоторые крайние случаи, такие как `&`, от экранирования как `&amp;`.

## Импорт WindiCSS

Добавьте `import "virtual:windi.css"` в `index.jsx` или `index.tsx`. `virtual` просто сообщает, что в файловой системе имеется `windi.css`.

```diff
/* @refresh reload */
+ import "virtual:windi.css"

import { render } from 'solid-js/web';

import './index.css';
import App from './App';

render(() => <App />, document.getElementById('root') as HTMLElement);
```

## Поддержка

Для получения дополнительной поддержки смотрите [Руководство по интеграции WindiCSS/Vite](https://windicss.org/integrations/vite.html) или присоединяйтесь к [официальным каналам WindiCSS](https://discord.com/invite/GTKYWq9zgA) и [Solid JS](https://discord.com/invite/solidjs) Discord. 👋

## Ссылки

-   [WindiCSS](https://docs.solidjs.com/guides/how-to-guides/styling-in-solid/windicss)
