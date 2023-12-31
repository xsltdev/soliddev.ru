---
description: Выполняет заданную функцию под указанным владельцем, вместо владельца внешней области видимости и не затрагивая его
---

# runWithOwner

```ts
function runWithOwner<T>(owner: Owner, fn: (() => void) => T): T;
```

Выполняет заданную функцию под указанным владельцем, вместо владельца внешней области видимости (и не затрагивая его). По умолчанию вычисления, созданные функциями `createEffect`, `createMemo` и т.д., принадлежат владельцу текущего выполняющегося кода (возвращаемое значение `getOwner`), поэтому, в частности, они будут утилизированы, когда это сделает их владелец. Вызов `runWithOwner` дает возможность переопределить это значение по умолчанию на указанного вручную владельца (обычно это возвращаемое значение предыдущего вызова `getOwner`), что позволяет более точно контролировать момент утилизации вычислений.

Наличие (правильного) владельца важно по двум причинам:

-   Вычисления без владельца не могут быть очищены. Например, если вызвать `createEffect` без владельца (например, в глобальной области видимости), то эффект будет работать вечно, а не будет утилизирован, когда его владелец избавится от него.
-   `UseContext` получает контекст, проходя по дереву владельцев в поисках ближайшего предка, предоставляющего нужный контекст. Таким образом, не имея владельца, вы не сможете найти ни одного предоставленного контекста (а при неправильном владельце вы можете получить неправильный контекст).

Ручная установка владельца особенно полезна при выполнении реактивных действий вне области действия владельца. В частности, при асинхронных вычислениях (через функции `async` или обратные вызовы типа `setTimeout`) автоматически установленный владелец теряется, поэтому запоминание первоначального владельца через `getOwner` и восстановление его через `runWithOwner` в таких случаях необходимо. Например:

```ts
const owner = getOwner();
setTimeout(() => {
    // This callback gets run without owner.
    // Restore owner via runWithOwner:
    runWithOwner(owner, () => {
        const foo = useContext(FooContext);
        createEffect(() => {
            console.log(foo);
        });
    });
}, 1000);
```

Обратите внимание, что владельцы не определяют отслеживание зависимостей, поэтому `runWithOwner` не поможет с отслеживанием в асинхронных функциях; использование реактивного состояния в асинхронной части (например, после первого `await`) не будет отслеживаться как зависимость.

## Ссылки

-   [runWithOwner](https://docs.solidjs.com/references/api-reference/reactive-utilities/runWithOwner)
