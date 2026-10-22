# Postman API tests: коллекция + Newman + CI

[![Postman API Tests](https://github.com/jekarevenko-byte/postman-api-tests/actions/workflows/tests.yml/badge.svg)](https://github.com/jekarevenko-byte/postman-api-tests/actions/workflows/tests.yml)
[![Report](https://img.shields.io/badge/HTML-report-FF6C37?logo=postman&logoColor=white)](https://jekarevenko-byte.github.io/postman-api-tests/)
![Newman](https://img.shields.io/badge/Newman-6.x-FF6C37)

Коллекция Postman с автотестами для публичного REST API [JSONPlaceholder](https://jsonplaceholder.typicode.com). Запускается из Postman и из командной строки (Newman), работает в GitHub Actions и публикует HTML-отчёт на GitHub Pages. Дополняет Python-проект [qa_api_tests](https://github.com/jekarevenko-byte/qa_api_tests): те же эндпоинты, другой инструмент.

**[HTML-отчёт последнего прогона](https://jekarevenko-byte.github.io/postman-api-tests/)** · [data-driven отчёт](https://jekarevenko-byte.github.io/postman-api-tests/data-driven.html)

## Что показано

| Приём | Где смотреть в коллекции |
|---|---|
| CRUD: GET / POST / PUT / PATCH / DELETE | папки `Users`, `Posts` |
| Проверки статус-кода, заголовков, времени ответа | тесты каждого запроса; общие проверки на уровне коллекции |
| Валидация **JSON-схемы** (`pm.response.to.have.jsonSchema`) | `GET all users`, `GET all posts`, `GET comments of a post` |
| Негативные сценарии: 404, пустая выборка | `GET user - not found`, `GET posts filtered by unknown userId` |
| **Pre-request скрипты** и динамические данные | `POST create user`, `POST create post` |
| **Цепочка запросов**: значение из ответа идёт в следующий запрос (collection variables) | папка `Scenario - chained requests` |
| **Data-driven**: один запрос, несколько строк данных | папка `Data-driven`, файл `data/posts.json` |
| **Переменные и окружения** | `environments/jsonplaceholder.postman_environment.json` |
| **CI** и HTML-отчёт | `.github/workflows/tests.yml` |

## Структура

```
postman-api-tests/
├── collections/jsonplaceholder.postman_collection.json
├── environments/jsonplaceholder.postman_environment.json   ← baseUrl, userId, postId, nonexistentId
├── data/posts.json                                         ← данные для data-driven прогона
├── package.json                                            ← скрипты запуска Newman
└── .github/workflows/tests.yml
```

## Запуск в Postman

1. Postman → **Import** → выберите оба файла: коллекцию и окружение.
2. Справа вверху выберите окружение **JSONPlaceholder**.
3. Правый клик по коллекции → **Run collection**. Для папки `Data-driven` на вкладке Run выберите *Select file* → `data/posts.json`.

## Запуск в командной строке (Newman)

Нужен [Node.js](https://nodejs.org/) 18+.

```bash
npm install
npm test             # папки Users, Posts и сценарий; отчёты в reports/
npm run test:data    # data-driven: по одной итерации на строку data/posts.json
npm run test:all     # оба прогона
```

После запуска откройте `reports/index.html` в браузере. Код возврата Newman не нулевой, если упала хоть одна проверка, поэтому CI краснеет автоматически.

## CI

Workflow запускается на push и pull request в `main`, по будням и вручную. Он ставит Newman, делает оба прогона, сохраняет отчёты как artifact `newman-reports` и публикует HTML в ветку `gh-pages`. Однократно включите **Settings → Pages → Deploy from a branch → `gh-pages` / root**.

## Примечания

- JSONPlaceholder не сохраняет изменения: POST/PUT/PATCH/DELETE возвращают ожидаемый ответ, но данные на сервере не меняются. Поэтому тесты проверяют ответ сервера, а не последующее чтение.
- Сервис общедоступный и может отвечать медленно; порог времени ответа задан в коллекции (3000 мс).
- Как коллекцию проверяли без Newman: пока не появится первый прогон в Actions, она проходила только на локальной имитации API. Если какой-то тест упадёт на реальном сервисе, смотрите отчёт в Actions.
