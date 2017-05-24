# Подтипы

`struct` не может наследовать другие `struct`'ы. Но
для таких несчастных структур D предоставляет
другой мощный механизм для расширения их функциональности -
**подтипы**.

Структура может определить один из своих членов как
`alias this`:

    struct SafeInt {
        private int theInt;
        alias theInt this;
    }

Любая функция или операция над `SafeInt`, которая
не может быть применена к самому типу, будет переадресована
члену, объявленному как `alias this`. Таким образом,
"снаружи" `SafeInt` выглядит как обычный целочисленный
тип.

Это позволяет расширить другие типы новым функционалом,
не добавляя нагрузки по памяти или времени исполнения.
Компилятор делает всё для принятия правильного решения
при доступе к члену через `alias this`.

`alias this` так же работает и с классами.

## {SourceCode}

```d
import std.stdio : writeln;

struct Vector3 {
    private double[3] vec;
    alias vec this;

    double dot(Vector3 rhs) {
        return vec[0]*rhs.vec[0] +
          vec[1]*rhs.vec[1] + vec[2]*rhs.vec[2];
    }
}

void main()
{
    Vector3 vec;
    // в принципе, мы работаем с double[].
    vec = [ 0.0, 1.0, 0.0 ];
    assert(vec.length == 3);
    assert(vec[$ - 1] == 0.0);

    auto vec2 = Vector3([1.0,0.0,0.0]);
    // но функционал был расширен!
    writeln("vec dot vec2 = ", vec.dot(vec2));
}
```
