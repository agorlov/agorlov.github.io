Оригинал: http://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html

# Семь качеств хорошего объекта

Мартин Фаулер [сказал](http://martinfowler.com/bliki/InversionOfControl.html):

> Библиотека это набор функций, которые вы можете вызвать, сейчас не редко оформляется в виде класса.

Функции, организованные в классы? При всем уважении, но это не правильно. 
Это типичное заблуждение о классе в объектно-ориентированном программировании.
Классы это не органайзеры для функций. И объекты это не структуры данных.

Так каков же "правильный" объект? А какой не правильный? И в чем разница?
Не смотря на то что данная тема является предметом дискуссий, она очень важна.
Пока мы не понимаем, что такое объект, как мы можем разрабатывать объектно-ориентированные программы?
Хотя, благодаря Java, Ruby и другим, мы можем. Но насколько хороши они будут?
К сожалению, нет точной науки, и существует большое количество мнений. 
Я приведу мой список качеств хорошего объекта.



## Класс vs. Объект

badge

Перед тем, как начать разговор об объектах, давайте определим что такое класс. Это место, где рождаются объекты (создаются экземпляры).
Основное предназначение класса - создавать новые объекты и уничтожать их, когда они больше не используются. Класс знает какими должны
быть его дети и как они должны себя вести. Другими словами, он знает какие контракты они должны соблюдать.

YouTube video #WSgP85kr6eU
Why Getters-and-Setters Is An Anti-Pattern? (webinar #4); 1 July 2015.

Иногда я слышу, классы также известны как "шаблоны объектов" (например, в Википедии так говорится). Это определение неверно, 
потому что оно ставит классы в пассивное положение. Говоря технический, это может быть и так, но концептуально неправильно.
Только класс и его дети должны учавствовать и больше никто. Объект просит класс создать другой объект и класс создает его; и только.
На Ruby эта концепция выражена гораздо лучше чем в Java и C++:

```
photo = File.new('/tmp/photo.png')
```

Объект ``photo`` создается классом ``File`` (new это входная точка класса). Как только создали, объект начинает действовать самостоятельно. Ему не нужно знать, кем он создан и сколько в его классе братьев и сестер. Верно, я считаю что рефлексия это
ужасная идея, я напишу подробнее об этом в одном из следующих  постов :) Теперь, давайте поговорим об объектах и их лучших и худших сторонах.

## 1. Он существует в реальной жизни

В первую очередь, объект это живой организм.  Более того, объект должен быть антропоморфизирован, т. е. рассматриваться как человек (или домашнее животное, если они вам больше нравятся). Под этим я в основном подразумеваю, что объект это не структура данных или коллекция функций. Наоборот, это независимая сущность со своим жизненным циклом, собственным поведением и своими привычками.

Сотрудник, отдел, HTTP-запрос, таблица в MySQL, строка в файле или файл сам по себе правильные объекты, потому что они существуют
в реальной жизни, даже когда наша программа не работает. Чтобы быть точнее, объект это представитель реального существа. 
Это прокси этого реального существа (за ним могут стоять другие объекты). Без этого существа, очевидно, нет и объекта.

```
photo = File.new('/tmp/photo.png')
puts photo.width()
```

В этом примере, я прошу ``File`` создать новый объект ``photo``, который будет представлять реальный файл на диске. Вы можете 
сказать, что файл это что-то не настоящее и существует только когда компьютер включен. Я соглашусь и уточню определение 
"реальной жизни": это все, что существует за рамками программы в которой живет объект. Файл на диске находится за рамками нашей
программы; поэтому совершенно правильно создать его представителя в программе.

Контроллер, парсер, фильтр, валидатор, сервис локатор, синглтон или фабрика примеры плохих объектов (да, большинство GoF шаблонов являются анти-патернами!). Они не существуют в реальной жизни, за рамками вашей программы. Они придуманы только для того чтобы
связать вместе другие объекты. Они искуственные и поддельные существа. Они ни кого не представляют. Серьезно, кого представляет
XML-парсер? Никого.

Некоторые из них могут стать хорошими если их переименовать; другие никогда не смогут оправдать свое существование. Например, этот
XML parser, может быть переименован в "parseable XML" и начать представлять XML-документ, который существует за пределами нашей
области.

Всегда задавайте себе вопрос, "Какая реальная сущность стоит за моим объектом?" Если вы не можете ответить, начинайте думать о рефакторинге.


## 2. Он работает по договору

badge

Хороший объект всегда работает по договорам. Его нанимают не за его личные заслуги, а потому, что он подчиняется договору.
С другой стороны, нанимая объект, мы не должны дискриминировать и ожидать, что это будет определенный объект определенного
класса. Мы должны ожидать что любой объект будет выполнять то, что говорится в договоре.
Пока объект делает то, что нам нужно, нас не должен интересовать его класс происхождения, его пол или его религия.

Например, мне нежуно показать фото на экране. Я хочу, чтобы это фото читалось из файла в PNG формате. Я нанимаю
объект класса DataFile и прошу его дать мне бинарный контент этого изображения.

Но постойте, важно ли для меня откуда будет взят контент, будет ли это файл на диске или HTTP-запрос или может быть документ из Dropbox? Мне нет. Все что мне нужно, это чтобы какой-то объект дал мне массив байт PNG-файла. Мой договор будет выглядеть так:

```
interface Binary {
  byte[] read();
}
```

Теперь, любой объект любого класса (не только DataFile) может работать на меня. Все что он должен делать, это следовать договору --- реализуя интерфейс ``Binary``.

Простое правило: каждый публичный метод в правильном объекте реализует таковой из интерфейса. Если в вашем объекте есть публичные методы отстутствующие в интерфейсах, он плохо спроектирован.

Есть две практические причины для этого. Первая, объект работающий без договора невозможно мокнуть в юнит-тесте.
Вторая, без интерфейсый объект невозможно расширить декоратором.

## 3. Он уникален

Хороший объект должен всегда инкапсулировать что-то, чтобы быть уникальным. Если инкапсулировать нечего, объект может
иметь идентичных колонов, что плохо. Вот пример плохого объекта, у которого могут быть клоны:

```
class HTTPStatus implements Status {
  private URL page = new URL("http://www.google.com");
  @Override
  public int read() throws IOException {
    return HttpURLConnection.class.cast(
      this.page.openConnection()
    ).getResponseCode();
  }
}
```

Я могу создать несколько экземпляров класса HTTPStatus, и все они будут равны друг другу:

```
first = new HTTPStatus();
second = new HTTPStatus();
assert first.equals(second);
```

Очевидно, что на основе классов-утилит, в которых есть только статические методы, нелья создать хорошие объекты.
Вобщем, классы-утилиты не содержат хороших качеств упомянутых в этой статье и их даже нельзя назвать "классами"
Они просто ужасные нарушители объектной парадигмы и существуют в современных объектных языках только потому, что их создатели включили статические методы.

## 4. Он неизменяемый (Immutable)

Хороший объект никогда не меняет свое внутреннее состояние. Помните, объект это представитель реальной сущности, и 
эта сущность должна оставаться неизменной весь свой жизненный цикл объекта. Другими словами объект не должен изменять
тому, кого представляет. Он не должен менять владельцев, никогда. :)

