# Sass

**Sass** - это супернабор CSS, который упрощает создание CSS. Sass интегрируется с Solid в качестве встроенного плагина Vite.

## Установите Sass

```sh
npm i --save-dev sass
pnpm i --dev sass   # using pnpm
yarn add --dev sass # using yarn
```

## Преобразование расширений имен файлов

После установки нам просто необходимо изменить расширения имен файлов `.css` на `.scss` или `.sass`. Имена файлов `.scss` представляют собой строгий супернабор CSS. Имена файлов `.sass` - это альтернативный синтаксис для создания Sasss, который синтаксически более свободен. Как правило, рекомендуется использовать `.scss`, но Vite поддерживает оба варианта.

Для начала работы достаточно преобразовать имена файлов `.css` в `.scss` или `.sass` и импортировать их:

```diff
// Card.scss
.grid {
  display: grid;
+  &.center { place-items: center; }
}
.screen { min-height: 100vh; }

.card {
  height: 160px;
  aspect-ratio: 2;
  border-radius: 16px;
  background-color: white;
  box-shadow: 0 0 0 4px hsl(0 0% 0% / 15%);
}
```

```diff
// Card.jsx
+ import "./card.scss"

function Card() {
  return (<>
    <div class="grid grid-center screen">
      <div class="card">Hello, world!</div>
    </div>
  </>);
}
```

Когда мы меняли `.css` на `.scss` или `.sass`, Vite автоматически распознавал эти имена файлов и компилировал Sass в CSS по требованию. При сборке для производства все файлы Sass преобразуются в CSS и, таким образом, могут быть интерпретированы браузером.

## Ссылки

-   [Sass](https://docs.solidjs.com/guides/how-to-guides/styling-in-solid/sass)
