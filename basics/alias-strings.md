# Алиас & Строки

Теперь, когда мы знаем, что собой представляют массивы, узнали про `immutable`
и пробежались по основным типам, пора познакомиться сразу с двумя новыми конструкциями:

    alias string = immutable(char)[];

Понятие строка (`string`) задано как инструкция алиас (`alias`, буквально "псевдоним"), которое определяет его как срез `immutable(char)`'ов. Это означает, что как только строка построена, её содержимое больше не может быть изменено. И это одна из тем, с которой познакомит эта глава: строки в кодировке UTF-8.

Из-за своей неизменности строки могут быть прекрасно разделены между разными
потоками. Так как строки являются срезами, их части могут быть взяты без
дополнительного выделения памяти. Например, когда стандартная функция
`std.algorithm.splitter` разделяет строку на части, дополнительная память не выделяется.

Помимо UTF-8 строк, есть ещё два типа:

    alias wstring = immutable(dchar)[]; // UTF-16
    alias dstring = immutable(wchar)[]; // UTF-32

Их проще всего преобразовывать между собой с помощью метода `to` из `std.conv`:

    dstring myDstring = to!dstring(myString);
    string myString   = to!string(myDstring);

Это означает, что строки формата `string` определены как массив 8-битных [значений Юникод](http://unicode.org/glossary/#code_unit). Всё, что можно делать с массивами, можно делать и со строками, но работать это будет на уровне значений Юникод, а не на уровне символов. В то же время алгоритмы из стандартной библиотеки интерпретируют строки как последовательность [элементов Юникод](http://unicode.org/glossary/#code_point), и помимо этого есть способ интерпретировать их как последовательность [графем](http://unicode.org/glossary/#grapheme), явно использовав [`std.uni.byGrapheme`](https://dlang.org/library/std/uni/by_grapheme.html).

Этот небольшой пример покажет разницу в интерпретациях:

    string s = "\u0041\u0308"; // Ä

    writeln(s.length); // 3

    import std.range : walkLength;
    writeln(s.walkLength); // 2

    import std.uni : byGrapheme;
    writeln(s.byGrapheme.walkLength); // 1

В примере выше длина массива `s`, вычисляющаяся с помощью свойства `.length`, равна 3, потому что он содержит три значения Юникод: `0x41`, `0x03` и `0x08`. Из них последние два означают один элемент (диакритический знак, который комбинируется с буквой), и [`walkLength`](https://dlang.org/library/std/range/primitives/walk_length.html) возвращает 2, так как подсчитывается всего два элемента Юникод. `walkLength` - это функция стандартной библиотеки для подсчета длины произвольного диапазона ([range](https://dlang.org/library/std/range.html)). И наконец `byGrapheme` - это довольно дорогостоящее вычисление, показывающее, что эти два элемента представляют собой один видимый символ.

Правильная обработка Юникода может быть очень непростой, но обычно разработчики могут считать строковые переменные просто абстрактным набором байтов и полагаться на алгоритмы стандартной библиотеки. Большая часть функциональности Юникода представлена в модуле [`std.uni`](https://dlang.org/library/std/uni.html), некоторые базовые примитивы доступны в [`std.utf`](https://dlang.org/library/std/utf.html).

Чтобы получить многострочный текст, можно использовать синтаксис
`string str = q{ ... }`.

    string multiline = q{ This
        may be a
        long document
    };

Уменьшить количество трудоёмкого ручного экранирования можно, определяя raw-строки с использованием обратных апострофов (`` ` ... ` ``), либо префикса r(aw) (`r" ... "`).

    string raw  =  `raw "string"`; // raw "string"
    string raw2 = r"raw "string""; // raw "string"

### Подробнее

- [Символы в _Programming in D_](http://ddili.org/ders/d.en/characters.html)
- [Строки в _Programming in D_](http://ddili.org/ders/d.en/strings.html)
- [std.utf](http://dlang.org/phobos/std_utf.html) - Алгоритмы (де)кодирования UTF
- [std.uni](http://dlang.org/phobos/std_uni.html) - Алгоритмы Юникод

## {SourceCode}

```d
import std.stdio;
import std.utf: count;
import std.string: format;

void main() {
    // format генерирует строку, используя
    // подобный printf синтаксис. D позволяет
    // нативную обработку UTF строк.
    string str = format("%s %s", "Hellö",
        "Wörld");
    writeln("My string: ", str);
    writeln("Array length of string: ",
        str.length);
    writeln("Character length of string: ",
        count(str));

    // Строки - это обычные массивы,
    // поэтому любые операции, применимые к
    // массивам, здесь также работают.
    import std.array: replace;
    writeln(replace(str, "lö", "lo"));
    import std.algorithm: endsWith;
    writefln("Does %s end with 'rld'? %s",
        str, endsWith(str, "rld"));

    import std.conv: to;
    // Преобразование в строку UTF-32
    dstring dstr = to!dstring(str);
    // .. которая, конечно же, выглядит так же!
    writeln("My dstring: ", dstr);
}
```
