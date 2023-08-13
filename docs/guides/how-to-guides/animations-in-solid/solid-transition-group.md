---
description: Solid Transiton Group - это библиотека анимации, предназначенная для создания плавных переходов между состояниями
---

# Solid Transition Group

**Solid Transiton Group** - это библиотека анимации, предназначенная для создания плавных переходов между состояниями. Это означает, что с ее помощью легче анимировать элементы, входящие и выходящие из DOM.

## Установка

```bash
npm install solid-transition-group
```

или

```bash
yarn add solid-transition-group
```

## Transition

Компонент `<Transition>` - это компонент-обертка, который обеспечивает эффекты перехода для одного дочернего компонента. Компонент `<Transition>` применяет поведение перехода только к обернутому внутри него содержимому; он не создает дополнительного элемента DOM и не отображается в проверяемой иерархии компонентов.

Приведем пример простого перехода с использованием CSS- и JS-анимации:

```js
// simple CSS animation
<Transition name="slide-fade">{show() && <div>Hello</div>}</Transition>

// JS Animation
<Transition
  onEnter={(el, done) => {
    const a = el.animate([{ opacity: 0 }, { opacity: 1 }], {
      duration: 600
    });
    a.finished.then(done);
  }}
  onExit={(el, done) => {
    const a = el.animate([{ opacity: 1 }, { opacity: 0 }], {
      duration: 600
    });
    a.finished.then(done);
  }}
>
  {show() && <div>Hello</div>}
</Transition>
```

Вот как выглядит CSS для `'slide-fade'`:

```css
.slide-fade-enter-active,
.slide-fade-exit-active {
    transition: all 0.3s ease;
}
.slide-fade-enter,
.slide-fade-exit-to {
    transform: translateX(10px);
    opacity: 0;
}
```

### Свойства

| Имя              | Тип                  | По умолчанию     | Описание                                                                                                                                                                                         |
| ---------------- | -------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| name             | `string`             | `undefined`      | Имя перехода, используемое для генерации имен CSS-классов переходов. Например, имя `fade` будет автоматически расширяться до `fade-enter`, `fade-enter-active`, `fade-exit`, `fade-exit-active`. |
| appear           | `boolean`            | `false`          | Применять ли переход при начальном рендеринге компонента.                                                                                                                                        |
| mode             | `'in-out', 'out-in'` | `'simultaneous'` | Режим перехода, который управляет временной последовательностью переходов выхода/входа.                                                                                                          |
| enterActiveClass | `string`             | `undefined`      | Имя CSS-класса, который будет применяться к элементу во время перехода enter.                                                                                                                    |
| enterClass       | `string`             | `undefined`      | Имя класса CSS, применяемое к элементу перед переходом enter.                                                                                                                                    |
| enterToClass     | `string`             | `undefined`      | Имя класса CSS, которое будет применяться к элементу после перехода enter.                                                                                                                       |
| exitActiveClass  | `string`             | `undefined`      | Имя CSS-класса, применяемое к элементу во время перехода exit.                                                                                                                                   |
| exitClass        | `string`             | `undefined`      | Имя CSS-класса, применяемого к элементу перед переходом на выход.                                                                                                                                |
| exitToClass      | `string`             | `undefined`      | Имя класса CSS, которое будет применяться к элементу после перехода выхода.                                                                                                                      |

### События

| Имя             | Тип                                       | Описание                                                                         |
| --------------- | ----------------------------------------- | -------------------------------------------------------------------------------- |
| `onEnter`       | `(el:Element, done: () => void) => void`  | Функция обратного вызова, которая будет вызвана, когда начнется переход `enter`. |
| `onBeforeEnter` | `(el:Element) => void`                    | Функция обратного вызова, которая будет вызвана до начала перехода `enter`.      |
| `onAfterEnter`  | `(el:Element) => void`                    | Функция обратного вызова, которая будет вызвана после начала перехода `enter`.   |
| `onExit`        | `(el:Element, done: () => void ) => void` | Функция обратного вызова, которая будет вызвана после начала перехода `exit`.    |
| `onBeforeExit`  | `(el:Element) => void`                    | Функция обратного вызова, которая будет вызвана перед началом перехода `exit`.   |
| `onAfterExit`   | `(el:Element) => void`                    | Функция обратного вызова, которая будет вызвана после начала перехода выхода.    |

## TransitionGroup

`<TransitionGroup>` - это компонент-обертка, который обеспечивает эффекты перехода для нескольких дочерних компонентов. Компонент `<TransitionGroup>` применяет поведение перехода только к содержимому, находящемуся внутри обертки; он не создает дополнительного элемента DOM и не отображается в проверяемой иерархии компонентов.

`<TransitionGroup>` поддерживает перемещение переходов с помощью трансформации CSS. При изменении положения дочернего компонента на экране после обновления к нему будет применен перемещаемый CSS-класс (автоматически сгенерированный из атрибута name или настроенный с помощью атрибута move-class). Если в момент применения класса перемещения свойство CSS transform имеет значение "transition-able", то элемент будет плавно анимирован до места назначения с использованием техники FLIP.

```js
<ul>
    <TransitionGroup name="slide">
        <For each={state.items}>
            {(item) => <li>{item.text}</li>}
        </For>
    </TransitionGroup>
</ul>
```

Приведенный выше код будет выдвигать каждый элемент на экран при добавлении его в список или при изменении порядка списка. Ниже приведен CSS для перехода `'slide'`:

```css
.slide-move {
    transition: transform 0.3s ease;
}
.slide-enter-active,
.slide-exit-active {
    transition: all 0.3s ease;
}
.slide-enter,
.slide-exit-to {
    transform: translateX(10px);
    opacity: 0;
}
```

### Свойства

Единственное различие между `<TransitionGroup>` и `<Transition>` заключается в том, что `<TransitionGroup>` имеет дополнительное свойство `'moveClass'`, которое перезаписывает CSS-класс, применяемый при переходе.

### События

`<TransitionGroup>` также поддерживает те же события, что и `<Transition>`.

!!!note ""

    Дополнительную информацию о Solid Transition Group можно найти в [GitHub Repo](https://github.com/solidjs/solid-transition-group).

    Более подробную информацию о методике FLIP можно найти на сайте [Aerotwist](https://aerotwist.com/blog/flip-your-animations/).

## Ссылки

-   [Solid Transition Group](https://docs.solidjs.com/guides/how-to-guides/animations-in-solid/solid-transition-group)
