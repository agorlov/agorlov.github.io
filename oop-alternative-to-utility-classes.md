---
title: "ООП альтернатива классам-утилитам"
date: 2014-05-05
tags: ооп
description: |
  Классы-утилиты очень популряный паттерн в Java  и
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

Такой спсоб проектирования очень популярен в мире Java (также как и C#, Ruby, и др.)
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

Было много дискуcсий на эту тему, подборка:

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

Скажем, например, вы хотите прочитать текстовый файл, разделить его на строки,
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

Станет объктно-ориентированным:

```java
int max = new Max(10, 5).intValue();
```

Масло масленное? Не совсем, просто продолжай читать...

## Objects Instead of Data Structures

This is how I would design the same file-transforming functionality as above but
in an object-oriented manner:

{% highlight java %}
void transform(File in, File out) {
  Collection<String> src = new Trimmed(
    new FileLines(new UnicodeFile(in))
  );
  Collection<String> dest = new FileLines(
    new UnicodeFile(out)
  );
  dest.addAll(src);
}
{% endhighlight %}

`FileLines` implements `Collection<String>` and encapsulates  all file reading
and writing operations. An instance of `FileLines` behaves exactly as a
collection of strings and hides all I/O operations. When we iterate it&mdash;a
file is being read. When we `addAll()` to it&mdash;a file is being written.

`Trimmed` also implements `Collection<String>` and encapsulates a collection of
strings ([Decorator pattern](http://en.wikipedia.org/wiki/Decorator_pattern)).
Every time the next line is retrieved, it gets trimmed.

All classes taking
participation in the snippet are rather small: `Trimmed`, `FileLines`, and
`UnicodeFile`.
Each of them is responsible for its own single feature, thus following perfectly
the [single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle).

{% youtube D0dqC_3Bch8 %}

On our side, as users of the library, this may be not so important, but for
their developers it is an imperative.
It is much easier to develop, maintain and unit-test class `FileLines` rather
than using a `readLines()` method in a 80+ methods and 3000 lines utility class
`FileUtils`. Seriously, look at
[its source code](https://github.com/apache/commons-io/blob/commons-io-2.5/src/main/java/org/apache/commons/io/FileUtils.java).

An object-oriented approach enables lazy execution. The `in` file is not read
until its data is required. If we fail to open `out` due to some I/O error, the
first file won't even be touched. The whole show starts only after we call `addAll()`.

All lines in the second snippet, except the last one, instantiate and compose
smaller objects into bigger ones. This object composition is rather cheap for
the CPU since it doesn't cause any data transformations.

Besides that, it is obvious that the second script runs in O(1) space, while the
first one executes in O(n). This is the consequence of our procedural approach
to data in the first script.

In an object-oriented world, there is no data; there are only objects and their behavior!
