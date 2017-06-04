# std.parallelism

Модуль `std.parallelism` реализует высокоуровневые
примитивы для удобной организации параллельного программирования.

### parallel

[`std.parallelism.parallel`](http://dlang.org/phobos/std_parallelism.html#.parallel) позволяет автоматически распределить
тело цикла `foreach` между разными потоками:

    // параллельно возвести в квадрат
    // элементы arr
    auto arr = iota(1,100).array;
    foreach(ref i; parallel(arr)) {
        i = i*i;
    }

Внутри `parallel` используется оператор `opApply`.
Глобальная функция `parallel` - это упрощённый
вариант вызова `taskPool.parallel`. Она применяет
`TaskPool`, использующий количество рабочих потоков,
равное *общему числу процессоров - 1*. Создание
отдельного пула позволяет назначить степень
параллелизма.

Обратите внимание, тело `parallel`-итерации не
должно изменять элементы, к которым может иметь
доступ другая рабочая единица.

Опциональный параметр `workingUnitSize` задаёт
количество элементов, обрабатываемых каждым
потоком.

### reduce

Функция
[`std.algorithm.iteration.reduce`](http://dlang.org/phobos/std_algorithm_iteration.html#reduce),
известная в других функциональных контекстах как *accumulate*
или *foldl*, вызывает функцию `fun(acc, x)` для каждого элемента
`x`, передавая через `acc` предыдущий результат:

    // 0 - начальное значение
    auto sum = reduce!"a + b"(0, elements);

[`Taskpool.reduce`](http://dlang.org/phobos/std_parallelism.html#.TaskPool.reduce) -
это параллельный аналог `reduce`:

    // Вычислить сумму диапазона параллельно, используя
    // первый элемент каждой рабочей единицы как начальное значение
    auto sum = taskPool.reduce!"a + b"(nums);

`TaskPool.reduce` разделяет диапазон на поддиапазоны,
которые обрабатываются параллельно. Как только результаты
от отдельных поддиапазонов становятся известны,
они так же обрабатываются.

### `task()`

[`task`](http://dlang.org/phobos/std_parallelism.html#.task) - это обёртка для функции,
исполнение которой может занять значительное время
или которую следует исполнить в собственном рабочем
потоке. Её можно поставить в очередь пула задач:

    auto t = task!read("foo.txt");
    taskPool.put(t);

или же непосредственно исполнить в отдельном новом потоке:

    t.executeInNewThread();

Чтобы получить результат задачи, необходимо вызвать `yieldForce`.
Этот вызов блокирует текущий поток до тех пор, пока
не будет получен результат.

    auto fileData = t.yieldForce;

### Подробнее

- [Parallelism in _Programming in D_](http://ddili.org/ders/d.en/parallelism.html)
- [std.parallelism](http://dlang.org/phobos/std_parallelism.html)

## {SourceCode}

```d
import std.parallelism : task,
    taskPool, TaskPool;
import std.array : array;
import std.stdio : writeln;
import std.range : iota;

string theTask()
{
    import core.thread : dur, Thread;
    Thread.sleep( dur!("seconds")(1) );
    return "Hello World";
}

void main()
{
    // пул задач с двумя потоками
    auto myTaskPool = new TaskPool(2);
    // Важно не забыть остановить пул!
    scope(exit) myTaskPool.stop();

    // Начать длительную задачу,
    // и тем временем делать что-нибудь ешё..
    auto task = task!theTask;
    myTaskPool.put(task);

    auto arr = iota(1, 10).array;
    foreach(ref i; myTaskPool.parallel(arr)) {
        i = i*i;
    }

    writeln(arr);

    import std.algorithm.iteration : map;

    // Используем reduce для параллельного
    // вычисления суммы квадратов.
    auto result = taskPool.reduce!"a+b"(
        0.0, iota(100).map!"a*a");
    writeln("Sum of squares: ", result);

    // Получить результат задачи, фоновое
    // исполнение которой мы начали ранее.
    writeln(task.yieldForce);
}
```
