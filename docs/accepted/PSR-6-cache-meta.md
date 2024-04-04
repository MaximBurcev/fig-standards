# Метадокумент PSR-кэша



## 1. Краткое содержание

Кэширование — это распространенный способ повысить производительность любого проекта, что делает библиотеки кэширования одной из наиболее распространенных функций многих фреймворков и библиотек. Это привело к ситуации, когда многие библиотеки используют свои собственные библиотеки кэширования с различными уровнями функциональности. Эти различия заставляют разработчиков изучать несколько систем, которые могут обеспечивать или не обеспечивать необходимую им функциональность. Кроме того, сами разработчики кэширующих библиотек сталкиваются с выбором между поддержкой ограниченного числа фреймворков или созданием большого количества классов адаптеров.

## 2. Зачем беспокоиться?

Общий интерфейс для систем кэширования решит эти проблемы. Разработчики библиотек и фреймворков могут рассчитывать на то, что системы кэширования будут работать так, как они ожидают, в то время как разработчикам систем кэширования потребуется реализовать только один набор интерфейсов, а не целый набор адаптеров.

Более того, представленная здесь реализация предназначена для будущего расширения. Он допускает множество внутренне отличных, но совместимых с API реализаций и предлагает четкий путь для будущего расширения более поздними PSR или конкретными разработчиками.

Плюсы:
* Стандартный интерфейс кэширования позволяет автономным библиотекам без труда поддерживать кэширование промежуточных данных; они могут просто (факультативно) зависеть от этого стандартного интерфейса и использовать его, не заботясь о деталях реализации.
* Обычно разрабатываемые библиотеки кэширования, совместно используемые несколькими проектами, даже если они расширяют этот интерфейс, вероятно, будут более надежными, чем дюжина отдельно разработанных реализаций.

Минусы:
* Любая стандартизация интерфейса сопряжена с риском задушить будущие инновации как «не то, как это делается (tm)». Тем не менее, мы считаем, что кэширование — это достаточно коммерциализированное проблемное пространство, поэтому предлагаемые здесь возможности расширения снижают любой потенциальный риск стагнации.

## 3. Рамки

### 3.1 Цели

* Общий интерфейс для базового и среднего уровня кэширования.
* Четкий механизм расширения спецификации для поддержки расширенных функций,
  как будущими PSR, так и отдельными реализациями. Этот механизм должен позволять
  для нескольких независимых расширений без коллизий.

### 3.2 Нецели

* Архитектурная совместимость со всеми существующими реализациями кэша.
* Расширенные функции кэширования, такие как пространство имен или теги, которые используются
  меньшинство пользователей.

## 4. Подходы

### 4.1 Выбранный подход

Эта спецификация принимает модель «репозиторий» или модель «сопоставителя данных» для кэширования.
вместо более традиционной модели «ключ-значение с истекающим сроком действия». Главная
причина в гибкости. Простую модель «ключ-значение» гораздо сложнее расширить.

Модель здесь предписывает использование объекта CacheItem, представляющего кеш.
запись и объект пула, который является заданным хранилищем кэшированных данных. Предметы
извлекаются из пула, взаимодействуют с ним и возвращаются в него. Пока еще немного
иногда многословный, он предлагает хороший, надежный и гибкий подход к кэшированию,
особенно в тех случаях, когда кеширование более сложно, чем просто сохранение и
получение строки.

Большинство имен методов были выбраны на основе общепринятой практики и имен методов в
обзор проектов участников и других популярных систем, не являющихся членами.

Плюсы:

* Гибкий и расширяемый
* Допускает множество вариантов реализации без нарушения интерфейса
* Неявно не предоставляет конструкторы объектов как псевдоинтерфейс.

Минусы:

* Немного более подробный, чем наивный подход

Примеры:

Некоторые общие шаблоны использования показаны ниже. Они не являются нормативными, но должны
продемонстрировать применение некоторых проектных решений.

~~~php
/**
 * Gets a list of available widgets.
 *
 * In this case, we assume the widget list changes so rarely that we want
 * the list cached forever until an explicit clear.
 */
function get_widget_list()
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    if (!$item->isHit()) {
        $value = compute_expensive_widget_list();
        $item->set($value);
        $pool->save($item);
    }
    return $item->get();
}
~~~

