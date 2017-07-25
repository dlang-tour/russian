# Исключения

Это руководство только по пользовательским прерываниям (`Exceptions`) - системные
ошибки (`Errors`) обычно непоправимы и __никогда__ не должны отлавливаться.

### Перехват исключений

Распространённый случай для использования исключений - это проверка потенциально
некорректного ввода пользовательских данных. При возникновении исключения стек
будет развёрнут до первого найденного обработчика исключений.
```d
try
{
    readText("dummyFile");
}
catch (FileException e)
{
    // ...
}
```

Можно иметь несколько блоков `catch` и блок `finally`, который выполняется независимо
от того, произошла ли ошибка. Исключения инициируются с помощью `throw`.

```d
try
{
    throw new StringException("You shall not pass.");
}
catch (FileException e)
{
    // ...
}
catch (StringException e)
{
    // ...
}
finally
{
    // ...
}
```

Помните, что [стражи области видимости](gems/scope-guards) обычно являются более подходящим решением шаблона `try-finally`.

### Пользовательские исключения

Можно легко наследоваться от `Exception` и создавать пользовательские исключения:

```d
class UserNotFoundException : Exception
{
    this(string msg, string file = __FILE__, size_t line = __LINE__) {
        super(msg, file, line);
    }
}
throw new UserNotFoundException("D-Man is on vacation");
```

### Войдите в безопасный мир с `nothrow`

Компилятор D может следить за тем, чтобы функция не стала причиной катастрофических побочных эффектов.
Такие функции могут быть объявлены с ключевым словом `nothrow`. Компилятор D статически
запрещает инициировать исключения в `nothrow` функциях.

```d
bool lessThan(int a, int b) nothrow
{
    writeln("unsafe world"); // вывод может инициировать исключения, поэтому такое запрещено
    return a < b;
}
```

Обратите внимание, что компилятор способен автоматически выводить атрибуты шаблонного кода.

### std.exception

Важно избегать контрактного программирования для пользовательского ввода, так как
контракты удаляются при компиляции в режиме Release.
Для удобства, `std.exception` предоставляет метод `enforce`, который может быть
использован подобно `assert`'ам, но инициирует `Exception`'ы вместо `AssertError`.

```d
import std.exception : enforce;
float magic = 1_000_000_000;
enforce(magic + 42 - magic == 42, "Floating-point math is fun");

// Инициирование пользовательского исключения
enforce!StringException('a' != 'A', "Case-sensitive algorithm");
```

Однако, в `std.exception` есть не только это. Например, когда ошибка может быть не фатальной,
исключение может быть "накоплено" (`collect`):

```d
import std.exception : collectException;
auto e = collectException(aDangerousOperation());
if (e)
    writeln("The dangerous operation failed with ", e);
```

Для проверки, было ли исключение в тестах, используйте `assertThrown`.

### Подробнее

- [Безопасность исключений в D](https://dlang.org/exception-safe.html)
- [std.exception](https://dlang.org/phobos/std_exception.html)
- system-level [core.exception](https://dlang.org/phobos/core_exception.html)
- [object.Exception](https://dlang.org/library/object/exception.html) - основной класс всех исключений, которые можно безопасно ловить и обрабатывать.

## {SourceCode}

```d
import std.file : FileException, readText;
import std.stdio : writeln;

void main()
{
    try
    {
        readText("dummyFile");
    }
    catch (FileException e)
    {
        writeln("Message:\n", e.msg);
        writeln("File: ", e.file);
        writeln("Line: ", e.line);
        writeln("Stack trace:\n", e.info);

        // Форматирование по умолчанию также
        // могло быть использовано 
        // writeln(e);
    }
}
```
