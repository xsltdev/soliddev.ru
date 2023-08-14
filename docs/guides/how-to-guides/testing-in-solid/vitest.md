---
description: Vitest - это очень быстрый фреймворк модульных тестов, построенный на основе Vite
---

# Vitest

## Что такое Vitest?

**Vitest** - это очень быстрый фреймворк модульных тестов, построенный на основе [Vite](https://vitejs.dev/). С ним можно быстро начать работу и легко писать тесты для своих компонентов. Vitest не предназначен в первую очередь для тестирования фронтенд-компонентов, но с добавлением некоторых замечательных пакетов мы можем использовать его для тестирования наших компонентов Solid.

### Пакеты

-   [JSDOM](github.com/jsdom/jsdom)
-   [JestDOM](https://testing-library.com/docs/ecosystem-jest-dom/)
-   [Solid.js Testing Library](https://github.com/solidjs/solid-testing-library)
-   [Vitest](https://vitest.dev/)

## Начало работы

Сначала приступим к установке. Нам потребуется установить перечисленные выше пакеты.

```bash
npm i -D vitest jsdom @testing-library/jest-dom @solidjs/testing-library
# or
yarn add -D vitest jsdom @testing-library/jest-dom @solidjs/testing-library
```

### Конфигурирование Vitest

Теперь, когда мы установили необходимые пакеты, необходимо настроить vitest для работы с Solid. Создадим файл `vitest.config.ts` в корне нашего проекта.

```ts
/// <reference types="vitest" />
/// <reference types="vite/client" />
// 👆 do not forget to add the references above

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
    test: {
        environment: 'jsdom',
        globals: true,
        transformMode: { web: [/\.[jt]sx?$/] },
    },
});
```

**Примечание:** Не забудьте добавить типы `reference` в верхней части файла. Если вы не добавите их, то при попытке запустить тесты вы получите ошибку типа.

После создания файла конфигурации нам необходимо добавить скрипт в наш файл `package.json`.

```diff
{
  "name": "solid-app-name",
  "version": "0.0.0",
  "description": "",
  "scripts": {
    "start": "vite",
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview",
+   "test": "vitest"
  },
  "license": "MIT",
  "devDependencies": {
    "@solidjs/testing-library": "^0.5.1",
    "@testing-library/jest-dom": "^5.16.5",
    "jsdom": "^20.0.3",
    "typescript": "^4.9.4",
    "vite": "^4.0.3",
    "vite-plugin-solid": "^2.5.0",
    "vitest": "^0.26.2"
  },
  "dependencies": {
    "solid-js": "^1.6.6"
  }
}
```

Теперь, когда мы добавили скрипт, мы можем запустить наши тесты, выполнив команду `npm run test` или `yarn test`. Однако тестов для запуска нет, поэтому нам нужно их создать.

## Создание тестов

В Typescript/Javascript файлы тестов могут быть созданы с расширением `.test.ts` или `.spec.ts`. Тесты обычно хранятся в отдельной папке в корне проекта, большинство разработчиков предпочитают называть эту папку `tests` или `__tests__`. Однако в некоторых случаях тесты лучше хранить в той же папке, что и тестируемый компонент. Изучите различные стили и найдите тот, который подходит именно вам 🙂 .

### Создание тестового файла

В Solid мы рекомендуем создать папку `test`, в которой будут храниться все тестовые файлы. Давайте создадим папку `test` в корне нашего проекта и создадим в ней файл `App.test.ts`.

```bash
mkdir test
touch test/App.test.ts
```

Теперь, когда мы создали тестовый файл, давайте добавим в него несколько тестов.

```ts
import { render } from '@solidjs/testing-library';
import App from '../src/App';
import { describe, expect, it } from 'vitest';
import '@testing-library/jest-dom'; // 👈 this is imported in order to use the jest-dom matchers

describe('App', () => {
    it('should render the app', () => {
        const { getByText } = render(() => <App />);
        expect(
            getByText('Learn Solid')
        ).toBeInTheDocument();
    });
});
```

Приведенный выше тест проверяет, находится ли фраза `Learn Solid` в компоненте `App` после его рендеринга. Если да, то тест будет пройден. Если нет, то тест будет провален. Приведенный выше фрагмент является примером теста компонента.

## Тестирование компонентов

Возьмем простой компонент счетчика кликов:

```tsx
//counter.tsx
import { createSignal, Component } from 'solid-js';

export const Counter: Component = () => {
    const [count, setCount] = createSignal(0);

    return (
        <div
            role="button"
            onClick={() => setCount((c) => c + 1)}
        >
            Count: {count()}
        </div>
    );
};
```

Здесь мы используем библиотеку `solid-testing-library`. Наиболее важными помощниками являются `render` для управляемого рендеринга компонента в DOM, `fireEvent` для диспетчеризации событий, напоминающих реальные события пользователя, и `screen` для предоставления глобальных селекторов. Мы также используем полезные утверждения, добавленные в `expect`, предоставленные `@testing-library/jest-dom`.

Давайте создадим тестовый файл для нашего компонента `Counter`:

```ts
// counter.test.tsx
import { Counter } from '../src/counter';
import {
    cleanup,
    fireEvent,
    render,
    screen,
    afterEach,
} from 'solid-testing-library';
import { describe, expect, it } from 'vitest';
import '@testing-library/jest-dom';

describe('Counter', () => {
    afterEach(cleanup);

    it('it starts with zero', () => {
        render(() => <Counter />);

        const button = screen.getByRole('button');

        expect(button).toBeInTheDocument();
        expect(button).toHaveTextContent('Count: 0');
    });

    it('it increases its value on click', async () => {
        render(() => <Counter />);

        const button = screen.getByRole('button');

        fireEvent.click(button);
        // the event loop takes one Promise to resolve to be finished
        await Promise.resolve();
        expect(button).toHaveTextContent('Count: 1');
        fireEvent.click(button);
        await Promise.resolve();
        expect(button).toHaveTextContent('Count: 2');
    });
});
```

Приведенный выше тест проверяет компонент `counter` на корректность отображения текста и увеличение счета при нажатии на кнопку мыши.

## Тестирование реактивного состояния

Для удобства обслуживания или поддержки нескольких представлений вы можете захотеть хранить часть состояния отдельно от компонентов. В этом случае интерфейсом, на котором проводится тестирование, является само состояние. Следует помнить, что вне реактивного корня (например, `createRoot`) состояние не отслеживается, и его обновление не будет вызывать эффекты и заметки.

В качестве примера создадим и протестируем функцию `createLocalStorage`, которая будет создавать реактивное состояние, сохраняемое в локальном хранилище браузера.

=== "createLocalStorage.jsx"

    ```js
    // createLocalStorage.js
    import { createEffect } from "solid-js";
    import { createStore, Store, SetStoreFunction } from "solid-js/store";

    export function createLocalStore(name, initState){
    const localState = localStorage.getItem(name);
    const [state, setState] = createStore(initState);

    if (localState) setState(JSON.parse(localState));

    createEffect(() => (localStorage.setItem(name, JSON.stringify(state))));

    return [state, setState];
    }
    ```

=== "createLocalStorage.tsx"

    ```ts
    // createLocalStorage.ts
    import { createEffect } from "solid-js";
    import { createStore, Store, SetStoreFunction } from "solid-js/store";

    export function createLocalStore<T extends object>(name:string, initState: T): [Store<T>, SetStoreFunction<T>] {
    const localState = localStorage.getItem(name);
    const [state, setState] = createStore(initState);

    if (localState) setState(JSON.parse(localState));

    createEffect(() => (localStorage.setItem(name, JSON.stringify(state))));

    return [state, setState];
    }
    ```

Вместо создания компонента мы можем протестировать эту функцию непосредственно в изоляции. При этом необходимо учитывать, что:

1.  реактивные изменения работают только тогда, когда они имеют контекст отслеживания, предоставляемый `render` или `createRoot`
2.  что они асинхронны, но мы можем использовать `createEffect` для их перехвата.

Преимущество использования `createRoot` заключается в том, что мы можем запускать удаление вручную:

```ts
import { createRoot, createEffect } from 'solid-js';
import { createLocalStorage } from '../src/createLocalStorage'; // 👈 our function
import { describe, expect, it, beforeEach } from 'vitest';

describe('createLocalStorage', () => {
    beforeEach(() => {
        localStorage.clear();
    });

    const initialState = {
        todos: [],
        newTitle: '',
    };

    it('reads pre-existing state from localStorage', () => {
        createRoot((dispose) => {
            const savedState = {
                todos: [{ title: 'Learn Solid' }],
                newTitle: 'Learn Solid',
            };

            localStorage.setItem(
                'state',
                JSON.stringify(savedState)
            );
            const [state] = createLocalStorage(
                'state',
                initialState
            );

            expect(state).toEqual(savedState);
            dispose();
        });
    });

    it('stores new state to localStorage', () => {
        createRoot((dispose) => {
            const [state, setState] = createLocalStorage(
                'state',
                initialState
            );

            setState('newTitle', 'updated');

            return new Promise((resolve) =>
                createEffect(() => {
                    expect(
                        JSON.parse(
                            localStorage.getItem('state') ||
                                ''
                        )
                    ).toEqual({
                        todos: [],
                        newTitle: 'updated',
                    });
                    dispose();
                    resolve();
                })
            );
        });
    });

    it('updates state multiple times', async () => {
        const { dispose, setState } = createRoot(
            (dispose) => {
                const [state, setState] = createLocalStore(
                    'state',
                    initialState
                );
                return { dispose, setState };
            }
        );

        setState('newTitle', 'first');
        // wait a tick to resolve all effects
        await new Promise((done) => setTimeout(done, 0));
        expect(
            JSON.parse(localStorage.getItem('state') || '')
        ).toEqual({
            todos: [],
            newTitle: 'first',
        });

        setState('newTitle', 'second');
        // wait a tick to resolve all effects
        await new Promise((done) => setTimeout(done, 0));
        expect(
            JSON.parse(localStorage.getItem('state') || '')
        ).toEqual({
            todos: [],
            newTitle: 'second',
        });
        dispose();
    });
});
```

Вот и все! Теперь вы можете с уверенностью тестировать свои приложения Solid. Более подробную информацию о тестировании фронтенд-приложений можно найти в [Библиотеке тестирования JestDOM](https://github.com/testing-library/jest-dom) и [Фреймворке тестирования Vitest](https://vitest.dev/).

## Ссылки

-   [Vitest](https://docs.solidjs.com/guides/how-to-guides/testing-in-solid/vitest)
