# Установка на Heroku

### Требования

- Heroku [account](https://signup.heroku.com/login)
- Установленный [Git](https://git-scm.com/)
- Ваше приложение должно компилироваться без ошибок при
  использовании dmd (`dub build`).

### 1. Настройка приложения

Heroku необходимо знать, как связаться с приложением.
Для этого предоставляется глобальная переменная окружения
`PORT`, которую необходимо использовать в приложении,
чтобы открыть соответствующий порт.
Во время разработки будет использоваться порт по умолчанию
(в данном случае __8080__):

```d
shared static this() {
  // ...
  auto settings = new HTTPServerSettings;
  // Указывается порт по умолчанию на случай,
  // если переменная $PORT не установлена
  settings.port = environment.get("PORT", "8080").to!ushort;
  listenHTTP(settings, router);
}
```

Также необходимо создать `Procfile` - текстовый
файл в корне приложения, в котором явно указывается,
какая команда используется для запуска приложения.

`Procfile` в примере выглядит так:

```
web: ./hello-world
```

### 2. Подготовка приложения

Перед тем, как продолжить, нужно получить доступ
к командной строке Heroku, используя
[Heroku Toolbelt](https://toolbelt.heroku.com/standalone).

Этот пакет предоставляет доступ к интерфейсу командной
строки (CLI) Heroky, с помощью которого можно
управлять приложением и дополнениями, а также
заниматься их масштабированием.

После установки Toolbelt необходимо выполнить
следующую команду:

```
$ heroku login
```

### 3. Создание приложения

Новое приложение создаётся в [heroku dashboard](https://dashboard.heroku.com).
Название приложения рекомендуется запомнить - оно
пригодится в дальнейшем.

Альтернативно, можно использовать командную строку:

```
$ heroku create
Creating app... done, ⬢ rocky-hamlet-67506
https://rocky-hamlet-67506.herokuapp.com/ | https://git.heroku.com/rocky-hamlet-67506.git
```

В данном случае название приложения - *rocky-hamlet-67506*.

### Установка с использованием git

Для установки приложения используются команды `git`.
Необходимо создать новый `remote`, куда
будут передаваться новые релизы.
Для этого необходимо передать название приложения
(см. предыдущий раздел) в качестве аргумента.
В нашем случае - это *rocky-hamlet-67506*.

```
$ heroku git:remote -a rocky-hamlet-67506
```

Обратите внимание, `remote` будет добавлен в
конфигурацию git:

```
$ git remote -v
heroku	https://git.heroku.com/rocky-hamlet-67506.git (fetch)
heroku	https://git.heroku.com/rocky-hamlet-67506.git (push)
```

### Добавление билд-паков

С помощью билд-паков генерируются ассеты или
компилируемый код.

Дополнительную информацию о них можно найти в
[документации Heroku](https://devcenter.heroku.com/articles/buildpacks)

Для vibe.d можно использовать [билд-пак vibe.d](https://github.com/MartinNowak/heroku-buildpack-d):

```
$ heroku buildpacks:set https://github.com/MartinNowak/heroku-buildpack-d
```
По умолчанию билд-пак использует новейшую версию
компилятора `dmd`. Использование GDC или LDC
так же возможно, для этого нужно добавить в проект
файл `.d-compiler` и в нём указать необходимый
компилятор и его версию.

При указании `dmd`, `ldc` или `gdc` будет использоваться
новейшая версия. Конкретную версию можно указать так:
`dmd-2.0xxx`, `ldc-1.0xxx` или `gdc-4.9xxx`.

### Размещение кода

Напишите какой-нибудь классный код, используя
любые удобные средства git.

Для установки новой версии нужно просто передать
её в соответствующий `remote` Heroku.

```
$ git add .
$ git commit -am "My first vibe.d release"
$ git push heroku master
Counting objects: 9, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 997 bytes, done.
Total 9 (delta 0), reused 0 (delta 0)

-----> Fetching custom git buildpack... done
-----> D (dub package manager) app detected
-----> Building libevent
-----> Building libev
-----> Downloading DMD
-----> Downloading dub package manager
-----> Setting PATH:
-----> Initializing toolchain
-----> Building app
       Running dub build ...
Building configuration "application", build type release
Running dmd (compile)...
Compiling diet template 'index.dt' (compat)...
Linking...
       Build was successful
-----> Discovering process types
       Procfile declares types -> web
-----> Compiled slug size: 3.5MB
-----> Launching... done, v4
       https://rocky-hamlet-67506.herokuapp.com/ deployed to Heroku
To git@heroku.com:rocky-hamlet-67506.git
 * [new branch]      master -> master
```

Браузер с приложением можно открыть командой

```
$ heroku open
```

### Масштабирование контейнеров dyno

Установленное приложение исполняется в контейнере dyno.
Это лекговесный контейнер, исполняющий команду,
указанную в Procfile.

Использование команды `ps` позволяет узнать,
сколько dyno запущено в данный момент:

```
$ heroku ps
Free dyno hours quota remaining this month: 550h 0m (100%)
For more information on dyno sleeping and how to upgrade, see:
https://devcenter.heroku.com/articles/dyno-sleeping

No dynos on ⬢ rocky-hamlet-67506
```

По умолчанию приложение размещается в свободном
dyno, который не принимает запросы. Свободные
dyno засыпают по истечении получаса бездействия.
Это приводит к задержке до нескольких секунд
при поступлении первого запроса.

Для запуска dyno используется команда:

```
$ heroku ps:scale web=1
```

### Проверка логов

В Heroku логи - это потоки упорядоченных по
времени событий, которые собираются из выходных
потоков всех ваших приложений и компонентов
Heroku. Таким образом организуется единый канал
для всех событий.

```
$ heroku logs --tail
```

## Дополнительная информация

После установки приложения в Heroku, его функционал
можно расширить при помощи дополнений. Например:

- [Postgresql](https://elements.heroku.com/addons/heroku-postgresql)
- [MongoDb](https://elements.heroku.com/addons/mongohq)
- [Logging](https://elements.heroku.com/addons#logging)
- [Caching](https://elements.heroku.com/addons#caching)
- [Error and exceptions](https://elements.heroku.com/addons#errors-exceptions)
