# REST-интерфейс на базе JSON

Vibe.d позволяет быстро реализовать веб-сервис
на основе JSON. Если необходимо получить следующий
вывод по HTTP запросу с адресом `/api/v1/chapters`:

    [
      {
        "title": "Hello",
        "id": 1,
        "sections": [
          {
            "title": "World",
            "id": 1
          }
        ]
      },
      {
        "title": "Advanced",
        "id": 2,
        "sections": []
      }
    ]

Сначала определяется интерфейс, реализующий
соответствующие функции и структуры, которые
сериализуются **1:1**:

    interface IRest
    {
        struct Section {
            string title;
            int id;
        }
        struct Chapter {
            string title;
            int id;
            Section[] sections;
        }
        @path("/api/v1/chapters")
        Chapter[] getChapters();
    }

Для заполнения струкур необходимо унаследовать
интерфейс и реализовать бизнес-логику:

    class Rest: IRest {
        Chapter[] getChapters() {
          // заполнение
        }
    }

С помощью объекта `URLRouter` регистрируется
объект класса `Rest`. Всё готово!

    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

*Генератор REST-интерфейсов* vibe.d так же поддерживает
запросы POST, при обработке которых потомки переданного
JSON-объекта сопоставляются параметрам функции-члена.

Интерфейс REST можно использовать для генерации
REST-клиента, прозрачно передающего JSON-запросы
на выбранный сервер, при этом используются те
же функции-члены, что и на бэк-энде. Такой подход
очень удобен, когда клиент и сервер используют
один и тот же код.

    auto api = new RestInterfaceClient!IRest("http://127.0.0.1:8080/");
    // передаёт GET /api/v1/chapters
    // и десериализует ответ в массив
    // IRest.Chapter[]
    auto chapters = api.getChapters();

## {SourceCode:disabled}

```d
import vibe.d;

interface IRest
{
    struct Section
    {
        string title;
        int id;
    }

    struct Chapter
    {
        string title;
        int id;
        Section[] sections;
    }

    @path("/api/v1/chapters")
    Chapter[] getChapters();

    /*
    Передать данные POST-запросом:
        { "title": "D Language" }
    */
    @path("/api/v1/add-chapter")
    @method(HTTPMethod.POST)
    int addChapter(string title);
}

class Rest: IRest
{
    private Chapter[] chapters_;

    this()
    {
        chapters_ = [ Chapter("Hello", 1,
                [ Section("World", 1) ] ),
                 Chapter("Advanced", 2) ];
    }

    Chapter[] getChapters()
    {
        return chapters_;
    }

    int addChapter(string title)
    {
        import std.algorithm : map, max, reduce;
        // Генерация следующего наибольшего ID
        auto newId = chapters_.map!(x => x.id)
                            .reduce!max + 1;
        chapters_ ~= Chapter(title, newId);
        return newId;
    }
}

shared static this()
{
    auto router = new URLRouter;
    router.registerRestInterface(new Rest);

    auto settings = new HTTPServerSettings;
    settings.port = 8080;
    listenHTTP(settings, router);
}
```
