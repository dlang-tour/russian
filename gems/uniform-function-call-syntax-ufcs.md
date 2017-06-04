# Унифицированный синтаксис вызова функций (UFCS)

**UFCS** - ключевая особенность D, открывающая новые возможности
повторного использования и масштабирования кода посредством
грамотно определённой инкапсуляции.

UFCS гласит, что любой вызов свободной функции
`fun(a)` может быть записан как вызов функции-члена `a.fun()`.

Если компилятор встречает `a.fun()`, и у типа отсутствует функция-член
`fun()`, он попробует найти глобальные функции, чей первый параметр
совпадает с типом `a`.

Эта особенность очень полезна в сложных цепочках вызовов.
Вместо того, чтобы писать

    foo(bar(a))

Можно написать

    a.bar().foo()

Кроме того, в D отсутствует необходимость использования скобок
для функций, не имеющих аргументов, а это означает, что _любую_
функцию можно использовать в качестве свойства:

    import std.uni : toLower;
    "D рулит".toLower; // "d рулит"

UFCS особенно важен при работе с *диапазонами*,
где можно совместить несколько алгоритмов для
выполнения сложных операций, при этом используя
понятный и поддерживаемый код.

    import std.algorithm : group;
    import std.range : chain, retro, front, retro;
    [1, 2].chain([3, 4]).retro; // 4, 3, 2, 1
    [1, 1, 2, 2, 2].group.dropOne.front; // tuple(2, 3u)

### Подробнее

- [UFCS in _Programming in D_](http://ddili.org/ders/d.en/ufcs.html)
- [_Uniform Function Call Syntax_](http://www.drdobbs.com/cpp/uniform-function-call-syntax/232700394) by Walter Bright
- [`std.range`](http://dlang.org/phobos/std_range.html)

## {SourceCode}

```d
import std.stdio : writefln, writeln;
import std.algorithm.iteration : filter;
import std.range : iota;

void main()
{
    "Hello, %s".writefln("World");

    10.iota // возвращает числа от 0 до 9
      // отфильтровать только чётные
      .filter!(a => a % 2 == 0)
      .writeln(); // вывести в stdout

    // Традиционный стиль:
    writeln(filter!(a => a % 2 == 0)
    			   (iota(10)));
}
```
