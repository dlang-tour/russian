# Управление потоком

Ход выполнения приложения можно контролировать с помощью условных операторов `if` и `else`:

    if (a == 5) {
        writeln("Condition is met");
    } else if (a > 10) {
        writeln("Another condition is met");
    } else {
        writeln("Nothing is met!");
    }

Если блок `if` или `else` содержит только одну команду, фигурные скобки могут
быть опущены.

Для проверки равенства переменных и их сравнения D предоставляет такие же
операторы, как и C/C++ и Java:

* `==` и `!=` для проверки равенства и неравенства
* `<`, `<=`, `>` и `>=` для проверки того, что значение меньше (- или равно) и
больше (- или равно)

Условия можно комбинировать, используя операторы `||` (логическое *ИЛИ*) и `&&` (логическое *И*).

В D есть конструкция `switch`..`case`, которая выбирает одну ветвь исполнения, в зависимости от значения *одной* переменной. `switch` работает со всеми основными типами, а также со строками. Для целочисленных типов возможно задавать диапазоны, используя синтаксис `case НАЧАЛО: .. case КОНЕЦ:`. Посмотрите, пожалуйста, пример кода.

### Подробнее

#### Основные ссылки

- [Логические выражение в _Programming in D_](http://ddili.org/ders/d.en/logical_expressions.html)
- [Условный оператор if в _Programming in D_](http://ddili.org/ders/d.en/if.html)
- [Тернарный оператор ?: в _Programming in D_](http://ddili.org/ders/d.en/ternary.html)
- [`switch` и `case` в _Programming in D_](http://ddili.org/ders/d.en/switch_case.html)

#### Дополнительные ссылки

- [Спецификация на выражения](https://dlang.org/spec/expression.html)
- [Спецификация на оператор if](https://dlang.org/spec/statement.html#if-statement)

## {SourceCode}

```d
import std.stdio;

void main()
{
    if (1 == 1)
        writeln("You can trust math in D");

    int c = 5;
    switch(c) {
        case 0: .. case 9:
            writeln(c, " is within 0-9");
            break; // необходимо!
        case 10:
            writeln("A Ten!");
            break;
        default: 
        // если ни одно из условий
        // не выполнилось
            writeln("Nothing");
            break;
    }
}
```
