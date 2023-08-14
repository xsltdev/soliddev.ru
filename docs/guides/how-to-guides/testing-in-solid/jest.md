---
description: Jest - это многофункциональный интегрированный набор модульных тестов, написанный компанией Facebook, включающий программу запуска тестов, поддержку различных окружений, расширяемую библиотеку утверждений, средства мокинга и измерения покрытия
---

# Jest

**Jest** - это многофункциональный интегрированный набор модульных тестов, написанный компанией Facebook, включающий программу запуска тестов, поддержку различных окружений, расширяемую библиотеку утверждений, средства мокинга и измерения покрытия.

!!!tip "Стартовый шаблон"

    В официальном каталоге [vite starter templates](../get-ready-for-solid/installation-and-setup.md) также имеется шаблон `ts-jest`, в котором уже установлен Jest и большинство других требований:

    ```bash
    npx degit solidjs/templates/ts-jest my-solid-project
    cd my-solid-project
    npm install # or pnpm install or yarn install
    ```

    Если он установлен, продолжите расширенную установку.

## Установка

Сначала нам необходимо установить пакет `jest`:

```bash
npm install --save-dev jest
pnpm i --dev jest           # Using pnpm
yarn add --dev jest         # Using yarn
```

## Настройка

Чтобы протестировать код Solid с помощью Jest, необходимо его настроить. Настройку можно выполнить в поле `"jest"` файла `package.json` или в отдельном файле `jest.config.js`. Последний позволяет иметь различные настройки, тестировать код как на стороне клиента, так и на стороне сервера, а также программно дополнять существующие пресеты.

На момент написания статьи Jest официально поддерживает непосредственно только формат модулей CJS и использует конвейер трансформации для работы с другими форматами и транспилированным JS. Поскольку Solid использует транспонированный код, нам необходимо его настроить.

Поскольку Jest запускает весь импортируемый код в формате CJS, нам нужно либо преобразовать файлы, которые не являются корректным CJS, либо заставить Jest игнорировать их.

### Преобразование кода Solid

Существует несколько способов преобразования Solid-кода: наиболее предпочтительным является пресет под названием `solid-jest`, содержащий конфигурацию для преобразования Solid JSX в CJS. Сначала нам необходимо установить его, если мы не хотим использовать `ts-jest` для преобразования TypeScript (см. ниже):

```bash
npm install --save-dev solid-jest
pnpm i --dev solid-jest           # Using pnpm
yarn add --dev solid-jest         # Using yarn
```

Чтобы добавить его в нашу конфигурацию Jest, необходимо в свойство `preset` конфигурации (в файле `package.json#jest` или `jest.config.js`) добавить правильный экспорт для тестирования на стороне клиента или на стороне сервера:

```js
{ "preset": "solid-jest/preset/browser" }
// or
{ "preset": "solid-jest/preset/node" }
```

### TypeScript преобразование кода

#### Альтернатива 1: Использование babel для преобразования TypeScript

Если мы используем babel для преобразования TypeScript, нам необходимо установить `@babel/preset-typescript`:

```bash
npm install --save-dev @babel/preset-typescript
pnpm i --dev @babel/preset-typescript           # Using pnpm
yarn add --dev @babel/preset-typescript         # Using yarn
```

Также необходимо настроить конфигурацию babel, если ее еще нет:

```bash
cat > .babelrc <<- EOF
{
  "presets": [
    "@babel/preset-env",
    "babel-preset-solid",
    "@babel/preset-typescript"
  ]
}
EOF
```

### Альтернатива 2: Использование ts-jest для преобразования TypeScript

Для этого не требуется `solid-jest. Вместо того чтобы использовать только babel-трансформер, мы можем использовать компилятор TypeScript для преобразования тестов. Это означает, что перед выполнением тесты будут статически проверены на соответствие типам. Хотя это и снижает скорость выполнения тестов, но может привести к обнаружению ошибок в коде тестов.

Для использования этого компилятора его необходимо установить:

```bash
npm install --save-dev ts-jest
pnpm i --dev ts-jest           # Using pnpm
yarn add --dev ts-jest         # Using yarn
```

Затем необходимо настроить Jest на его использование в файле `package.json#jest` или `jest.config.js`:

```js
{
  "preset": "ts-jest",
  "globals": {
    "ts-jest": {
      "tsconfig": "tsconfig.json",
      "babelConfig": {
        "presets": [
          "babel-preset-solid",
          "@babel/preset-env"
        ]
      }
    }
  }
}
```

Если мы хотим протестировать код так, как он будет работать в браузере, нам также необходимо присвоить псевдоним solid-модулям правильной версии, поскольку `ts-jest`, к сожалению, не может определить браузерный режим. Добавьте к предыдущей конфигурации следующее:

