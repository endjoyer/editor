# Editor

Проект "Editor" - это продвинутый WYSIWYG-редактор.

## Что в нем есть

1. Панель с кнопками.
   Функционал по очереди:

   - назад,
   - вперед по истории,
   - превратить выделенный текст в заголовок,
   - превратить выделенный текст в абзац,
   - загрузить картинку на место курсора в редакторе по URL’у (при нажатии появляется модальное окно). Размер картинки после добавления можно менять,
   - копирование HTML’a в буфер обмена,
   - сохранение контента в кэш,
   - очистка контента из кэша и со страницы.

2. Модальное окно, для добавления изображений.
3. Блокнот. Ни знаю что о нем сказать, ну например на Tab создаются 4 пробела.

## Технологии

- Vue
- CSS

## О разработке

На создание этого приложения я потратил 1 день. Так что получилась достаточно небрежно, я не разбивал код на компоненты и пользовался chatGPT. Но все функции, что от него требуется, он выполнят.

## Планы на улучшения

- Добавить аутентификацию, авторизацию и регистрацию
- Добавить бэкенд с базой данных
- Добавить разделы с заметками
- Добавить поиск заметок
- Добавить форму создания новой заметки

## Веб-сайт с этим приложением

[https://endjoyer.github.io/editor/](https://endjoyer.github.io/editor/)

## Project setup

```
npm install
```

### Compiles and hot-reloads for development

```
npm run serve
```

### Compiles and minifies for production

```
npm run build
```

### Deploy on Github pages

Use file "deploy.sh"
