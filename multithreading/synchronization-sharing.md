# Синхронизация и разделение данных

Хотя в D предпочтительным методом организации
многопоточности является использование `immutable`
данных и синхронизация посредством обмена сообщениями,
язык так же предлагает встроенную поддержку примитивов
*синхронизации*, и поддержку на уровне типов, с применением
ключевого слова `shared` для обозначения объектов,
доступ к которым осуществляется из нескольких потоков.

Квалификатор типа `shared` позволяет пометить переменные,
разделяемые между различными потоками:

    shared(int)* p = new int;
    int* t = p; // ОШИБКА

Например, функция `std.concurrency.send` позволяет передавать
только `immutable` или `shared` данные, и копирует сообщение
перед отправкой. `shared` транзитивно, и если класс или структура
помечены `shared`, все их члены так же будут помечены.
Обратите внимание, `static`-переменные по умолчанию не
являются `shared`, т.к. хранятся в локальном хранилище потока
(TLS), и каждый поток получает собственную переменную.

Блоки `synchronized` позволяют указать компилятору необходимость
создания критической секции, в которую потоки могут входить
только по одному.

    synchronized {
        importStuff();
    }

При определении функций-членов класса, такие
блоки можно разграничить посредством членов-*мьютексов*,
с использованием `synchronized(член1, член2)`,
для снижения нагрузки на ресурс. Компилятор D
автоматически вставит критические секции. Класс
можно так же целиком пометить как `synchronized`,
и в этом случае компилятор гарантирует, что только
один поток имеет доступ к объекту в отдельно взятый
момент.

Атомарные операции над `shared`-переменными
можно производить посредством вспомогательной
функции `core.atomic.atomicOp`:

    shared int test = 5;
    test.atomicOp!"+="(4);

### В деталях

- [Data Sharing Concurrency in _Programming in D_](http://ddili.org/ders/d.en/concurrency_shared.html)
- [`shared` type qualifier](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=11)
- [Lock-Based Synchronization with `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=13)
- [Deadlocks and `synchronized`](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=15)
- [`synchronized` specification](https://dlang.org/spec/statement.html#SynchronizedStatement)
- [Implicit conversions with `shared` data types](https://dlang.org/spec/const3.html#implicit_conversions)

## {SourceCode}

```d
import std.concurrency : receiveOnly, send,
    spawn, Tid, thisTid;
import core.atomic : atomicOp, atomicLoad;

/*
Очередь, которую можно безопасно использовать
несколькими потоками. Доступ к объекту
блокируется автоматически, благодаря
ключевому слову synchronized.
*/
synchronized class SafeQueue(T)
{
    // Внимание: данные должны быть приватными
    // в synchonized-классах, иначе компилятор
    // выдаст ошибку.
    private T[] elements;

    void push(T value) {
        elements ~= value;
    }

    /// Возвращает T.init, если очередь пуста
    T pop() {
        import std.array : empty;
        T value;
        if (elements.empty)
            return value;
        value = elements[0];
        elements = elements[1 .. $];
        return value;
    }
}

/*
Безопасный вывод сообщений, вне зависимости
от количества одновременно исполняемых потоков.
Обратите внимание, через args передаётся
переменное количество аргументов. То есть
функцию можно вызвать с любым количеством
параметров 0 .. N.
*/
void safePrint(T...)(T args)
{
    // Исполняется по одному
    synchronized {
        import std.stdio : writeln;
        writeln(args);
    }
}

void threadProducer(shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    import std.range : iota;
    // Добавить значения от 1 до 11
    foreach (i; iota(1,11)) {
        queue.push(i);
        safePrint("Pushed ", i);
        atomicOp!"+="(*queueCounter, 1);
    }
}

void threadConsumer(Tid owner,
    shared(SafeQueue!int) queue,
    shared(int)* queueCounter)
{
    int popped = 0;
    while (popped != 10) {
        auto i = queue.pop();
        if (i == int.init)
            continue;
        ++popped;
        // безопасно извлечь текущее значение
        // queueCounter, используя atomicLoad
        safePrint("Popped ", i,
            " (Consumer pushed ",
            atomicLoad(*queueCounter), ")");
    }

    // Дело сделано!
    owner.send(true);
}

void main()
{
    auto queue = new shared(SafeQueue!int);
    shared int counter = 0;
    spawn(&threadProducer, queue, &counter);
    auto consumer = spawn(&threadConsumer,
                    thisTid, queue, &counter);
    auto stopped = receiveOnly!bool;
    assert(stopped);
}
```
