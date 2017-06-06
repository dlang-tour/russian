# Шаблоны DIET

Чтобы упростить создание веб-страниц, в vibe.d
включена поддержка [шаблонов DIET](https://vibed.org/templates/diet),
предоставляющих упрощённый синтаксис для HTML-страниц.
DIET основаны на [шаблонах Jade](http://jade-lang.com/).

    doctype html
    html(lang="en")
      head
        // Код D исполняется
        title #{pageTitle}
        // атрибуты
        script(type='text/javascript')
          if (foo) bar(1 + 5)
        // ID = body-id
        // style = the-style
        body#body-id.the-style
          h1 DIET template

Синтаксис строится на отступах, поэтому закрывающие
теги вставлять не нужно.

Все шаблоны DIET компилируются и хранятся в
памяти для обеспечения максимального быстродействия.
Шаблоны DIET позволяют использовать код D, который
исполняется при генерации страницы. Одиночные
выражения записываются в блоках вида `#{ 1 + 1}`.
Их можно использовать в любом месте шаблона.
Полноценные строки кода D записываются с префиксом
`-` на новой строке:

    - foreach(title; titles)
      h1 #{title}

Таким образом можно использовать сложные выражения
и даже определять функции, используемые для вывода
HTML.

Шаблоны DIET компилируются с помощью **CTFE**;
они должны быть размещены в папке `views` при использовании
стандартного проекта vibe.d. Для применения шаблона
в обработчике URL используется функция `render`:

    void foo(HTTPServerResponse res) {
        string pageTitle = "Hello";
        int test = 10;
        res.render!("my-template.dt", pageTitle, test);
    }

Все переменные D, доступные в шаблоне DIET,
передаются в виде параметров шаблона `render`.

## {SourceCode:disabled}

```d
doctype html
html
  head
    title D statement test
  body
    - import std.algorithm : min;
    p Four items ahead:
    - foreach( i; 0 .. 4 )
      - auto num = i+1;
      p Item
        b= num
    p Выводится 8:
    p= min(10, 2*6, 8)
```
