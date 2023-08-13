---
description: Здесь мы расскажем о наиболее важных возможностях и паттернах JavaScript для Solid, которые вам необходимо знать
---

# JavaScript для Solid

Вы только начинаете осваивать фронтенд-фреймворки или возвращаетесь к JavaScript после нескольких лет работы? Здесь мы расскажем о наиболее важных возможностях и паттернах JavaScript, которые вам необходимо знать. JavaScript - гибкий язык, и существует множество подходов к его написанию, поэтому это не окончательное руководство для всех кодовых баз JavaScript, а руководство по "лучшим практикам" работы с Solid.

## Избегайте этих ключевых слов

Когда вы только начинаете писать на современном JavaScript, у вас может возникнуть соблазн использовать одно из этих ключевых слов:

```js
var, this, class
```

Хорошее правило - избегать их. Без них можно построить любое приложение Solid, они добавляют неоправданную сложность в код, и для них есть замена, о которой мы поговорим ниже. Есть несколько случайных случаев использования `this`, о которых мы расскажем позже в документации. А некоторые продвинутые пользователи строят свои системы с использованием `class`, но не прислушиваются к нашим советам.

## `let` и `const`

Раньше в JavaScript для объявления переменной можно было использовать `var`. В современном JavaScript есть два ключевых слова для объявления переменных: `let` и `const`. `const` объявляет переменную, которая никогда не будет установлена в какое-либо другое значение, а `let` объявляет переменную, которая _может_ быть присвоена чему-либо другому.

```js
let counter = 2;
counter++; // counter is now set to 3
const badCounter = 2;
badCounter++; // TypeError: Assignment to constant variable.

const person = { name: 'Ryan' };

// This works; we haven't reassigned `person`, we simply
// mutated the object that was stored there.
person.name = 'Joe';

const people = [person];

// This works; we haven't reassigned `people`, we simply added
// something to the array that was stored there.
people.push({ name: 'David' });
```

При работе с объектами и массивами гораздо чаще используется `const`, чем `let`, поскольку обычно требуется _мутировать_ объект, а не _переприсваивать_ переменную.

По умолчанию используется `const`. Объявляйте ее с помощью `let` только в том случае, если обнаружите, что впоследствии переменную необходимо переназначить. Таким образом, ваши намерения будут понятны читателям.

## Vite и импорт ES6

До появления современных цепочек инструментов для веб-разработки мы "импортировали" несколько JavaScript-файлов, включая в HTML теги `script`.

```html
<script src="utils.js"></script>
<script src="main.js"></script>
```

Это было непросто, поскольку необходимо было убедиться, что файл выполняется только после загрузки всех его зависимостей - порядок тегов сценария имел значение, и все это приходилось декларировать в HTML, а не в JavaScript, где использовались зависимости.

