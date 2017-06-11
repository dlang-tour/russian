# Локальная установка D

Последнюю версию референс-компилятора **DMD** (Digital Mars D) можно
[загрузить](http://dlang.org/download.html) и установить с официального сайта
[dlang.org](https://dlang.org)

### Windows

* [Установщик](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd-{{latest-release}}.exe)
* или: [Архив](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.windows.7z)
* или с помощью [chocolatey](https://chocolatey.org/packages/dmd): `choco install dmd`

### macOS

* `.dmg` [Пакет](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.dmg)
* или: [Архив](http://downloads.dlang.org/releases/2.x/{{latest-release}}/dmd.{{latest-release}}.osx.tar.xz)
* или с помощью [Homebrew](http://brew.sh): `brew install dmd`

### Linux / FreeBSD / macOS

Для быстрой установки dmd в свою пользовательскую директорию выполните:
`curl -fsS https://dlang.org/install.sh | bash -s dmd`

Также доступны пакеты для различных дистрибутивов:

* [ArchLinux](https://wiki.archlinux.org/index.php/D_(programming_language))
* [Debian/Ubuntu](http://d-apt.sourceforge.net).
* [Fedora/CentOS](http://dlang.org/download.html#dmd)
* [Gentoo](https://wiki.gentoo.org/wiki/Dlang)
* [OpenSuse](http://dlang.org/download.html#dmd)

### Другие компиляторы

Помимо референс-компилятора DMD, который использует свой собственный backend,
существует ещё два компилятора, которые можно найти на странице загрузок
[dlang.org](https://dlang.org):

* [**LDC**](https://github.com/ldc-developers/ldc#installation), основанный на LLVM backend
* [**GDC**](http://gdcproject.org/downloads) использует GCC backend

LDC и GDC не всегда соответствуют самой последней frontend версии DMD, 
но предоставляют более высокие уровни оптимизации и возможность компиляции на
другие платформы вроде ARM.

Подробности о компиляторах можно узнать из [вики](https://wiki.dlang.org/Compilers).
