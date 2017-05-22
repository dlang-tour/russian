# opDispatch и opApply

D позволяет переопределять операторы, такие как
`+`, `-` или оператор вызова `()` для
[классов и структур](https://dlang.org/spec/operatoroverloading.html).
Взглянем поближе на переопределение специальных операторов
`opDispatch` и `opApply`.

### opDispatch

`opDispatch` можно определить как функцию-член для типов
`struct` или `class`. Все неизвестные вызовы членов такого типа
передаются в `opDispatch`, при этом имя неизвестного члена
передаётся как параметр шаблона типа `string`.
`opDispatch` - это  *перехватывающая всё*
функция-член, предоставляющая дополнительный уровень
обобщённого программирования - полностью **во время компиляции**!

    struct C {
        void callA(int i, int j) { ... }
        void callB(string s) { ... }
    }
    struct CallLogger(C) {
        C content;
        void opDispatch(string name, T...)(T vals) {
            writeln("вызов ", name);
            mixin("content." ~ name)(vals);
        }
    }
    CallLogger!C l;
    l.callA(1, 2);
    l.callB("ABC");

### opApply

Альтернативный способ реализации итерирования посредством
`foreach` без необходимости определять пользовательский
*диапазон* - реализация функции-члена `opApply`.
Итерирование такого типа при помощи `foreach` приводит
к вызову `opApply` с параметром в виде специального делегата:

    class Tree {
        Tree lhs;
        Tree rhs;
        int opApply(int delegate(Tree) dg) {
            if (lhs && lhs.opApply(dg)) return 1;
            if (dg(this)) return 1;
            if (rhs && rhs.opApply(dg)) return 1;
            return 0;
        }
    }
    Tree tree = new Tree;
    foreach(node; tree) {
        ...
    }

Компилятор преобразует тело `foreach` в специальный
делегат, который передаёт объекту. Единственный его
параметр - индекс текущей итерации.
"Волшебное" возвращаемое значение типа `int`
необходимо интерпретировать, и, в случае, если это не `0`,
прекратить цикл.

### В деталях

- [Operator overloading in _Programming in D_](http://ddili.org/ders/d.en/operator_overloading.html)
- [`opApply` in _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [Operator overloading in D](https://dlang.org/spec/operatoroverloading.html)

## {SourceCode}

```d
/*
Variant - объект, способный хранить любой другой
тип:
https://dlang.org/phobos/std_variant.html
*/

import std.variant : Variant;

/*
Тип, который может иметь произвольное число
членов, благодаря opDispatch.
Подобие var в JavaScript.
*/
struct var {
    private Variant[string] values;

    @property
    Variant opDispatch(string name)() const {
        return values[name];
    }

    @property
    void opDispatch(string name, T)(T val) {
        values[name] = val;
    }
}

void main() {
    import std.stdio : writeln;

    var test;
    test.foo = "тест";
    test.bar = 50;
    writeln("test.foo = ", test.foo);
    writeln("test.bar = ", test.bar);
    test.foobar = 3.1415;
    writeln("test.foobar = ", test.foobar);
    // ОШИБКА, т.к. такой член ещё не существует
    // writeln("test.notthere = ",
    //   test.notthere);
}
```
