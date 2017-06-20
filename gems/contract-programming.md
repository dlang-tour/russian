# Контрактное программирование

Контрактное программирование в D включает в себя набор
конструкций языка, позволяющий повысить качество кода
путём проверок "разумности" - тестов, удостоверяющихся,
что код работает согласно изначальным намерениям.
Контракты доступны только в режиме **отладки**, и не
исполняются в релизном режиме. Поэтому они не должны
использоваться для проверки пользовательского ввода.

### `assert`

Простейшая форма контрактного программирования в D -
выражение `assert(...)`, которое проверяет, что заданное
условие выполнено, и останавливает программу посредством
`AssertionError` в противном случае.

    assert(sqrt(4) == 2);
    // дополнительно, можно указать
    // собственное сообщение
    assert(sqrt(16) == 4, "sqrt не работает!");

### Контракты функций

`in` и `out` позволяют формализовать контракты для
параметров и возвращаемых значений функций.

    long square_root(long x)
    in {
        assert(x >= 0);
    } out (result) {
        assert((result * result) <= x
            && (result+1) * (result+1) > x);
    } body {
        return cast(long)std.math
                            .sqrt(cast(real)x);
    }

Содержимое блока `in` можно перенести в тело функции,
однако при использовании такого блока намерения выражаются
более чётко. Получить возвращаемое значение в блоке `out`,
чтобы затем проверить его верность, можно выражением
`out(result)`.

### Проверка инвариантов

`invariant()` - специальная функция-член структур
и классов, позволяющая контролировать состояние
объекта всё время его жизни:

* Вызывается после исполнения конструктора и перед
  исполнением деструктора.
* Вызывается перед исполнением функции-члена.
* Вызывается после исполнения функции-члена.

### Проверка пользовательского ввода

Поскольку все контракты удаляются в релизной сборке, пользовательский ввод
нельзя проверять с помощью контрактов. `assert`'ы можно использовать
в `nothrow`-функциях, поскольку они вызывают "фатальные" ошибки (`Error`).
Аналог времени исполнения для `assert` - [`std.exception.enforce`](https://dlang.org/phobos/std_exception.html#.enforce),
вызывающий исключения (`Exceptions`), которые разрешено ловить.

### Подробнее

- [`assert` and `enforce` in _Programming in D_](http://ddili.org/ders/d.en/assert.html)
- [Contract programming in _Programming in D_](http://ddili.org/ders/d.en/contracts.html)
- [Contract Programming for Structs and Classes in _Programming in D_](http://ddili.org/ders/d.en/invariant.html)
- [Contract programming in D spec](https://dlang.org/spec/contracts.html)
- [`std.exception`](https://dlang.org/phobos/std_exception.html)

## {SourceCode:incomplete}

```d
import std.stdio : writeln;

/**
Упрощённый тип Date
Вместо него используйте std.datetime
*/
struct Date {
    private {
        int year;
        int month;
        int day;
    }

    this(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    invariant() {
        assert(year >= 1900);
        assert(month >= 1 && month <= 12);
        assert(day >= 1 && day <= 31);
    }

    /**
    Сериализует объект типа Date из строки
    в формате ГГГГ-ММ-ДД.
    
    Params:
        date = сериализуемая строка
        
    Returns: Объект типа Date.
    /*
    void fromString(string date)
    in {
        assert(date.length == 10);
    }
    body {
        import std.format : formattedRead;
        // formattedRead обрабатывает
        // форматирующую строку,
        // и возвращает результат
        // через переменные
        formattedRead(date, "%d-%d-%d",
            &this.year,
            &this.month,
            &this.day);
    }

    /**
    Сериализует объект типа Date в строку
    в формате ГГГГ-ММ-ДД

    Returns: Строковое представление Date
    */
    string toString() const
    out (result) {
        import std.algorithm : all, count,
                              equal, map;
        import std.string : isNumeric;
        import std.array : split;

        // удостовериться, что возвращается
        // ГГГГ-ММ-ДД
        assert(result.count("-") == 2);
        auto parts = result.split("-");
        assert(parts.map!`a.length`
                    .equal([4, 2, 2]));
        assert(parts.all!isNumeric);
    }
    body {
        import std.format : format;
        return format("%.4d-%.2d-%.2d",
                      year, month, day);
    }
}

void main() {
    auto date = Date(2016, 2, 7);

    // Такой ввод вызовет ошибку в инварианте.
    // Не проверяйте пользовательский ввод
    // контрактами, вместо этого
    // пользуйтесь исключениями.
    date.fromString("2016-13-7");

    date.writeln;
}
```
