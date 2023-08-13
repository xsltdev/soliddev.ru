---
description: Давайте рассмотрим, как можно использовать LESS в приложениях Solid
---

# Less

LESS расшифровывается как Leaner CSS. **LESS** - это препроцессор CSS, основанный на Javascript. LESS дает возможность использовать миксины и другие программные инструменты. Это помогает сделать код стилей чище и менее избыточным.

Давайте рассмотрим, как можно использовать LESS в приложениях Solid.

## Установка зависимостей

Для того чтобы использовать файлы LESS в приложении Solid, необходимо установить зависимость как зависимость разработки, как показано ниже:

```bash
npm install --save-dev less
pnpm i --dev less           # Using pnpm
yarn add --dev less         # Using yarn
```

## Использование LESS в приложении

Создадим файл `.less` в каталоге `src` и назовем его `styles.less`.

```less
//styles.less

.foo {
    color: red;
}

.bar {
    background-color: blue;
}
```

Обратите внимание на то, что основной синтаксис очень похож на синтаксис CSS. Если вы хотите объявить переменные в LESS, вы можете сделать это как обычно, например:

```less
//styles.less

@plainred: red;
@plainblue: blue;

.foo {
    color: @plainred;
}

.bar {
    background-color: @plainblue;
}
```

Пишите стили LESS так же, как и в любом другом месте. Давайте изменим расширение файла импорта `styles.css` на `.less`.

```js
//component.jsx

import './styles.less';

function Component() {
    return (
        <>
            <div class="foo bar">Hello, world!</div>
        </>
    );
}
```

Используя в качестве расширения файла стилей `.less` вместо `.css`, Vite (инструмент для сборки Solid) автоматически распознает, что мы импортируем LESS-файл, и компилирует LESS в CSS по требованию.

## Ссылки

-   [Less](https://docs.solidjs.com/guides/how-to-guides/styling-in-solid/less)
