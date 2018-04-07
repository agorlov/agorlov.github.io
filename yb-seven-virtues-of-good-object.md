Оригинал: http://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html

# Семь качеств хорошего объекта

Мартин Фаулер [сказал](http://martinfowler.com/bliki/InversionOfControl.html):

> Библиотека это набор функций, которые вы можете вызывать, сейчас обычно организуются в виде классов.

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

In this example, I'm asking File to construct a new object photo, which will be a representative of a real file on disk. You may say that a file is also something virtual and exists only when the computer is turned on. I would agree and refine the definition of "real life" as follows: It is everything that exists aside from the scope of the program the object lives in. The disk file is outside the scope of our program; that's why it is perfectly correct to create its representative inside the program.

A controller, a parser, a filter, a validator, a service locator, a singleton, or a factory are not good objects (yes, most GoF patterns are anti-patterns!). They don't exist apart from your software, in real life. They are invented just to tie other objects together. They are artificial and fake creatures. They don't represent anyone. Seriously, an XML parser—who does it represent? Nobody.

Some of them may become good if they change their names; others can never excuse their existence. For example, that XML parser can be renamed to "parseable XML" and start to represent an XML document that exists outside of our scope.

Always ask yourself, "What is the real-life entity behind my object?" If you can't find an answer, start thinking about refactoring.
