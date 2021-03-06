---
title: "Декларативное ООП, переводы"
date: 2018-06-24
tags: ооп
description: |
  Декларативный PHP или Объектный PHP - заметки
keywords:
  - декларативное и императивное программирование
  - объекты и классы
  - объектно ориентированное программирование на PHP
---

# Декларативный PHP (он же Объектный PHP)

Программист сейчас -- это переводчик языка заказчика на язык машины.
Декларативный подход, позволяет формулировать программу терминами предметнеой области, 
**описывая результат**, а не процесс его достижения.

Фактически, читая декларативный код за ним не сложно увидеть тех-задание,
если читать процедурный код, это будет сделать гораздо сложнее.

На текущий момент не существует языков "тех.заданий", когда машина будет
понимать, что нужно сделать прочитав ТЗ. Декларативный подход - это переходная
парадигма от машинных языков к естественным.

## Основные принципы

- Декларативность вместо имеративности &mdash; программа описывает результат, а не перечисялет инструкции, как его достичь
(инструкции понятны компьютерам, результат понятен людям);
- Проектирование от предметной области (результата), а не от данных; (сначала проектруют базу, потом классы,
получается data-driven design, вместо **domain driven**)
- Имена объектов отвечают на вопрос **Кто**, а не _Что он делает_ (объекты это представители сущностей предметной области);
- Объекты **компактны**: несколько методов, в конструкторах несколько параметров до 5-7;
- Объекты **неизменны** (immutable);
- Объекты расширяются декораторами (не наследованием) (why? and why?);
- Объекты и декораторы легко стыкуются благодаря **общим интерфейсам**;
- Легкие конструкторы: не содержат кода, только инициализация состояния;
- Не используется null (why?);
- Не используются статические методы, в том числе приватные (это аналог глобальных функций и процедур);
- нет сеттеров и геттеров;
- Не используется instanceof, type casting или рефлексия;
- Все публичные методы объявлены в интерфейсе;
- Все проверки в юнит-тестах только ``assertThat`` (why?)
- Группировка композиции объектов с помощью ``Объектов оболочек`` (Deocrating Envelopes/Wraps)
аналог: группировки при создании векторных изображений - несколько линий сгруппировали, получилась фигрура)
- Расширяющие/Уточняющие интерфейсы (Smart) - позволяют 

## Преимущества

- Поддерживаемость: все объекты это термины предметной области (не путать с сущностью в БД!);
- Поддерживаемость: все объекты компактны, их легко читать;
- Поддерживаемость: забудьте про костыли, их больше нет;
- Адаптивность: объектный код гораздо легче поддается изменениям; изменения это по сути реконфигурация готовых объектов;
- Адаптивность: теперь код это Лего - кубики это компактные объекты, сязующие интерфейсы - шипы.
- Програмирование делится на две фазы: программирование - создание новых объектов и конфигурирование - композиция объектов, х сочетание описывающее результат;
- Надежность: легкое создание юнит-тестов, т.к. [бъекты неизменяемые immutable](https://wiki.php.net/rfc/immutability);
- Надежность: очень компактные объекты и тесты (до 5 методов).

## Недостатки

- программа может иметь меньшую производительность;
- подход не привычен для большинства программистов.
  
## Пример набора объектов 

Набор объектов представляющих ``Пользователя`` в программе.
Все они реализуют интерфейс ``User``.

Интерфейс:

```php
/**
 * Пользователь.
 */
interface User
{
    /**
     * Информация о пользователе
     *
     * @return array
     */
    public function info(): array;
}
```

Базовая реализация:
```php
/**
 * Пользователь по user.id
 */
class UserStd implements User
{
    private $db;
    private $userId;

    public function __construct($userId, $db = null)
    {
        $this->userId = $userId;
        $this->db = null === $db ? getDB() : $db;
    }


    /**
     * Информация пользователя
     *
     * @return array $info
     */
    public function info(): array
    {
        $info = $this->db->query(
            "SELECT * FROM users WHERE id=" . $this->db->quote($this->userId)
        )->fetch();

        if ($info === false) {
            throw new \Exception("Не существует пользователя с id = " . $this->userId);
        }

        return $info;
    }
}
```

Группирующая оболочка:
```php
/**
 * Пользователь - обертка
 */
class UserWrap implements User
{
    private $orig;

    /**
     * Конструктор.
     *
     * @param User|callable
     */
    public function __construct(User $orig)
    {
        if (is_callable($orig)) {
            $this->orig = $orig;
        } else {
            $this->orig = function () use ($orig) {
                return $orig;
            };
        }
    }

    /**
     * Информация о пользователе
     *
     * @return array
     */
    public function info(): array
    {
        return $this->orig->call($this)->info();
    }
}
```

Пользователь по логину (в Java это было бы реализовано вторым конструктором):
```php
/**
 * Пользователь по Login-у
 */
class UserByLogin extends UserWrap
{
    /**
     * Конструктор.
     *
     * @param string $login логин пользователя
     * @param PDO $db база
     */
    public function __construct(string $login, $db = null)
    {
        parent::__construct(function () use ($login, $db) {
            $db = $db ?? getDB();

            $userId = $db->query(
                "SELECT id FROM users WHERE login=" . $db->quote($login)
            )->fetchColumn();

            if ($userId === false) {
                throw new \Exception("Не существует пользователя с login = " . $login);
            }

            return new UserStd($userId, $db);
        });
    }
}
```


## Патерны

### Декоратор 

Наращивает или видоимзеняет возможности базового объекта.

### Оболочка (Envelope)

Содединяет несколько объектов в группу.

### Отложенная загрузка (Lazy Loading)

Позволяет избавиться от исполняемого кода в конструкторах

### Smart-объекты

Дополняют функционал объекта.

### Overloading

В Java есть возможность объявлять множество конструкторов.
В PHP придется на каждый конструктор создавать новый объект.


## Что такое хорошо и что такое плохо?

С точки зрения описываемого подхода:

**Плохо**: null, static, extends, trait, singleton, new вне конструктора

**Хорошо**: interface, immutable, final, class


## Источники

- [Object Thinking by David West](https://read.amazon.com/kp/embed?asin=B00JDMPOKM&preview=newtab&linkCode=kpe&ref_=cm_sw_r_kb_dp_GG2lBbQ6SRBHK&tag=agdp-20)
- [Yegor Bugaenko Blog](https://www.yegor256.com/)
