# Циклы

В D есть четыре конструкции для организации циклов.

### 1) Классический цикл `for`

Классический цикл `for`, известный из C/C++ или Java, с
_инициализатором_, _условием цикла_ и _оператором цикла_:

    for (int i = 0; i < arr.length; ++i) {
        ...

### 2) `while`

Цикл `while` выполняет блок кода, пока верно заданное условие:

    while (условие) {
        foo();
    }

### 3) `do ... while`

Цикл `do .. while` выполняет код, пока верно заданное условие, но, в
отличие от `while`, _тело цикла_ выполняется до того, как проверяется условие.

    do {
        foo();
    } while (условие);

#### 4) `foreach`

[Цикл `foreach`](basics/foreach), который будет рассмотрен в следующем разделе.

#### Специальные ключевые слова и метки

Специальное ключевое слово `break` немедленно прерывает текущий цикл.
Во вложенных циклах _метку_ (label) можно использовать для прерывания
любого внешнего цикла:

    outer: for (int i = 0; i < 10; ++i) {
        for (int j = 0; j < 5; ++j) {
            ...
            break outer;

Ключевое слово `continue` запускает следующую итерацию цикла.

### Подробнее

- Цикл `for` в [_Programming in D_](http://ddili.org/ders/d.en/for.html), [спецификация](https://dlang.org/spec/statement.html#ForStatement)
- Цикл `while` в [_Programming in D_](http://ddili.org/ders/d.en/while.html), [спецификация](https://dlang.org/spec/statement.html#WhileStatement)
- Цикл `do-while` в [_Programming in D_](http://ddili.org/ders/d.en/do_while.html), [спецификация](https://dlang.org/spec/statement.html#do-statement)

## {SourceCode}

```d
import std.stdio;

/// Вычисляет среднее число элементов массива.
double average(int[] array) {
    // Свойство .empty для массивов не является
    // "родным" в D, но доступ к нему можно
    // получить, импортировав функцию
    // из std.array.
    import std.array: empty, front;

    double accumulator = 0.0;
    auto length = array.length;
    while (!array.empty) {
        // Также это можно сделать с помощью
        // .front, импортировав std.array:front;
        accumulator += array[0];
        array = array[1 .. $];
    }

    return accumulator / length;
}

void main()
{
    auto testers = [ [5, 15], // 20
          [2, 3, 2, 3], // 10
          [3, 6, 2, 9] ]; // 20

    for (auto i = 0; i < testers.length; ++i) {
      writeln("The average of ", testers[i],
        " = ", average(testers[i]));
    }
}
```
