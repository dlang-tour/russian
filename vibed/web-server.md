# Веб-сервер

Vibe.d позволяет создавать HTTP(S) веб-серверы
практически мгновенно:

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, &foo);

Этот код запускает веб-сервер на порту 8080,
все запросы обрабатываются функцией `foo`:

    void foo(HTTPServerRequest req,
        HTTPServerResponse res) { ... }

Для упрощения типичного использования и конфигурации
различных путей предоставляется класс `URLRouter`,
который позволяет регистрировать обработчики
`GET`, `POST` и т.п. либо посредством функций-членов
`.get("path", handler)` и `.post("path", handler)`,
либо посредством регистрации пользовательского класса,
реализующего пути в виде функций-членов:

    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    listenHTTP(settings, router);

Пути для вызова соответствующих функций пользовательского
класса `WebService` определяются по следующей простой схеме:
* `index()` обрабатывает `/index`
* `getName()` обрабатывает `GET`-запрос `/name`
* `postUsername()` обрабатывает `POST`-запрос
  `/username`

Пользовательские пути можно назначить, указав
для функции члена атрибут `@path("/hello/world")`.
Параметры `POST`-запросов передаются в виде
переменных, имя которых имеет префикс `_`.
Также возможно указать параметры непосредственно
в пути:

    @path("/my/api/:id")
    void foo(int _id)

Нет необходимости вручную передавать объекты `HTTPServerResponse` и
`HTTPServerRequest` в виде параметров каждой функции.
Vibe.d статически проверяет, есть ли они в списке параметров
функции и передаёт их, если они требуются.

## {SourceCode:disabled}

```d
import vibe.d;

class WebService
{
    /*
    При использовании переменных сессии,
    таких как эта, можно сохранять информацию,
    ассоциированную с конкретным пользователем,
    в процессе обработки всех запросов
    в течение пользовательской сессии.
    */
    private SessionVar!(string, "username")
        username_;

    /*
    По умолчанию запросы по корневому пути ("/")
    передаются в метод index.
    */
    void index(HTTPServerResponse res)
    {
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <form action="/username" method="POST">
        Your name:
        <input type="text" name="username">
        <input type="submit" value="Submit">
        </form>
        </body>
        </html>};

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    /*
    Атрибут @path можно использовать для
    тонкой настройки маршрутизации. В данном
    случае запросы "/home" передаются
    в метод getHome.
    */
    @path("/name")
    void getName(HTTPServerRequest req,
            HTTPServerResponse res)
    {
        import std.string : format;

        // Инспектируется свойство запроса
        // headers и генерируются
        // теги <li>.
        string[] headers;
        foreach(key, value; req.headers) {
            headers ~=
                "<li>%s: %s</li>"
                .format(key, value);
        }
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Your name: %s</h1>
        <h2>Headers</h2>
        <ul>
        %s
        </ul>
        </body>
        </html>}.format(username_,
                headers.join("\n"));

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }

    void postUsername(string username,
            HTTPServerResponse res)
    {
        username_ = username;
        auto contents = q{<html><head>
            <title>Tell me!</title>
        </head><body>
        <h1>Your name: %s</h1>
        </body>
        </html>}.format(username_);

        res.writeBody(contents,
                "text/html; charset=UTF-8");
    }
}

void helloWorld(HTTPServerRequest req,
        HTTPServerResponse res)
{
    res.writeBody("Hello");
}

shared static this()
{
    auto router = new URLRouter;
    router.registerWebInterface(new WebService);
    router.get("/hello", &helloWorld);

    auto settings = new HTTPServerSettings;
    // Необходимо для использования SessionVar.
    settings.sessionStore =
        new MemorySessionStore;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