```js
{
  "moduleNameMapper": {
    "solid-js/web": "<rootDir>/node_modules/solid-js/web/dist/web.cjs",
    "solid-js": "<rootDir>/node_modules/solid-js/dist/solid.cjs"
  }
}
```

### Игнорирование некоторых файлов

Как уже говорилось, Jest будет рассматривать все импортированное (включая CSS, изображения) как CJS. Это может привести к ошибкам. Чтобы заставить Jest игнорировать их, создайте пустой файл:

```bash
touch .empty.js
```

А затем создайте псевдоним для любого файла, который может привести к ошибкам в конфигурации Jest:

```js
{
  "moduleNameMapper": {
    ".*\\.(css|jpe?g|png|svg)$": "<rootDir>/.empty.js"
  }
}
```

Однако это может изменить поведение вашего кода. Если вам нужно проверить наличие имен классов HTML, предоставляемых модулями CSS, это не сработает.

### Другие преобразования

Существует несколько преобразований для других типов файлов.

-   CSS-модули: `jest-css-modules-transform`.
-   SCSS: `jest-scss-transform`.
-   Импорт пути для обычных файлов: `jest-file-loader`.
-   JSON5: `@talabes/json5-jest`
-   GraphQL: `@graphql-tools/jest-transform`.
-   Web Worker: `jest-webworker`.
-   подробнее о [npmjs](https://www.npmjs.com/search?q=jest+transform)

Если поиск подходящей трансформации не дал результата, то решением может стать пользовательская трансформация.

Мы можем просто установить их пакеты, как и раньше, только замените `[transform-module]` на указанное выше имя модуля:

```bash
npm install --save-dev [transform-module]
pnpm i --dev [transform-module]       # Using pnpm
yarn add --dev [transform-module]     # Using yarn
```

Самый простой способ их использования - дополнить существующий пресет в файле `jest.config.js`. В противном случае необходимо вручную объединить содержимое пресета и дополнительные преобразования:

```js
// jest.config.js
const jestConfig = require('solid-jest/preset/browser');
jestConfig.transform['\\.module\\.css$'] =
    'jest-css-modules-transform';
modules.export = jestConfig;
```

В случае если мы хотим придерживаться конфигурации в файле `package.json#jest`, нам необходимо скопировать в объявления трансформаций включенные трансформации для файлов `.jsx` и `.tsx`:

```js
{
  "transform": {
    "\\.[jt]s$": "babel-jest",
    "\\.module\\.css$": "jest-css-modules-transform"
  }
}
```

## Окружение

Solid.js может работать как в браузере, так и на сервере. По умолчанию Jest тестирует все в окружении node. Чтобы протестировать браузерный код с помощью Jest без реального браузера, нам необходимо DOM-окружение. Стандартом де-факто является `jsdom`. Чтобы использовать его, необходимо установить его вместе со средой Jest:

```bash
npm install --save-dev jest-environment-jsdom
pnpm i --dev jest-environment-jsdom           # Using pnpm
yarn add --dev jest-environment-jsdom         # Using yarn
```

Также необходимо добавить окружение в конфигурацию Jest в файле `package.json#jest` или `jest.config.js`:

```js
{ "testEnvironment": "jsdom" }
```

## Запуск тестов

Наконец, мы можем добавить в package.json следующий скрипт для вызова интерфейса командной строки Jest:

```js
{
  "scripts": {
    "test": "jest"
  }
}
```

Если наш тест работает, то вывод выглядит примерно так:

```bash
> jest

 PASS  src/testing.test.tsx
  ✓ changes text on click (53 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        4.012 s
Ran all test suites.
```

## Тестирование кода с помощью testing-library

Бесспорным золотым стандартом тестирования компонентов на сегодняшний день является библиотека Testing Library Кента Доддса, которая изначально была написана для React, но существует множество портов для других библиотек, и Solid не является исключением.

### Установка testing-library

Для тестирования с помощью `@solidjs/testing-library` нам сначала нужно установить ее:

```bash
npm install --save-dev @solidjs/testing-library
pnpm i --dev @solidjs/testing-library      # Using pnpm
yarn add --dev @solidjs/testing-library    # Using yarn
```

Testing-library также предоставляет небольшое полезное расширение для встроенных утверждений jest под названием `@testing-library/jest-dom`, которое мы также можем установить:

```bash
npm install --save-dev @testing-library/jest-dom
pnpm i --dev @testing-library/jest-dom     # Using pnpm
yarn add --dev @testing-library/jest-dom   # Using yarn
```

Теперь нам нужно запустить его перед тестом, чтобы мы могли использовать расширенные утверждения. Мы можем либо включить его вручную в наши тесты, либо, что более удобно, включить его в `setupFiles` нашей конфигурации Jest:

```js
{
  "setupFiles": [
    "node_modules/@testing-library/jest-dom/extend-expect"
  ]
}
```

Есть еще одно полезное дополнение от Testing Library для тестирования пользовательских событий: `@testing-library/user-events`. Оно пытается воспроизвести реальные события, которые обычно происходят в ситуации с пользователем, например, он фокусирует поля ввода перед тем, как набрать в них текст. Если мы хотим протестировать сложную интерактивность, то следует установить и ее.

```bash
npm install --save-dev @testing-library/user-events
pnpm i --dev @testing-library/user-events    # Using pnpm
yarn add --dev @testing-library/user-events  # Using yarn
```

### Тестирование компонентов

Давайте протестируем наш самый простой пример - счетчик. Рассмотрим следующий компонент, который вы могли видеть на игровой площадке:

=== "counter.jsx"

    ```js
    import { createSignal } from 'solid-js';

    export function Counter() {
    	const [count, setCount] = createSignal(1);
    	const increment = () => setCount(count() + 1);

    	return (
    		<button type="button" onClick={increment}>
    			{count()}
    		</button>
    	);
    }
    ```

=== "counter.tsx"

    ```ts
    import { createSignal, type VoidComponent } from 'solid-js';

    export const Counter: VoidComponent = () => {
    	const [count, setCount] = createSignal(1);
    	const increment = () => setCount(count() + 1);

    	return (
    		<button type="button" onClick={increment}>
    			{count().toString()}
    		</button>
    	);
    };
    ```

Мы хотим проверить, есть ли кнопка, которая подсчитывает количество нажатий; чтобы убедиться, что она работает не один раз, мы попробуем сделать это во второй раз, так что вот как выглядит наш тест:

=== "counter.test.jsx"

    ```js
    import { render } from '@solidjs/testing-library';
    import userEvent from '@testing-library/user-event';
    import { Counter } from './counter';

    const user = userEvent.setup();

    test('counts on click', async () => {
    	const { result, getByRole } = render(() => <Counter />);
    	const button = getByRole('button');
    	expect(button).toHaveTextContent('1');
    	await user.click(button);
    	expect(button).toHaveTextContent('2');
    	await user.click(button);
    	expect(button).toHaveTextContent('3');
    });
    ```

=== "counter.test.tsx"

    ```ts
    import { render } from '@solidjs/testing-library';
    import userEvent from '@testing-library/user-event';
    import { Counter } from './counter';

    const user = userEvent.setup();

    test('counts on click', async () => {
    	const { result, getByRole } = render(() => <Counter />);
    	const button: HTMLButtonElement = getByRole('button');
    	expect(button).toHaveTextContent('1');
    	await user.click(button);
    	expect(button).toHaveTextContent('2');
    	await user.click(button);
    	expect(button).toHaveTextContent('3');
    });
    ```

Для использования `toHaveTextContent` необходимо, чтобы у нас была установлена `@testing-library/jest-dom`, в противном случае необходимо заменить ее на

```ts
expect(button).toHaveProperty('textContent', '1');
```

что немного менее разборчиво, и в случае неудачи сообщение об ошибке будет менее полезным.

Если мы не хотим использовать `userEvent`, мы можем также использовать `fireEvent` из `@solidjs/testing-library`, который является синхронным, но имейте в виду, что эти события менее похожи на реальные события пользователя и в некоторых случаях могут давать разные результаты в зависимости от контекста.

### Тестирование логики многократного использования

Логика многократного использования, также известная как хуки или примитивы, также может быть протестирована с помощью `@solidjs/testing-library`, используя функцию `renderHook`.

Рассмотрим хук, который выдает нам текст "Lorem ipsum" с определенным количеством слов. Мы хотим протестировать его с 3, 2 и 5 словами, чтобы убедиться, что он работает (выберите `lorem.test.ts`):

=== "lorem.js"

    ```js
    const loremIpsumWords =
    	'Lorem ipsum dolor sit amet, consectetur…'.split(/\s+/);
    export const createLorem = (words) => {
    	return createMemo(() => {
    		const output = [],
    			len =
    				typeof words === 'function'
    					? words()
    					: words;
    		while (output.length <= len) {
    			output.push(...loremIpsumWords);
    		}
    		return output.slice(0, len).join(' ');
    	});
    };
    ```

=== "lorem.test.js"

    ```js
    import { createSignal } from 'solid-js';
    import { renderHook } from '@solidjs/testing-library';
    import { createLorem } from 'createLorem';

    test('it shows the right amount of words', () => {
    	const [words, setWords] = createSignal(3);
    	const { result } = renderHook(createLorem, [words]);
    	expect(result()).toBe('Lorem ipsum dolor');
    	setWords(2);
    	expect(result()).toBe('Lorem ipsum');
    	setWords(5);
    	expect(result()).toBe('Lorem ipsum dolor sit amet');
    });
    ```

=== "lorem.ts"

    ```ts
    const loremIpsumWords =
    	'Lorem ipsum dolor sit amet, consectetur…'.split(/\s+/);
    export const createLorem = (
    	words: Accessor<number> | number
    ) => {
    	return createMemo(() => {
    		const output = [],
    			len =
    				typeof words === 'function'
    					? words()
    					: words;
    		while (output.length <= len) {
    			output.push(...loremIpsumWords);
    		}
    		return output.slice(0, len).join(' ');
    	});
    };
    ```

=== "lorem.test.ts"

    ```ts
    import { createSignal } from 'solid-js';
    import { renderHook } from '@solidjs/testing-library';
    import { createLorem } from 'createLorem';

    test('it shows the right amount of words', () => {
    	const [words, setWords] = createSignal(3);
    	const { result } = renderHook(createLorem, [words]);
    	expect(result()).toBe('Lorem ipsum dolor');
    	setWords(2);
    	expect(result()).toBe('Lorem ipsum');
    	setWords(5);
    	expect(result()).toBe('Lorem ipsum dolor sit amet');
    });
    ```

### Тестирование пользовательских директив

Solid.js обладает полезной способностью делать многоразовыми не только логические, но и DOM-взаимодействия с помощью так называемых пользовательских директив. Очевидно, что тестирование таких директив должно быть таким же удобным, как и тестирование хуков, поэтому в нашей библиотеке тестирования имеется утилита `renderDirective`.

Примитив, который мы хотим протестировать, - это `onClickOutside`, который вызывает функцию, указанную в его аргументе, и нам нужно протестировать его, щелкнув мышью внутри и снаружи, и проверить, была ли вызвана наша функция:

=== "click-outside.js"

    ```js
    import { onCleanup } from 'solid-js';

    export function clickOutside(node, handlerAccessor) {
    	const handler = (ev) =>
    		node.contains(ev.target) || handlerAccessor()?.(ev);
    	document.body.addEventListener('click', handler);
    	onCleanup(() =>
    		document.body.removeEventListener('click', handler)
    	);
    }
    ```

=== "click-outside.test.js"

    ```js
    import { renderDirective } from '@solidjs/testing-library';
    import userEvent from '@testing-library/user-event';
    import { clickOutside } from './click-outside';

    const user = userEvent.setup();

    test('only triggers the callback on click outside', async () => {
    	const handler = jest.fn();
    	const { container: outside } = renderDirective(
    		clickOutside,
    		{ initialValue: handler }
    	);
    	const inside = outside.querySelector('div');
    	await user.click(inside);
    	expect(handler).not.toHaveBeenCalled();
    	await user.click(outside);
    	expect(handler).toHaveBeenCalledTimes(1);
    });
    ```

=== "click-outside.ts"

    ```ts
    import { onCleanup, type Accessor } from 'solid-js';

    export const clickOutside = (
    	node: HTMLElement,
    	handlerAccessor: Accessor<
    		((ev: ClickEvent) => void) | undefined
    	>
    ) => {
    	const handler = (ev: ClickEvent) =>
    		node.contains(ev.target) || handlerAccessor()?.(ev);
    	document.body.addEventListener('click', handler);
    	onCleanup(() =>
    		document.body.removeEventListener('click', handler)
    	);
    };
    ```

=== "click-outside.test.ts"

    ```ts
    import { renderDirective } from '@solidjs/testing-library';
    import userEvent from '@testing-library/user-event';
    import { clickOutside } from './click-outside';

    const user = userEvent.setup();

    test('only triggers the callback on click outside', async () => {
    	const handler = jest.fn();
    	const { container: outside } = renderDirective(
    		clickOutside,
    		{ initialValue: handler }
    	);
    	const inside: HTMLDivElement =
    		outside.querySelector('div');
    	await user.click(inside);
    	expect(handler).not.toHaveBeenCalled();
    	await user.click(outside);
    	expect(handler).toHaveBeenCalledTimes(1);
    });
    ```

## Ссылки

-   [Jest](https://docs.solidjs.com/guides/how-to-guides/testing-in-solid/jest)