~~~php
/**
 * Caches a list of available widgets.
 *
 * In this case, we assume a list of widgets has been computed and we want
 * to cache it, regardless of what may already be cached.
 */
function save_widget_list($list)
{
    $pool = get_cache_pool('widgets');
    $item = $pool->getItem('widget_list');
    $item->set($list);
    $pool->save($item);
}
~~~

~~~php
/**
 * Clears the list of available widgets.
 *
 * In this case, we simply want to remove the widget list from the cache. We
 * don't care if it was set or not; the post condition is simply "no longer set".
 */
function clear_widget_list()
{
    $pool = get_cache_pool('widgets');
    $pool->deleteItems(['widget_list']);
}
~~~

~~~php
/**
 * Clears all widget information.
 *
 * In this case, we want to empty the entire widget pool. There may be other
 * pools in the application that will be unaffected.
 */
function clear_widget_cache()
{
    $pool = get_cache_pool('widgets');
    $pool->clear();
}
~~~

~~~php
/**
 * Load widgets.
 *
 * We want to get back a list of widgets, of which some are cached and some
 * are not. This of course assumes that loading from the cache is faster than
 * whatever the non-cached loading mechanism is.
 *
 * In this case, we assume widgets may change frequently so we only allow them
 * to be cached for an hour (3600 seconds). We also cache newly-loaded objects
 * back to the pool en masse.
 *
 * Note that a real implementation would probably also want a multi-load
 * operation for widgets, but that's irrelevant for this demonstration.
 */
function load_widgets(array $ids)
{
    $pool = get_cache_pool('widgets');
    $keys = array_map(function($id) { return 'widget.' . $id; }, $ids);
    $items = $pool->getItems($keys);

    $widgets = array();
    foreach ($items as $key => $item) {
        if ($item->isHit()) {
            $value = $item->get();
        } else {
            $value = expensive_widget_load($id);
            $item->set($value);
            $item->expiresAfter(3600);
            $pool->saveDeferred($item, true);
        }
        $widget[$value->id()] = $value;
    }
    $pool->commit(); // If no items were deferred this is a no-op.

    return $widgets;
}
~~~

~~~php
/**
 * This examples reflects functionality that is NOT included in this
 * specification, but is shown as an example of how such functionality MIGHT
 * be added by extending implementations.
 */

interface TaggablePoolInterface extends Psr\Cache\CachePoolInterface
{
    /**
     * Clears only those items from the pool that have the specified tag.
     */
    clearByTag($tag);
}

interface TaggableItemInterface extends Psr\Cache\CacheItemInterface
{
    public function setTags(array $tags);
}

/**
 * Caches a widget with tags.
 */
function set_widget(TaggablePoolInterface $pool, Widget $widget)
{
    $key = 'widget.' . $widget->id();
    $item = $pool->getItem($key);

    $item->setTags($widget->tags());
    $item->set($widget);
    $pool->save($item);
}
~~~

### 4.2 Alternative: "Weak item" approach

A variety of earlier drafts took a simpler "key value with expiration" approach,
also known as a "weak item" approach.  In this model, the "Cache Item" object
was really just a dumb array-with-methods object.  Users would instantiate it
directly, then pass it to a cache pool.  While more familiar, that approach
effectively prevented any meaningful extension of the Cache Item.  It effectively
made the Cache Item's constructor part of the implicit interface, and thus
severely curtailed extensibility or the ability to have the cache item be where
the intelligence lives.

In a poll conducted in June 2013, most participants showed a clear preference for
the more robust if less conventional "Strong item" / repository approach, which
was adopted as the way forward.

Pros:
* More traditional approach.

Cons:
* Less extensible or flexible.

### 4.3 Alternative: "Naked value" approach

Some of the earliest discussions of the Cache spec suggested skipping the Cache
Item concept all together and just reading/writing raw values to be cached.
While simpler, it was pointed out that made it impossible to tell the difference
between a cache miss and whatever raw value was selected to represent a cache
miss.  That is, if a cache lookup returned NULL it's impossible to tell if there
was no cached value or if NULL was the value that had been cached.  (NULL is a
legitimate value to cache in many cases.)

Most more robust caching implementations we reviewed -- in particular the Stash
caching library and the home-grown cache system used by Drupal -- use some sort
of structured object on `get` at least to avoid confusion between a miss and a
sentinel value.  Based on that prior experience FIG decided that a naked value
on `get` was impossible.

