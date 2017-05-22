# Шаблоны и мета-программирование

Если Вы когда-либо сталкивались с *мета-программированием
на основе шаблонов* в C++, Вам будет приятно узнать,
какие инструменты D предлагает, чтобы упростить Вашу жизнь.
Мета-программирование на основе шаблонов - техника,
позволяющая принимать решения на основе свойств типов,
и таким образом делающая обобщённые типы ещё более
гибкими, в зависимости от типов, с которыми шаблоны
инстанциируются.

### `static if` и `is`

Подобно обычному `if`, `static if` компилирует
блок кода в зависимости от условия, вычисляемого
во время компиляции:

    static if(is(T == int))
        writeln("T - это int");
    static if (is(typeof(x) :  int))
        writeln("Переменная x может быть неявно преобразована в int");

Выражение [`is`](http://wiki.dlang.org/Is_expression) -
помощник, вычисляющий выражения во время компиляции.

    static if(is(T == int)) { // T - параметр шаблона
        int x = 10;
    }

Фигурные скобки удаляются, если условие истинно - новая область
видимости не создаётся. `{ {` и `} }` явно создают новый блок.

`static if` можно использовать в разных участках кода - в функциях,
в глобальной области видимости, или в теле определения типов.

### `mixin template`

Всякий раз, как Вы встречаете *избыточный код*, на помощь
придёт `mixin template`:

    mixin template Foo(T) {
        T foo;
    }
    ...
    mixin Foo!int; // с этого моменда доступна переменная foo типа int.

`mixin template` может содержать произвольное количество
комплексных выражений, которые будут вставлены в месте
инстанцирования. Помашите ручкой препроцессору, если Вы пришли из C!

### Ограничения шаблонов

Шаблону можно указать произвольный набор ограничений,
помогающих указать необходимые свойства типов:

    void foo(T)(T value)
      if (is(T : int)) { // foo!T принимается только если T
                         // можно преобразовать в int
    }

Ограничения можно комбинировать в логическое выражение,
они могут содержать вызовы функций, которые можно вычислить
во время компиляции. Например, `std.range.primitives.isRandomAccessRange`
проверяет, является ли тип диапазоном, поддерживающим оператор `[]`.

### В деталях

### Основы

- [Tutorial to D Templates](https://github.com/PhilippeSigaud/D-templates-tutorial)
- [Conditional compilation](http://ddili.org/ders/d.en/cond_comp.html)
- [std.traits](https://dlang.org/phobos/std_traits.html)
- [More templates  _Programming in D_](http://ddili.org/ders/d.en/templates_more.html)
- [Mixins in  _Programming in D_](http://ddili.org/ders/d.en/mixin.html)

### Углублённо

- [Conditional compilation](https://dlang.org/spec/version.html)
- [Traits](https://dlang.org/spec/traits.html)
- [Mixin templates](https://dlang.org/spec/template-mixin.html)
- [D Templates spec](https://dlang.org/spec/template.html)

## {SourceCode}

```d
import std.traits : isFloatingPoint;
import std.uni : toUpper;
import std.string : format;
import std.stdio : writeln;

/*
Vector, "просто работающий" с
числами - целыми и с плавающей точкой.
*/
struct Vector3(T)
  if (is(T: real))
{
private:
    T x,y,z;

    /*
    Генератор для свойств, ведь мы ненавидим
    писать один и тот же код!
    
    var -> T getVAR() и void setVAR(T)
    */
    mixin template GetterSetter(string var) {
        // Используем mixin, чтобы сконструировать
        // имена функций.
        mixin("T get%s() const { return %s; }"
          .format(var.toUpper, var));

        mixin("void set%s(T v) { %s = v; }"
          .format(var.toUpper, var));
    }

    /*
    Запросто сгенерировать getX, setX, и т.п.
    функции, используя mixin template.
    */
    mixin GetterSetter!"x";
    mixin GetterSetter!"y";
    mixin GetterSetter!"z";

public:
    /*
    Функция dot доступна только для типов
    с плавающей точкой.
    */
    static if (isFloatingPoint!T) {
        T dot(Vector3!T rhs) {
            return x * rhs.x + y * rhs.y +
                z * rhs.z;
        }
    }
}

void main()
{
    auto vec = Vector3!double(3,3,3);
    // Не сработает из-за ограничения шаблона!
    // Vector3!string illegal;

    auto vec2 = Vector3!double(4,4,4);
    writeln("vec dot vec2 = ", vec.dot(vec2));

    auto vecInt = Vector3!int(1,2,3);
    // не имеет функции dot, поскольку мы
    // статически ограничили её только для вещественных
    // чисел
    // vecInt.dot(Vector3!int(0,0,0));

    // сгенерированные свойства!
    vecInt.setX(3);
    vecInt.setZ(1);
    writeln(vecInt.getX, ",",
      vecInt.getY, ",", vecInt.getZ);
}
```
