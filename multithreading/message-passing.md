# Обмен сообщениями

Взамен работы с потоками и выполнения синхронизации
вручную, D позволяет использовать *обмен сообщениями*
как средство эксплуатации мощности нескольких ядер.
Потоки обмениваются *сообщениями* - произвольными значениями -
для распределения работы и самосинхронизации. Согласно
такому дизайну, между потоками нет разделяемых данных,
что позволяет избежать проблем, часто встречающихся при
организации многопоточности.

Все функции, реализующие обмен сообщениями в D,
находятся в модуле [`std.concurrency`](https://dlang.org/phobos/std_concurrency.html).
`spawn` создаёт новый *поток* на основе пользовательской
функции:

    auto threadId = spawn(&foo, thisTid);

`thisTid` - переменная, встроенная в  `std.concurrency`,
это идентификатор текущего потока, необходимый для обмена сообщениями.
`spawn` первым параметром принимает функцию, а через дополнительные
параметры передаются аргументы этой функции.

    void foo(Tid parentTid) {
        receive(
          (int i) { writeln("An ", i, " was sent!"); }
        );
        
        send(parentTid, "Done");
    }

Функция `receive` - своего рода `switch`-`case`,
она распределяет значения, полученные от других потоков,
между переданными ей `делегатами`, в зависимости от
типа полученного значения.

Чтобы послать сообщение определённому потоку, используйте
функцию `send` и идентификатор потока:

    send(threadId, 42);

`receiveOnly` можно использовать для получения сообщений
строго определённого типа:

    string text = receiveOnly!string();
    assert(text == "Done");

Семейство функций `receive` блокирует поток до тех пор,
пока что-либо не будет передано в его почтовый ящик.


### Подробнее

- [Exchanging Messages between Threads](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=5)
- [Messaging passing concurrency](http://ddili.org/ders/d.en/concurrency.html)
- [Pattern Matching with receive](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=6)
- [Multi-threaded file copying](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=7)
- [Thread Termination](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=8)
- [Out-of-Band Communication](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=9)
- [Mailbox crowding](http://www.informit.com/articles/article.aspx?p=1609144&seqNum=10)

## {SourceCode}

```d
import std.stdio : writeln;
import std.concurrency : receive, receiveOnly,
    send, spawn, thisTid, Tid;

/*
Пользовательская структура, используемая как
сообщение для небольшой армии потоков.
*/
struct NumberMessage {
    int number;
    this(int i) {
        this.number = i;
    }
}

/*
Это сообщение используется как знак "стоп"
для других потоков.
*/
struct CancelMessage {
}

/// Подтверждает принятие CancelMessage
struct CancelAckMessage {
}

/*
Основная рабочая функция потока,
которой аргументом передаётся идентификатор
родительского потока.
*/
void worker(Tid parentId)
{
    bool canceled = false;
    writeln("Starting ", thisTid, "...");

    while (!canceled) {
      receive(
        (NumberMessage m) {
          writeln("Received int: ", m.number);
        },
        (string text) {
          writeln("Received string: ", text);
        },
        (CancelMessage m) {
          writeln("Stopping ", thisTid, "...");
          send(parentId, CancelAckMessage());
          canceled = true;
        }
      );
    }
}

void main()
{
    Tid threads[];
    // Породить 10 рабочих потоков.
    for (size_t i = 0; i < 10; ++i) {
        threads ~= spawn(&worker, thisTid);
    }

    // Нечётным потокам передаётся число,
    // чётным - строка!
    foreach(int idx, ref tid; threads) {
        import std.string : format;
        if (idx  % 2)
            send(tid, NumberMessage(idx));
        else
            send(tid, format("T=%d", idx));
    }

    // И все потоки получают стоп-сигнал
    foreach(ref tid; threads) {
        send(tid, CancelMessage());
    }

    // Подождём, пока все потоки не подтвердили
    // получение запроса на останов
    foreach(ref tid; threads) {
        receiveOnly!CancelAckMessage;
        writeln("Received CancelAckMessage!");
    }
}
```
