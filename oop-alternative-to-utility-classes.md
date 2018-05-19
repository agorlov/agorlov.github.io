---
title: "ООП альтернатива классам-утилитам"
date: 2014-05-05
tags: ооп
description: |
  Классы-утилиты очень популярный паттерн в Java  и
  в других объектно-ориентированных языках. Однако многие
  считают их ужасной практикой, которую следует избегать.
keywords:
  - статические методы это зло
  - static это зло
  - классы-утилиты это зло
  - классы-утилиты плохие
  - utility classes зло
  - utility classes
  - классы утилиты это плохая практика
translated:
  - Japanese: http://tbd.kaitoy.xyz/2016/01/03/oop-alternative-to-utility-classes/
book: elegant-objects-1 3.2
---

Класс-утилита (или классы-хелперы helper-class) это "структура" которая содержит
только статические методы и не инкапсулирет состояния. `StringUtils`, `IoUtils`,
`FileUtils` из [Apache Commons](http://commons.apache.org/); `Iterables` и `Iterators` из
[Guava](https://code.google.com/p/guava-libraries/), и [`Files`](http://docs.oracle.com/javase/7/docs/api/java/nio/file/Files.html)
из JDK7 отличные примеры классов-утилит.

<!--more-->

Такой способ проектирования очень популярен в мире Java (также как и C#, Ruby, и др.)
потому что классы-утилиты предоставляют общую функционально используемую повсюду.

{% youtube psrp3TtaYYI %}

Мы хотим следовать [принципу DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
и избегать дублирования. Поэтому, мы помещаем общие блоки кода в классы-утилиты и 
повторно их используем, когда нужно:

```java
// This is a terrible design, don't reuse
public class NumberUtils {
  public static int max(int a, int b) {
    return a > b ? a : b;
  }
}
```

Действительно, это очень удобная техника!?

## Классы-утилиты &mdash; зло

Однако, в объектно-ориентированном мире, классы-утилиты считаются очень плохой 
(некоторые даже могут сказать "ужасной") практикой.

Было много дискуссий на эту тему, подборка:

[Are Helper Classes Evil?](http://blogs.msdn.com/b/nickmalik/archive/2005/09/06/461404.aspx) Nick Malik,
[Why helper, singletons and utility classes are mostly bad](http://smart421.wordpress.com/2011/08/31/why-helper-singletons-and-utility-classes-are-mostly-bad-2/)  Simon Hart,
[Avoiding Utility Classes](https://github.com/marshallward/marshallward.org/blob/master/content/avoid_util_classes.rst) Marshal Ward,
[Kill That Util Class!](http://www.jroller.com/DhavalDalal/entry/kill_that_util_class) Dhaval Dalal,
[Helper Classes Are A Code Smell](http://www.robbagby.com/posts/helper-classes-are-a-code-smell/) Rob Bagby.


Кроме того, есть несколько вопросов на StackExchange о классах утилитах:
[If a “Utilities” class is evil, where do I put my generic code?](http://stackoverflow.com/questions/3339929/if-a-utilities-class-is-evil-where-do-i-put-my-generic-code),
[Utility Classes are Evil](http://stackoverflow.com/questions/3340032/utility-classes-are-evil).

В сухом остатке всех приводимых аргументов это то,  что классы-утилиты не 
являются правильными **объектами**; поэтому они не вписываются в
объектно-ориентированный мир. Они были позаимствованы из **процедурного**
программирования, в основном потому, что мы привыкли к функциональной 
парадигме декомпозиции.

Предполагаю, что вы согласны с аргументами и хотите перестать использовать
классы-утилиты, я покажу на примерах как можно заменить эти создания
правильными [объектами](/yb-object.html).
Assuming you agree with the arguments and want to stop using utility classes,
I'll show by example how these creatures can be replaced with proper
[objects](/yb-object.html).

## Процедурный пример

Скажем, вы хотите прочитать текстовый файл, разделить его на строки,
к каждой строке применить trim и потом сохранить результат в другом файле.
Это можно сделать с помощью [`FileUtils`](http://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/FileUtils.html)
из Apache Commons:

```java
void transform(File in, File out) {
  Collection<String> src = FileUtils.readLines(in, "UTF-8");
  Collection<String> dest = new ArrayList<>(src.size());
  for (String line : src) {
    dest.add(line.trim());
  }
  FileUtils.writeLines(out, dest, "UTF-8");
}
```
Код выше выглядит вполне нормально; однако, это процедурное программирование,
не объектно-ориентированное. Мы оперируем данными (байтами и битами) и
явно инструктируем компьютер откуда их брать и потом куда класть, 
в каждой отдельной строке кода. Мы определили *процедурное исполнение*.

## Объектная альтернатива

В объектно-ориентированной парадигме мы должны создавать экземпляры и комбинировать
объекты (композиция), таким образом позволяя им управлять данными когда и как **они**
пожелают. Вместо вызова вспомогательных статических функций, мы должны создавать
объекты, которые способы реализовывать нужное поведение:


```java
public class Max implements Number {
  private final int a;
  private final int b;
  public Max(int x, int y) {
    this.a = x;
    this.b = y;
  }
  @Override
  public int intValue() {
    return this.a > this.b ? this.a : this.b;
  }
}
```

Этот процедурный вызов:

```java
int max = NumberUtils.max(10, 5);
```

Станет объектно-ориентированным:

```java
int max = new Max(10, 5).intValue();
```

Масло масленое? Не совсем, просто продолжай читать...

## Объекты вместо структур данных

Вот так я бы спроектировал аналогичный приведенному выше функционал,
преобразовывающий файл, но в объектно ориентированной форме:

```java
void transform(File in, File out) {
  Collection<String> src = new Trimmed(
    new FileLines(new UnicodeFile(in))
  );
  Collection<String> dest = new FileLines(
    new UnicodeFile(out)
  );
  dest.addAll(src);
}
```

`FileLines` реализует `Collection<String>` и заключает в себе операции чтения 
и записи. Экземпляр `FileLines` выглядит как коллекция строк и скрывает все
операции ввода вывода. Когда мы итерируем его, файл считывается. Когда мы
выполняем `addAll()` , файл записывается.

`Trimmed` также реализует `Collection<String>` и инкапсулирует коллекцию строк ([Decorator pattern](http://en.wikipedia.org/wiki/Decorator_pattern)). Каждый раз, считыая следующую строку она обрезается.

Все учавствующие классы в примере довольно маленькие: Trimmed`, `FileLines`  и
`UnicodeFile`.

Каждый из них отвечает за одну собственную фичу, полностью соответствуя [принципу единственной
ответственности](http://en.wikipedia.org/wiki/Single_responsibility_principle).

{% youtube D0dqC_3Bch8 %}

С нашей стороны, как пользователям библиотеки, это может быть не так важно, но
для разработчиков это **императивно**. На много проще разрабатывать, поддерживать
и писать юнит-тест для класса `FileLines` чем пользоваться методом `readLines()` 
из класса утилиты `FileUtils` содержащей более 80 методов и 3000 строк кода.
Серьезно, взгляните [на исходник](https://github.com/apache/commons-io/blob/commons-io-2.5/src/main/java/org/apache/commons/io/FileUtils.java).

Объектный подход позволяет использовать отложенное выполнение (lazy execution).
`in` файл не читается, до тех пор пока данные не потребуются. Если нам не удалось
открыть `out` из за ошибки ввода/вывода, первый файл даже не будет тронут.
Все начнется только после вызова `addAll()`.

Все строки во втором фрагменте, за исключением последней строки, создают
композицию из небольших объектов в большие. Такая композиция на много 
дешевле для процессора, т.к. она не вызывает каких-либо преобразований
данных.

Помимо этого, очевидно, что второй скрипт выполняется в O(1), тогда как
первый выполняется в O(n). Это следствие нашего процедурного подхода к данным
в первом скрипте.

В объектно-ориентированном мире, нет данных; там только объекты и их поведение!
