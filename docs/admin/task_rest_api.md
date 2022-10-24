# Taskrest API

Интерфейс сервиса устроен как `REST API`, который предоставляет несколько URL,
для вызова которых, используются _HTTP_ запросы _GET и POST_.

Дванные запросов передаются как:

- `GET` - строковые параметры URL;
- `POST` - в виде данных формы в теле запроса.

В ответ на запрос, сервис возвращает `JSON` объект. В каждом запросе (за исключением
тестовых), в зависимости от конфигурации, сервис может выполнять авторизацию
пользователя с помощью токена [JWT](./jwt_auth.md), который передается в специальном
заголовке запроса.

## Заголовок авторизации

Для авторизации и предоставления пользователю прав на выполнение задач сервиса,
в запросе нужно передать серверу заголовок

    Authorization Bearer JWT

Где JWT - токен подписанный секретным ключом указанным в конфигурационном
файле сервиса. Информация указанная в токене описана в разделе [Контроль доступа](./access.md).

SPA приложения автоматически добавляют этот заголовок к каждому запросу.

## JSON объект возвращаемый сервисом

В ответ на запрос сервис всегда возвращает объект в формате `JSON`:

    {
        "file": String,
        "message": String,
        "done": Boolean
    }

Поля ответа:

- `file` - если задача должна вернуть файл для скачивания, (например при формировании
пакета XML), в данном поле содержится абсолютный путь к указанному файлу;
- `message` - сообщение о результате выполнения задачи;
- `done` - логическое значение, если `true`, задача выполнена успешно.

## URL (точки) API

В описании не указана схема, адрес и порт сервер **task_rest**.

Формат описания API:

    METHOD /url
    -- тело запроса если есть
    form_field_name1: data_type (default if any) -- описание
    form_field_name2: data_type -- описание
    ...
    { -- ответ сервера
      "file": '',
      "message": '',
      "done": true/false
    }

### test - Тестовые запросы

Предназначены для проверки доступности сервера и БД.

**1) Проверка доступности сервера:**

    GET /test?param=task
    ...
    {
      "file": "",
      "message": "Тест приложения: 2022-09-26 Время: 0.0 cek.",
      "done": true
    }

**2) Проверка доступности БД:**

    GET /test?param=rdbm
    ...
    {
      "file": "",
      "message": "ТФОМС записей: 86 Время: 0.05 cek.",
      "done": true
    }

### reestr - Работа с XML реестрами и счетами

<h6>xml - Формирование XML пакета, импорт ошибок</h6>

**1) pack - Формирование пакета:**

    POST /reestr/xml/pack
    --
    mo_code: string ("250796") -- код МО
    month: string -- месяц отчета в формате YYYY-MM
    type: string ("xml") -- тип пакета, три буквы из перечисления TYPES
    pack: int (1) -- последовательный номер пакета 1-9
    test: boolean (False) -- выполнить только проверку, пакет не формировать
    sent: boolean (False) -- отметить записи включенные в пакет как отправленные
    fresh: boolean (False) -- не включать в пакет записи помеченные как отправленные
    ...
    {
      "file": имя сформированного файла,
      "message": "H записей: n, L записей: m. НАЙДЕНО ОШИБОК: en. Время: cek.",
      "done": статус завершения (false/true)
    }

**2) errf - Импорт файла ошибок:**

    POST /reestr/xml/errf
    --
    ptype: int (1) -- Тип фала с ошибками: 1-АПП 2-Онкология
    files: file-form-data -- данные загружаемого файла
    ...
    {
      "file": имя импортированного файла,
      "message": "Файл ошибок: file Записей считано en. Время: cek.",
      "done": статус завершения (false/true)
    }

    GET /reestr/xml/errf
    Выгрузка таблицы принятых ошибок в формат CSV.

<h6>inv - Импорт файла счета, формирование реестра в СМО/ТФОМС</h6>

**1) impex - Импорт файла счета:**

    POST /reestr/inv/impex
    --
    pack: int (1) -- Тип фала счета: 1-АПП 2-Онкология
    files: file-form-data -- данные загружаемого файла
    ...
    {
      "file": имя файла реестра в формате xlsx,
      "message": "Счет: имя xml файла. Записей в счете n, (МЭК m), записей в реестре rc."
      "done": статус завершения (false/true)
    }

**2) calc - Предварительный расчет услуг по месяцу:**

    POST /reestr/inv/calc
    --
    pack: int (1) -- Тип фала счета: 1-АПП 2-Онкология
    smo: int (0) -- Номер СМО из выпадающего списка
    month: string -- месяц отчета в формате YYYY-MM,
    ...
    {
      "file": имя файла реестра в формате xlsx,
      "message": "Расчет: тип расчета. Записей обработано n"
      "done": статус завершения (false/true)
    }

**3) mek - Перенос отказов по МЭК:**

    POST /reestr/inv/mek
    --
    month: string -- месяц в котором выставлен МЭК в формате YYYY-MM,
    target: string -- месяц на который нужно перенести записи с МЭК в формате YYYY-MM,
    ...
    {
      "file": "",
      "message": "Перенесли МЭКи на target записей n."
      "done": статус завершения (false/true)
    }

    GET /reestr/inv/mek
    Выгрузка записей отказов по МЭК в формат CSV.

**4) /utils/file/:dir/:subdir/:filename - Выгрузка сформированных файлов:**

    GET /utils/file/:dir/:subdir/:filename

    Точка API является служебной и не предназначена для регулярного использования.