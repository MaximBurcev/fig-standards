Взаимные объятия
====================


Ключевые слова «ОБЯЗАН», «НЕ ДОЛЖЕН», «REQUIRED», «SHALL», «SHALL NOT», «СЛЕДУЕТ»,
«SHOULD NOT», «RECOMMENDED», «МОЖЕТ» и «OPTIONAL» в данном документе следует
интерпретировать так, как описано в [RFC 2119](http://tools.ietf.org/html/rfc2119).

## 1. Обзор

Данный стандарт устанавливает общий способ для объектов выражать взаимную признательность и поддержку посредством объятий. Это позволяет объектам поддерживать друг друга конструктивным образом, способствуя сотрудничеству между различными PHP-проектами.

## 2. Спецификация

Данная спецификация определяет два интерфейса: \Psr\Hug\Huggable и \Psr\Hug\GroupHuggable.

### Huggable-объекты (обнимаемые объекты)

1. Huggable-объект выражает привязанность и поддержку другому объекту, вызывая его метод hug() и передавая $this в качестве первого параметра.

2. Объект, у которого вызван метод hug(), ОБЯЗАН обнять вызывающий объект в ответ как минимум один раз.

3. Два объекта, находящихся в процессе объятий, МОГУТ продолжать обнимать друг друга любое количество итераций. Однако каждый huggable-объект ОБЯЗАН иметь условие завершения, которое предотвратит бесконечный цикл. Например, объект МОЖЕТ быть настроен на допущение не более 3 взаимных объятий, после чего он разрывает цепочку объятий и возвращает управление.

4. Объект МОЖЕТ предпринимать дополнительные действия, в том числе изменять состояние, при получении объятий. Распространённым примером является увеличение внутреннего счётчика счастья или удовлетворённости.

### GroupHuggable-объекты (объекты групповых объятий)

1. Объект МОЖЕТ дополнительно реализовывать GroupHuggable, чтобы указать, что он способен поддерживать и одобрять несколько объектов одновременно.

## 3. Интерфейсы

### HuggableInterface

~~~php
namespace Psr\Hug;

/**
 * Defines a huggable object.
 *
 * A huggable object expresses mutual affection with another huggable object.
 */
interface Huggable
{

    /**
     * Hugs this object.
     *
     * All hugs are mutual. An object that is hugged MUST in turn hug the other
     * object back by calling hug() on the first parameter. All objects MUST
     * implement a mechanism to prevent an infinite loop of hugging.
     *
     * @param Huggable $h
     *   The object that is hugging this object.
     */
    public function hug(Huggable $h);
}
~~~

~~~php
namespace Psr\Hug;

/**
 * Defines a huggable object.
 *
 * A huggable object expresses mutual affection with another huggable object.
 */
interface GroupHuggable extends Huggable
{

  /**
   * Hugs a series of huggable objects.
   *
   * When called, this object MUST invoke the hug() method of every object
   * provided. The order of the collection is not significant, and this object
   * MAY hug each of the objects in any order provided that all are hugged.
   *
   * @param Huggable[] $huggables
   *   An array or iterator of objects implementing the Huggable interface.
   */
  public function groupHug($huggables);
}
~~~
