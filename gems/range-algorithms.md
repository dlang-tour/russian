# Алгоритмы для диапазонов

Стандартные модули [std.range](http://dlang.org/phobos/std_range.html)
и [std.algorithm](http://dlang.org/phobos/std_algorithm.html)
предоставляют плеяду отличных функций, которые
можно комбинировать для создания комплексных
операций, без потери читаемости - на основе
*диапазонов*.

Отличительная черта таких алгоритмов в том,
что Вам лишь требуется определить собственный
диапазон, и Вы сможете использовать все возможности
стандартной библиотеки.

### std.algorithm

`filter` - Используя лямбда-выражение, передаваемое как
 параметр шаблона, генерирует новый диапазон,
 фильтрующий элементы:

    filter!"a > 20"(range);
    filter!(a => a > 20)(range);

`map` - Генерирует новый диапазон, используя предикат,
 передаваемый как параметр шаблона:

    [1, 2, 3].map!(x => to!string(x));

`each` - `foreach` "для бедных" - функция, итерирующая
диапазон:

    [1, 2, 3].each!(a => writeln(a));

### std.range
`take` - Взять *N* элементов:

    theBigBigRange.take(10);

`zip` - одновременно итерирует два диапазона,
возвращает tuple - пары элементов из обоих
диапазонов:

    assert(zip([1,2], ["hello","world"]).front
      == tuple(1, "hello"));

`generate` - создаёт диапазон, вызывая переданную
функцию в каждой итерации:

    alias RandomRange = generate!(x => uniform(1, 1000));

`cycle` - возвращает диапазон, бесконечно повторяющий
переданный диапазон ввода:

    auto c = cycle([1]);
    // диапазон никогда не будет пуст!
    assert(!c.empty);

### Документация ожидает Вашего визита!


### В деталях

- [Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges.html)
- [More Ranges in _Programming in D_](http://ddili.org/ders/d.en/ranges_more.html)

## {SourceCode}

```d
// А, давайте сразу целую армию!
import std.algorithm : canFind, map,
  filter, sort, uniq, joiner, chunkBy, splitter;
import std.array : array, empty;
import std.range : zip;
import std.stdio : writeln;
import std.string : format;

void main()
{
    string text = q{Этот тур предоставит Вам
обзор этого мощного и выразительного языка
программирования системного уровня, компилируемого
непосредственно в *нативный* машинный код.};

    // предикат-разделитель
    alias pred = c => canFind(" ,.\n", c);
    // как всякий хороший алгоритм,
    // работает лениво и не выделяет память!
    auto words = text.splitter!pred
      .filter!(a => !a.empty);

    auto wordCharCounts = words
      .map!"a.count";

    // Вывести количество символов на
    // слово, при этом читаемо,
    // и начиная с наименьшего количества!
    zip(wordCharCounts, words)
      // преобразовать в массив для упорядочивания
      .array()
      .sort()
      // дубликаты ведь нам не нужны, так?
      .uniq()
      // всё с одинаковым количеством символов
      // поместить в один ряд.
      // chunkBy поможет сгенерировать диапазоны
      // диапазонов, разделённые по длине
      .chunkBy!(a => a[0])
      // эти элементы будут объединены в одну строку
      .map!(chunk => format("%d -> %s",
          chunk[0],
          // только слова
          chunk[1]
            .map!(a => a[1])
            .joiner(", ")))
      // joiner объединяет, но лениво!
      // а теперь строки и перевод строки
      .joiner("\n")
      // передать в stdout
      .writeln();
}
```
