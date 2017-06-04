# Типажи (трейты)

Залог мощи D - его система исполнения функций во время компиляции (CTFE).
Благодаря интроспекции можно писать обобщённые программы и добиваться
серьёзной оптимизации.

## Явные контракты

Типажи помогают явно указать, какой ввод является приемлемым.
К примеру, функция `splitIntoWords` может работать с любым строковым
типом:

```d
S[] splitIntoWord(S)(S input)
if (isSomeString!S)
```

Это применимо и к параметрам шаблонов. `myWrapper` проверяет, что
переданный символ можно вызывать (как функцию):

```d
void myWrapper(alias f)
if (isCallable!f)
```

Проанализируем простой пример - [`commonPrefix`](https://dlang.org/phobos/std_algorithm_searching.html#.commonPrefix)
из `std.algorithm.searching`, возвращающий общий префикс двух диапазонов:

```d
auto commonPrefix(alias pred = "a == b", R1, R2)(R1 r1, R2 r2)
if (isForwardRange!R1
    isInputRange!R2 &&
    is(typeof(binaryFun!pred(r1.front, r2.front)))) &&
    !isNarrowString!R1)
```

Вызов такой функции скомпилируется только если:

- `r1` можно сохранить (гарантируется `isForwardRange`)
- `r2` можно итерировать (гарантируется `isInputRange`)
- `pred` можно вызвать с параметрами типов элементов `r1` and `r2`
- `r1` - не "узкая" строка (`char[]`, `string`, `wchar` или `wstring`) - для простоты, иначе может потребоваться декодирование

### Специализация

Многие API стремятся быть расширить область их применения,
однако при этом не хотят расплачиваться
производительностью за такую обобщённость.

При помощи мощных механизмов интроспекции и CTFE становится возможным
специализировать метод во время компиляции для достижения наилучшей
производительности для конкретных типов.

Часто встречающаяся проблема - неизвестная заранее длина потока или списка.
В `std.range` определён простой метод `walkLength`, обобщённый для
любого итерируемого типа:

```d
static if (hasMember!(r, "length"))
    return r.length; // O(1)
else
    return r.walkLength; // O(n)
```

#### `commonPrefix`

Интроспекция во время компиляции применяется в Phobos повсеместно.
К примеру, `commonPrefix` различает диапазоны со случайным доступом
(`RandomAccessRange`) и линейно-итерируемые диапазоны, поскольку
при наличии случайного доступа можно свободно перепрыгивать с места
на место, за счёт чего ускорить алгоритм.

#### Больше магии CTFE

[std.traits](https://dlang.org/phobos/std_traits.html) - оболочка вокруг
большинства [типажей D](https://dlang.org/spec/traits.html), за исключением
некоторых, например `compiles`, который невозможно "обернуть":

```d
__traits(compiles, obvious error - $%42); // false
```

#### Специальные ключевые слова

В отладочных целях D предоставляет несколько специальных ключевых слов:

```d
void test(string file = __FILE__, size_t line = __LINE__, string mod = __MODULE__,
          string func = __FUNCTION__, string pretty = __PRETTY_FUNCTION__)
{
    writefln("file: '%s', line: '%s', module: '%s',\nfunction: '%s', pretty function: '%s'",
             file, line, mod, func, pretty);
}
```

А благодаря вычислениям из командной строки, можно забыть про `time` -
будет использовано CTFE!

```d
rdmd --force --eval='pragma(msg, __TIMESTAMP__);'
```

## Подробнее

- [std.range.primitives](https://dlang.org/phobos/std_range_primitives.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [std.meta](https://dlang.org/phobos/std_meta.html)
- [Specification on Traits in D](https://dlang.org/spec/traits.html)

## {SourceCode}

```d
import std.functional : binaryFun;
import std.range.primitives : empty, front,
    popFront,
    isInputRange,
    isForwardRange,
    isRandomAccessRange,
    hasSlicing,
    hasLength;
import std.stdio : writeln;
import std.traits : isNarrowString;

/**
Возвращает общий префикс двух диапазонов,
особый случай с авто-декодированием
не рассматривается.

Params:
    pred = Предикат для определения "общности"
    r1 = Диапазон forward range.
    r2 = Диаразон input range.

Returns:
Срез r1, содержащий символы,
одинаковые в начале обоих диапазонов.
 */
auto commonPrefix(alias pred = "a == b", R1, R2)
                 (R1 r1, R2 r2)
if (isForwardRange!R1 && isInputRange!R2 &&
    !isNarrowString!R1 &&
    is(typeof(binaryFun!pred(r1.front,
                             r2.front))))
{
    import std.algorithm.comparison : min;
    static if (isRandomAccessRange!R1 &&
               isRandomAccessRange!R2 &&
               hasLength!R1 && hasLength!R2 &&
               hasSlicing!R1)
    {
        immutable limit = min(r1.length,
                              r2.length);
        foreach (i; 0 .. limit)
        {
            if (!binaryFun!pred(r1[i], r2[i]))
            {
                return r1[0 .. i];
            }
        }
        return r1[0 .. limit];
    }
    else
    {
        import std.range : takeExactly;
        auto result = r1.save;
        size_t i = 0;
        for (;
             !r1.empty && !r2.empty &&
             binaryFun!pred(r1.front, r2.front);
             ++i, r1.popFront(), r2.popFront())
        {}
        return takeExactly(result, i);
    }
}

void main()
{
    // выводит: "hello, "
    writeln(commonPrefix("hello, world"d,
                         "hello, there"d));
}
```