Имейте в виду, что неизменяемость не означает, что все методы всегда возвращают одинаковые значения.

Вместо этого хороший неизменяемый объект очень динамичен. Однако, он никогда не меняет свое внутреннего состояния. Например:

```
@Immutable
final class HTTPStatus implements Status {
  private URL page;
  public HTTPStatus(URL url) {
    this.page = url;
  }
  @Override
  public int read() throws IOException {
    return HttpURLConnection.class.cast(
      this.page.openConnection()
    ).getResponseCode();
  }
}
```
Несмотря на то, что метод read() может возвращать различные значения, объект является неизменяемым. Он указывает
на определенную веб-страницу и никогда не будет указывать куда-то еще. Он не изменит свое внутреннее состояние, и
никогда не предаст URL который он представляет.

Почему неизменность это добро? Эта статья объясняет в деталях: [Объекты должны быть неизменными](http://www.yegor256.com/2014/06/09/objects-should-be-immutable.html).

В двух словах, неизменяемые объекты лучше, потому что:

- Неизменяемые объекты проще создавать, тестировать и использовать.
- Истинно-неизменяемые объекты всегда [потокобезопасны](http://www.yegor256.com/2017/01/17/synchronized-decorators.html) (thread-safe).
- Они помогают избегать [temporal coupling](http://www.yegor256.com/2015/12/08/temporal-coupling-between-method-calls.html)
- Их использование гарантирует вас от побочных эффектов.
- Они всегда атомарны в отказе (failure atomicity).
- Их очень просто закешировать.
- Они предотвращают NULL-ссылки.

И конечно, хороший объект не имеет сеттеров, которые могут изменить его состояние и спровоцировать его поменять URL.
Другими словами, добавлять метод setURL() в HTTPStatus будет большой ошибкой.

Помимо прочего, неизменяемые объекты будет повысят качество дизайна, добавят такие качества: стыкуемость (cohesive), единство (solid) и добавят легкость восприятия кода.


## 5. Он не содержит ничего статического

Статический метод реализует поведение класса, а не объекта. Предположим у нас есть класс ``File``, и метод ``size()`` у его потомков:

```
final class File implements Measurable {
  @Override
  public int size() {
    // calculate the size of the file and return
  }
}
```

	
Пока все хорошо; метод ``size()`` тут потому что есть контракт ``Measurable`` и каждый объект класса
``File`` может измерить свой размер. Ошибкой будет спроектировать данный класс со статическим методом
(такой дизайн известен как класс-утилита и очень популярен в Java, Ruby и почти всех ООП языках):


```
// УЖАСНЫЙ ДИЗАЙН, НЕ ДЕЛАЙТЕ ТАК!!!
class File {
  public static int size(String file) {
    // calculate the size of the file and return
  }
}
```
Такой дизайн идет в разрез с объектной парадигмой. Почему? Потому, что статические методы превращают объектно-ориентированное
программирование в "классо-ориентированное" программирование. Этот метод ``size()``, выставляет поведение класса,
а не его объекта. Ну и что в этом плохого, вы спросите? Почему не использовать и объекты и классы как основу нашего
кода? И почему и те и другие не могут иметь методы и свойства?

Проблема в том, что при классовом программировании декомпозиция перестает работать. Мы не можем разбить сложную задачу
на части, потому что во всей программеъект является локальной переменной в области видимости метода.
Двести двенадцать существует только один экземпляр класса. Сила ООП в том, что он позволяет
использовать объект как инструмент для декомпозиции. Когда я создаю экземпляр объекта внутри метода, он посвящен моей конкретной задаче. Он идеально изолирован от всех других объектов снаружи метода. Этот объект является локальной 
переменной внутри метода. Класс с его статическими методами всегда является глобальной переменной независимо от того, где я его использую. Из-за этого я не могу изолировать свое взаимодействие с этой переменной от других.

Помимо того, что публичные статические методы концептуально противоречат объектно-ориентированным принципам, они имеют ряд практических недостатков:

Превое, их невозможно подменить (to mock) (Ну, вы можете использовать PowerMock, но это будет самым ужасным решением, которое вы могли бы сделать в проекте Java... Я сделал это однажды, несколько лет назад).

Во-вторых, они не являются потокобезопасными по определению, поскольку всегда работают со статическими переменными, доступными из всех потоков. Их можно сделать потокобезопасными, но для этого всегда потребуется явная синхронизация.

Всякий раз, когда вы видите ``public static`` метод, сразу же переписывайте. 
Я даже не хочу упоминать, насколько ужасны статические (или глобальные) переменные. Я думаю это очевидно.

### 6. Его имя это не должность

badge

The name of an object should tell us what this object is, not what it does, just like we name objects in real life: book instead of page aggregator, cup instead of water holder, T-shirt instead of body dresser. There are exceptions, of course, like printer or computer, but they were invented just recently and by those who didn't read this article. :)

For example, these names tell us who their owners are: an apple, a file, a series of HTTP requests, a socket, an XML document, a list of users, a regular expression, an integer, a PostgreSQL table, or Jeffrey Lebowski. A properly named object is always possible to draw as a small picture. Even a regular expression can be drawn.

In the opposite, here is an example of names that tell us what their owners do: a file reader, a text parser, a URL validator, an XML printer, a service locator, a singleton, a script runner, or a Java programmer. Can you draw any of them? No, you can't. These names are not suitable for good objects. They are terrible names that lead to terrible design.

In general, avoid names that end with "-er"—most of them are bad.

"What is the alternative of a FileReader?" I hear you asking. What would be a better name? Let's see. We already have File, which is a representative of a real-world file on disk. This representative is not powerful enough for us, because he doesn't know how to read the content of the file. We want to create a more powerful one that will have that ability. What would we call him? Remember, the name should say what he is, not what he does. What is he? He is a file that has data; not just a file, like File, but a more sophisticated one, with data. So how about FileWithData or simply DataFile?

The same logic should be applicable to all other names. Always think about what it is rather than what it does. Give your objects real, meaningful names instead of job titles.

More about this in Don't Create Objects That End With -ER.

## 7. His Class Is Either Final or Abstract

badge

A good object comes from either a final or abstract class. A final class is one that can't be extended via inheritance. An abstract class is one that can't have instances. Simply put, a class should either say, "You can never break me; I'm a black box for you" or "I'm broken already; fix me first and then use."

There is nothing in between. A final class is a black box that you can't modify by any means. He works as he works, and you either use him or throw him away. You can't create another class that will inherit his properties. This is not allowed because of that final modifier. The only way to extend such a final class is through decoration of his children. Let's say I have the class HTTPStatus (see above), and I don't like him. Well, I like him, but he's not powerful enough for me. I want him to throw an exception if HTTP status is over 400. I want his method, read(), to do more that it does now. A traditional way would be to extend the class and overwrite his method:

```
class OnlyValidStatus extends HTTPStatus {
  public OnlyValidStatus(URL url) {
    super(url);
  }
  @Override
  public int read() throws IOException {
    int code = super.read();
    if (code >= 400) {
      throw new RuntimeException("Unsuccessful HTTP code");
    }
    return code;
  }
}
```

Why is this wrong? It is very wrong because we risk breaking the logic of the entire parent class by overriding one of his methods. Remember, once we override the method read() in the child class, all methods from the parent class start to use his new version. We're literally injecting a new "piece of implementation" right into the class. Philosophically speaking, this is an offense.

On the other hand, to extend a final class, you have to treat him like a black box and decorate him with your own implementation (a.k.a. Decorator Pattern):

```
final class OnlyValidStatus implements Status {
  private final Status origin;
  public OnlyValidStatus(Status status) {
    this.origin = status;
  }
  @Override
  public int read() throws IOException {
    int code = this.origin.read();
    if (code >= 400) {
      throw new RuntimeException("Unsuccessful HTTP code");
    }
    return code;
  }
}
```

Make sure that this class is implementing the same interface as the original one: Status. The instance of HTTPStatus will be passed into him through the constructor and encapsulated. Then every call will be intercepted and implemented in a different way, if necessary. In this design, we treat the original object as a black box and never touch his internal logic.

If you don't use that final keyword, anyone (including yourself) will be able to extend the class and... offend him :( So a class without final is a bad design.

An abstract class is the exact opposite case—he tells us that he is incomplete and we can't use him "as is." We have to inject our custom implementation logic into him, but only into the places he allows us to touch. These places are explicitly marked as abstract methods. For example, our HTTPStatus may look like this:

```
abstract class ValidatedHTTPStatus implements Status {
  @Override
  public final int read() throws IOException {
    int code = this.origin.read();
    if (!this.isValid()) {
      throw new RuntimeException("Unsuccessful HTTP code");
    }
    return code;
  }
  protected abstract boolean isValid();
}
```

As you see, the class doesn't know how exactly to validate the HTTP code, and he expects us to inject that logic through inheritance and through overriding the method isValid(). We're not going to offend him with this inheritance, since he defended all other methods with final (pay attention to the modifiers of his methods). Thus, the class is ready for our offense and is perfectly guarded against it.

To summarize, your class should either be final or abstract—nothing in between.

Update (April 2017): If you also agree that implementation inheritance is evil, all your classes must be final.