### 4.4 Alternative: ArrayAccess Pool

There was a suggestion to make a Pool implement ArrayAccess, which would allow
for cache get/set operations to use array syntax.  That was rejected due to
limited interest, limited flexibility of that approach (trivial get and set with
default control information is all that's possible), and because it's trivial
for a particular implementation to include as an add-on should it desire to
do so.

## 5. People

### 5.1 Editor

* Larry Garfield

### 5.2 Sponsors

* Paul Dragoonis, PPI Framework (Coordinator)
* Robert Hafner, Stash

## 6. Votes

[Acceptance vote on the mailing list](https://groups.google.com/forum/#!msg/php-fig/dSw5IhpKJ1g/O9wpqizWAwAJ)

## 7. Relevant Links

_**Note:** Order descending chronologically._

* [Survey of existing cache implementations][1], by @dragoonis
* [Strong vs. Weak informal poll][2], by @Crell
* [Implementation details informal poll][3], by @Crell

[1]: https://docs.google.com/spreadsheet/ccc?key=0Ak2JdGialLildEM2UjlOdnA4ekg3R1Bfeng5eGlZc1E#gid=0
[2]: https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdDdVd2llN1kxczZQejZaa3JHcXA3b0E#gid=0
[3]: https://docs.google.com/spreadsheet/ccc?key=0AsMrMKNHL1uGdEE3SU8zclNtdTNobWxpZnFyR0llSXc#gid=1

## 8. Errata

### 8.1 Handling of incorrect DateTime values in expiresAt()

The `CacheItemInterface::expiresAt()` method's `$expiration` parameter is untyped
in the interface, but in the docblock is specified as `\DateTimeInterface`.  The
intent is that either a `\DateTime` or `\DateTimeImmutable` object is allowed.
However, `\DateTimeInterface` and `\DateTimeImmutable` were added in PHP 5.5, and
the authors chose not to impose a hard syntactic requirement for PHP 5.5 on the
specification.

Despite that, implementers MUST accept only `\DateTimeInterface` or compatible types
(such as `\DateTime` and `\DateTimeImmutable`) as if the method was explicitly typed.
(Note that the variance rules for a typed parameter may vary between language versions.)

Simulating a failed type check unfortunately varies between PHP versions and thus is not
recommended.  Instead, implementors SHOULD throw an instance of `\Psr\Cache\InvalidArgumentException`.  
The following sample code is recommended in order to enforce the type check on the expiresAt()
method:

```php

class ExpiresAtInvalidParameterException implements Psr\Cache\InvalidArgumentException {}

// ...

if (! (
        null === $expiration
        || $expiration instanceof \DateTime
        || $expiration instanceof \DateTimeInterface
)) {
    throw new ExpiresAtInvalidParameterException(sprintf(
        'Argument 1 passed to %s::expiresAt() must be an instance of DateTime or DateTimeImmutable; %s given',
        get_class($this),
        is_object($expiration) ? get_class($expiration) : gettype($expiration)
    ));
}
```

### 8.2 Type additions

The 2.0 release of the `psr/cache` package includes scalar parameter types.  The 3.0 release of the package includes return types.  This structure leverages PHP 7.2 covariance support to allow for a gradual upgrade process, but requires PHP 8.0 for type compatibility.

The 2.0 version also corrects the Errata 8.1 above by providing a correct type hint for the `CacheItemInterface::expiresAt()` method's `$expiration` parameter.  That results in a slight change in the error thrown on invalid input; as it is still a fatal disallowed case, FIG has deemed it an acceptably small BC break in order to leverage correct native typing.

Implementers MAY add return types to their own packages at their discretion, provided that:

* the return types match those in the 3.0 package.
* the implementation specifies a minimum PHP version of 8.0.0 or later.

Implementers MAY add parameter types to their own packages in a new major release, either at the same time as adding return types or in a subsequent release, provided that:

* the parameter types match those in the 2.0 package.
* the implementation specifies a minimum PHP version of 8.0.0 or later.
* the implementation depends on `"psr/link": "^1.1 || ^2.0"` so as to exclude the untyped 1.0 version.

Implementers are encouraged but not required to transition their packages toward the 3.0 version of the package at their earliest convenience.
