# Аутентификация по JWT

Запросы к сервисам [pg_rest](./pg_rest.md) и [task_rest](./task_rest.md) выполняются в [SPA](./spa.md) с помощию объекта <a href="https://developer.mozilla.org/ru/docs/Web/API/XMLHttpRequest" target=_blank>XMLHttpRequest</a> обернутого в <a href="https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise">Promise</a>.

При загрузке _SPA_ в брузер пользователя в <a href="https://developer.mozilla.org/ru/docs/Web/API/Window/localStorage" target=_blank>localStorage</a> бразера передается параметр:

    "token_url": "/apps/token"

Если пользователь аутентифицировался на сайте [omslite.site](http://omslite.site), то при обращении по адресу __"token_url"__ клиент в `HTTP` ответе получит дополнительный заголовок вида:

    Authorization: eyJ0eXAiOiJ...

Значением заголовка является <a href="https://jwt.io/" target=_blank>**JWT**</a>, который
сохранияется, и далее используется в зпросах к сервисами *pg_rest* и *task_rest* для
аутентифицикации запросов клиента к БД.

Не аутентифицированный клиент такого заголовка не получает, и статус запроса будет равен:

    404 Not Found.

_SPA_ автоматически обращаются по адресу *"token_url"* для получения _JWT_, если в ответе
нет заголовка `Authorization`, или заголовок пустой, в зпросах к сервисами *pg_rest*
и *task_rest* он не используестся.

В актуальной версии сервиса аутентифицированный клиент всегда получает _JWT_, соответвенно
_SPA_ всегда будут отправлять сервисам *pg_rest* и *task_rest* заголовок `Authorization`
в запрсах, и следовательно сервисы должны быть сконфигурированы таким образом, чтобы кореектно
обрабатывать _JWT_.

Более подробную информацию относительно обработки _JWT_ можно найти в разделе
[Контроль доступа](./access.md).
