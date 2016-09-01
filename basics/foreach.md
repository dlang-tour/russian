# Foreach

{{#img-right}}dman-teacher-foreach.jpg{{/img-right}}

Особенности `foreach` в D позволяют уменьшить возможность ошибок и улучшить
читаемость кода для перебора элементов.

### Перебор элементов

Для заданного массива `arr` типа `int[]` можно перебрать все его элементы, используя цикл `foreach`:

    foreach (int e; arr) {
        writeln(e);
    }

Первое поле в определении `foreach` - это переменная, используемая для
организации цикла. Её тип вычисляется автоматически:

    foreach (e; arr) {
        // typeof(e) is int
        writeln(e);
    }

Второе поле должно быть массивом, либо специальным поддерживающим перебор объектом, называемом **range** (диапазон), о котором будет рассказано в следующем разделе.

### Доступ по ссылке

Во время перебора элементы массива или диапазона будут скопированы.
Это допустимо для основных типов, но может стать проблемой для больших типов.
Для предотвращения копирования или для того, чтобы изменять оригинальные значения, можно использовать `ref`:

    foreach (ref e; arr) {
        e = 10; // перезаписать значение
    }

### Обратный перебор с помощью `foreach_reverse`

Коллекцию можно перебрать в обратном порядке с помощью `foreach_reverse`:

    foreach_reverse (e; [1, 2, 3]) {
        writeln(e);
    }
    // 3 2 1

### Подробнее

- [`foreach` в _Programming in D_](http://ddili.org/ders/d.en/foreach.html)
- [`foreach` со структурами и классами в _Programming in D_](http://ddili.org/ders/d.en/foreach_opapply.html)
- [Спецификация на `foreach`](https://dlang.org/spec/statement.html#ForeachStatement)

## {SourceCode}

```d
import std.stdio;

void main() {
    auto arr = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    // Перебор массива в обратном порядке
    import std.range: retro;
    foreach (row; retro(arr))
    {
        double accumulator = 0.0;
        foreach (c; row)
            accumulator += c;

        writeln("The average of ", row,
            " = ", accumulator / row.length);
    }
}
```
