# Интерфейсы

D позволяет определять интерфейсы (`interface`), которые технически похожи на
типы `class`, но их методы должны быть реализованы любым классом,
наследующим от интерфейса.

    interface Animal {
        void makeNoise();
    }

Метод `makeNoise` должен быть реализован классом `Dog`, потому что он наследуется от
интерфейса `Animal`. По сути, `makeNoise` работает подобно абстрактному методу основного класса.

    class Dog: Animal {
        override makeNoise() {
            ...
        }
    }

    auto dog = new Dog;
    Animal animal = dog; // неявное преобразование к интерфейсу
    animal.makeNoise();

Количество интерфейсов, которые класс может реализовать, не ограничено, но они
могут наследоваться только от *одного* основного класса.

### Шаблон NVI (невиртуальный интерфейс)

[Шаблон NVI](https://en.wikipedia.org/wiki/Non-virtual_interface_pattern)
предотвращает нарушение обычного выполнения шаблона путём разрешения
_невиртуальных_ методов для обычного интерфейса.
D позволяет использовать NVI шаблоны, разрешая определение в интерфейсах `final` функций, которые нельзя переопределять. Это обеспечивает специфические формы поведения, изменяемые переопределением других функций интерфейса.

    interface Animal {
        void makeNoise();
        final doubleNoise() // шаблон NVI
        {
            makeNoise();
            makeNoise();
        }
    }

### Подробнее

- [Интерфейсы в _Programming in D_](http://ddili.org/ders/d.en/interface.html)
- [Интерфейсы в D](https://dlang.org/spec/interface.html)

## {SourceCode}

```d
import std.stdio;

interface Animal {
    /*
    Виртуальная функция, которую
    нужно переопределить!
    */
    void makeNoise();

    /*
    Шаблон NVI. Внутренне использует
    makeNoise, чтобы изменять поведение
    наследующих классов.
    
    Параметры: 
        n =  количество повторений
    */
    final void multipleNoise(int n) {
        for(int i = 0; i < n; ++i) {
            makeNoise();
        }
    }
}

class Dog: Animal {
    override void makeNoise() {
        writeln("Bark!");
    }
}

class Cat: Animal {
    override void makeNoise() {
        writeln("Meeoauw!");
    }
}

void main() {
    Animal dog = new Dog;
    Animal cat = new Cat;
    Animal[] animals = [dog, cat];
    foreach(animal; animals) {
        animal.multipleNoise(5);
    }
}
```
