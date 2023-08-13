---
description: UnoCSS - это утилитарная CSS-библиотека, создаваемая по требованию пользователя. UnoCSS интегрируется с Solid как плагин Vite
---

# UnoCSS

**UnoCSS** - это утилитарная CSS-библиотека, создаваемая по требованию пользователя. UnoCSS интегрируется с Solid как плагин Vite.

## Установить плагин Vite

```sh
npm i --save-dev unocss
pnpm i --dev unocss   # Using pnpm
yarn add --dev unocss # Using yarn
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

Теперь `import unocssPlugin from "unocss/vite"` и вызываем его как функцию внутри плагинов:

```diff
import { defineConfig } from "vite"

+ import unocssPlugin from "unocss/vite"
import solidPlugin from "vite-plugin-solid"

export default defineConfig({
	plugins: [
+		unocssPlugin(),
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

Обратите внимание, что `unocssPlugin` должен быть упорядочен перед `solidPlugin`. Это предотвращает некоторые крайние случаи, такие как `&`, от экранирования как `&amp;`.

## Импорт UnoCSS

Добавьте `import "uno.css"` в `index.jsx` или `index.tsx`:

```diff
/* @refresh reload */
+ import "uno.css"

import { render } from 'solid-js/web';

import './index.css';
import App from './App';

render(() => <App />, document.getElementById('root') as HTMLElement);
```

Можно также использовать псевдоним `import "virtual:uno.css"`; это эквивалентно `import "uno.css"`. Псевдоним `virtual` просто сообщает, что в файловой системе есть файл `uno.css`.

```diff
/* @refresh reload */
+ import "virtual:uno.css"

import { render } from 'solid-js/web';

import './index.css';
import App from './App';

render(() => <App />, document.getElementById('root') as HTMLElement);
```

## Поддержка

Для получения дополнительной поддержки см. руководство [UnoCSS/Vite integration guide](https://github.com/unocss/unocss/tree/main/packages/vite) или присоединяйтесь к [официальному обсуждению UnoCSS на Github](https://github.com/unocss/unocss/discussions) и Discord-каналу [Solid JS](https://discord.com/invite/solidjs). 👋

## Ссылки

-   [UnoCSS](https://docs.solidjs.com/guides/how-to-guides/styling-in-solid/unocss)
