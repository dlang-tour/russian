# Волокна

**Волокна** - это метод реализации одновременного
исполнения в *кооперативном* стиле. Класс `Fiber`
определён в модуле [`core.thread`](https://dlang.org/phobos/core_thread.html).

Основная идея состоит в том, что когда
волокну нечего исполнять, или оно ожидает
нового ввода, оно *активно* передаёт возможность
исполнения инструкций посредством вызова
`Fiber.yield()`. Родительский контекст
вновь получает управление, но состояние волокна -
все переменные в стеке - сохраняется. В дальнейшем
волокно можно возобновить, и оно продолжит исполняться
с инструкции, *следующей* за вызовом `Fiber.yield()`.
Магия? Да.

    void foo() {
        writeln("Hello");
        Fiber.yield();
        writeln("World");
    }
    // ...
    auto f = new Fiber(&foo);
    f.call(); // Выводит Hello
    f.call(); // Выводит World

Эту возможность можно использовать для реализации
одновременного исполнения, при котором несколько
волокон в кооперативном режиме работают на одном
ядре. Преимуществом волокон перед потоками является
то, что объём потребляемых ими ресурсов меньше, поскольку
не производятся переключения контекста.

Отличное применение данной техники можно увидеть на примере
[фреймворка vibe.d](http://vibed.org), реализующего
неблокирующие (асинхронные) операции ввода-вывода
на основе волокон, предоставляя более чистый код.

### В деталях

- [Fibers in _Programming in D_](http://ddili.org/ders/d.en/fibers.html)
- [Documentation of core.thread.Fiber](https://dlang.org/library/core/thread/fiber.html)

## {SourceCode}

```d
import core.thread : Fiber;
import std.stdio : write;
import std.range : iota;

/**
Итерирует `range` и исполняет
функцию `Fnc` для каждого элемента x,
и возвращает результат через переменную
`result`. Волокно передаёт управление
после каждой итерации.
*/
void fiberedRange(alias Fnc, R, T)(
    R range,
    ref T result)
{
    for(; !range.empty; range.popFront) {
        result = Fnc(range.front);
        Fiber.yield();
    }
}

void main()
{
    int squareResult, cubeResult;
    // Создать волокно, инициализировать его
    // делегатом, который генерирует диапазон
    // квадратов.
    auto squareFiber = new Fiber({
        fiberedRange!(x => x*x)(
            iota(1,11), squareResult);
    });
    // .. а этот возводит в куб!
    auto cubeFiber = new Fiber({
        fiberedRange!(x => x*x*x)(
            iota(1,9), cubeResult);
    });

    // если возвращается состояние TERM,
    // значит волокно завершило исполнение
    // связанной с ним функции.
    squareFiber.call();
    cubeFiber.call();
    while (squareFiber.state
        != Fiber.State.TERM && cubeFiber.state
        != Fiber.State.TERM)
    {
        write(squareResult, " ", cubeResult);
        squareFiber.call();
        cubeFiber.call();
        write("\n");
    }

    // squareFiber можно вызывать ещё, поскольку
    // оно не завершило исполнение!
}
```
