---
description: Здесь мы будем использовать сервис Firebase's hosting для размещения нашего проекта Solid
---

# Развертывание на Firebase

Здесь мы будем использовать сервис [Firebase's hosting](https://firebase.google.com/products/hosting) для размещения нашего проекта Solid.

Если вы еще не слышали о Firebase, то это платформа для разработки приложений, управляемая компанией Google. Для получения более подробной информации о Firebase и предлагаемых ею услугах обязательно посетите их [веб-сайт](https://firebase.google.com/).

Прежде чем приступить к работе, убедитесь, что в консоли Firebase создан проект, который будет использоваться для размещения проекта Solid. Как это сделать, описано в первом шаге [данного руководства](https://firebase.google.com/docs/web/setup#create-firebase-project-and-app).

## Использование инструмента Firebase CLI

**Шаг 1:** Убедитесь, что у вас правильно установлен [`firebase-tools`](https://www.npmjs.com/packages/firebase-tools)

```bash
npm i -g firebase-tools
# or
pnpm i -g firebase-tools
# or
yarn global add firebase-tools
```

**Шаг 2:** Выполните команду `firebase login` и убедитесь, что вы вошли в свою учетную запись Firebase, в которой был создан проект Firebase, используемый для размещения проекта Solid.

**Шаг 3:** В корневой каталог проекта Solid добавьте два файла: `firebase.json` и `.firebaserc`.

В файле `firebase.json` скопируйте следующий код:

```json
{
    "hosting": {
        "public": "dist",
        "ignore": []
    }
}
```

И в `.firebaserc` скопируйте следующий код:

```json
{
    "projects": {
        "default": "<YOUR_FIREBASE_PROJECT_ID>"
    }
}
```

**Шаг 4:** Запустите команду `npm run build` и выполните команду `firebase deploy`. После выполнения команды deploy в командной строке должно появиться сообщение следующего вида

![Шаг 4](firebase-deploy-done.png)

URL `Hosting URL` - это реальное развертывание вашего проекта. Не стесняйтесь посетить его и посмотреть на свою работу в действии.

!!!note ""

    Более подробную информацию о Firebase и о том, какие еще интересные вещи можно создавать с помощью Solid и Firebase, можно найти на сайте [Firebase documentation](https://firebase.google.com/docs/web)

## Ссылки

-   [Deploying To Firebase](https://docs.solidjs.com/guides/how-to-guides/deployment/deploying-to-firebase)