Такие средства разработки, как [Vite](https://vitejs.dev/), позволяют использовать более эффективный подход к импорту JavaScript. Vite - это сервер разработки и средство пакетирования: он выполняет ваш код локально, предоставляя такие возможности, как автоматическая перезагрузка, что облегчает разработку проекта. Когда вы готовы развернуть свой сайт, он "собирает" ваш код, оптимизируя его и разделяя JavaScript на множество тегов сценариев.

При написании наших проектов Solid мы используем [подход к импорту](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/import), основанный на JavaScript, который был представлен в версии языка ES6. Если вы еще не знакомы с этим, обязательно ознакомьтесь с полной [MDN-документацией](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/export), но вот краткий пример для ознакомления:

```js
//utils.js
function areaOfCircle(radius) {
    return Math.PI * radius ** 2;
}

export { areaOfCircle };

//main.js
import { areaOfCircle } from './utils.js';

console.log(areaOfCircle(4));
```

## Фабричные функции и замыкания

Существует множество способов объектно-ориентированного программирования на JavaScript, но наиболее распространенным из них, который можно встретить наряду с Solid, является техника определения функции, возвращающей объект.

```js
function createCar(make, year) {
    const currentYear = new Date().getFullYear();
    const age = currentYear - year;

    return {
        make,
        age,
    };
}

const myCar = createCar('Ford Fusion', 2018);
console.log(myCar.age);
```

Поскольку функции в JavaScript можно передавать как любые другие значения (т.е. JavaScript имеет _функции первого класса_), объект, созданный фабричной функцией, может иметь свои собственные функции:

```js
function createCar(make, year) {
    const currentYear = new Date().getFullYear();
    const age = currentYear - year;

    function drive() {
        console.log(make + ' goes VROOM');
    }

    return {
        make,
        age,
        drive,
    };
}

const myCar = createCar('Ford Fusion', 2018);
myCar.drive(); //Ford Fusion goes VROOM
```

Обратите внимание, что функция `drive` использует значение `make`, которое было параметром внешней функции. В дальнейшем, когда вызывается `myCar.drive()`, она "запоминает" значение `make`. Это называется _закрытием_, поскольку значение, находящееся вне функции, было "вложено" в функцию, чтобы она могла использовать его в дальнейшем.

Мы можем использовать эти идеи для написания фабрик, непосредственно возвращающих функции:

```js
function multiplyBy(multiplier) {
    return function (number) {
        return multiplier * number;
    };
}

const multiplicator = multiplyBy(2);
console.log(multiplicator(7)); //14
```

## Деструктуризация

В JavaScript принято передавать объекты в качестве аргументов:

```js
function gameSetup(options) {
    initializeScreen(
        options.screenWidth,
        options.screenHeight
    );
    if (options.multiplayer) {
        startMultiplayerGame();
        return;
    }
    startGame();
}
```

Если мы знаем, что эти три свойства (`screenWidth`, `screenHeight` и `multiplayer`) присутствуют у объекта, то мы можем "деструктурировать" его и избавить себя от повторения имени объекта:

```js
function gameSetup(options, playersArray) {
    const { screenWidth, screenHeight, multiplayer } =
        options;
    initializeScreen(screenWidth, screenHeight);
    if (multiplayer) {
        console.log(
            'Starting a game with' +
                playersArray[0] +
                ' and ' +
                playersArray[1]
        );
        startTwoPlayerGame(
            playersArray[0],
            playersArray[1]
        );
        return;
    }
    startGame();
}
```

Мы также можем деструктурировать массивы:

```js
function gameSetup(options, playersArray) {
    const { screenWidth, screenHeight, multiplayer } =
        options;
    initializeScreen(screenWidth, screenHeight);
    if (multiplayer) {
        const [player1, player2] = playersArray;
        console.log(
            'Starting a game with' +
                player1 +
                ' and ' +
                player2
        );
        startTwoPlayerGame(player1, player2);
        return;
    }
    startGame();
}
```

Деструктуризация позволяет нам делать [много трюков](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), и вы часто будете видеть ее в Solid.

## Шаблонные литералы

Мы можем упростить вывод на консоль в предыдущем примере, используя шаблонные литералы:

```js
console.log(
    `Starting a game with ${player1} and ${player2}`
);
```

Шаблонные литералы также позволяют создавать многострочные строки без необходимости вручную вставлять символ `\n`:

```js
const multiline = `spans two
lines`;
console.log(multiline);
//spans two
//lines
```

Некоторые инструменты в экосистеме Solid используют [tagged templates](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Template_literals#tagged_templates), которые позволяют определить функцию, работающую с литералом шаблона и любыми включенными в него переменными. Это позволяет легко придать строкам особую функциональность. Например, при использовании Solid без JSX используется тэгированный шаблон `html`:

```js
import html from "https://cdn.skypack.dev/solid-js/html";

function App() {
  const [count, setCount] = ...
  ...
  return html`<div>${count}</div>`;
}

```

## Стрелочные функции

В JavaScript существует множество способов объявления функции. Вот большинство из них:

1.  [Объявление функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions#%D0%BE%D0%B1%D1%8A%D1%8F%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D0%B8_%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%86%D0%B8%D1%8F_function)

    ```js
    function areaOfCircle(radius) {
        return radius * Math.PI ** 2;
    }
    ```

2.  [Метод](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions#%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BC%D0%B5%D1%82%D0%BE%D0%B4%D0%BE%D0%B2)

    ```js
    const utils = {
        areaOfCircle(radius) {
            return radius * Math.PI ** 2;
        },
    };
    ```

3.  [Функциональное выражение](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions#%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F-%D0%B2%D1%8B%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5_%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%82%D0%BE%D1%80_function)

    ```js
    const areaOfCircle = function (radius) {
        return radius * Math.PI ** 2;
    };
    ```

4.  [Стрелочные функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions#%D1%81%D1%82%D1%80%D0%B5%D0%BB%D0%BE%D1%87%D0%BD%D0%B0%D1%8F_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F-%D0%B2%D1%8B%D1%80%D0%B0%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5)

    ```js
    const areaOfCircle = (radius) => {
        return radius * Math.PI ** 2;
    };

    // Or the shorthand if you only have one line of code
    // in the function body:
    const areaOfCircle = (radius) => radius * Math.PI ** 2;
    ```

Между этими объявлениями есть нюансы - например, добавление `console.log(this);` внутри каждого из них - но большинство из них не имеют отношения к повседневной работе с Solid (если интересно, посмотрите эту [подробную статью](https://dmitripavlutin.com/differences-between-arrow-and-regular-functions/)).

Если вы сомневаетесь, используйте функцию-стрелку (пример №4). При условии, что вы следуете предыдущему совету и не используете ключевое слово `this`, разницы между стрелочной функцией и функциональным выражением не будет.

## Функции как аргументы

Ранее мы рассмотрели, как можно вернуть функцию из функции. Функцию также можно принимать в качестве аргумента функции:

```js
function callForEach(array, func) {
    for (let i = 0; i < array.length; i++) {
        const currentElement = array[i];
        func(currentElement);
    }
}

const myArray = ['I', 'love', 'Solid'];
callForEach(myArray, console.log);
//I
//love
//Solid
```

Многие функции, встроенные в JavaScript (и многие, поставляемые с Solid), принимают аргумент функции. Приведенная выше функция действительно является встроенной в массивы JavaScript:

```js
const myArray = ['I', 'love', 'Solid'];
myArray.forEach(console.log);
//I
//love
//Solid
```

Другим распространенным примером является встроенный метод `map`. Он принимает в качестве аргумента функцию, которая сопоставляет текущий элемент массива с новым результатом:

```js
const myArray = ['I', 'love', 'Solid'];
const uppercase = myArray.map((element) =>
    element.toUpperCase()
); // ["I", "LOVE", "SOLID"]
```

## Декларативный и императивный код

Методы массивов JavaScript, такие как `map` (и другие, такие как [`reduce`](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) и [`filter`](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)), могут заменить циклы, которые вы можете сделать с помощью `for` или `while`. Эти традиционные циклы называются "императивными" - в них пошагово описывается, как выполнить то или иное действие:

```js
const myNumbers = [1, 2, 3, 4, 5, 11, 18, 65];
let oddNumbers = [];
for (const number of myNumbers) {
    if (number % 2 !== 0) {
        oddNumbers.push(number);
    }
}
console.log(oddNumbers); // [ 1, 3, 5, 11, 65 ]
```

Использование методов массивов JavaScript позволяет получить более "декларативный" код, в котором указывается _что_ нужно сделать, но не указываются все детали того, как это сделать.

```js
const myNumbers = [1, 2, 3, 4, 5, 11, 18, 65];
const oddNumbers = myNumbers.filter(
    (number) => number % 2 !== 0
);
console.log(oddNumbers); // [ 1, 3, 5, 11, 65 ]
```

Здесь мы сделали то же самое в одной строке кода с помощью метода массива `filter`, который принимает функцию и возвращает новый массив, содержащий только те элементы, которые возвращают true при передаче функции.

Когда мы говорим, что пишем "декларативный код", мы имеем в виду, что используем абстракции для написания кода, который более упорядочен: больше сосредоточен на том, что мы пытаемся сделать, чем на том, как именно. Использование `фильтра` позволяет нам писать меньше "шаблонного" зацикленного кода, а вместо этого сосредоточиться на основной функциональности, которая нам нужна. Код проще читать: циклы `for` могут использоваться для самых разных целей, но всякий раз, когда вы видите `filter`, вы знаете, что целью цикла является игнорирование некоторых элементов в массиве.

HTML - отличный пример _декларативного_ способа написания: вы указываете, какой должна быть структура, а не то, как она должна быть собрана или отображена. Как мы надеемся показать в этом руководстве, Solid облегчает написание декларативного кода при работе с пользовательскими интерфейсами.

## Ссылки

-   [JavaScript for Solid](https://docs.solidjs.com/guides/foundations/javascript-for-solid)
