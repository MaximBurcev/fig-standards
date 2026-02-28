Общий интерфейс для библиотек кэширования
==========================================


Этот документ описывает простой, но расширяемый интерфейс для элемента кэша и
драйвера кэша.

Ключевые слова «ОБЯЗАН» («MUST»), «НЕ ДОЛЖЕН» («MUST NOT»), «ТРЕБУЕТСЯ» («REQUIRED»), «ДОЛЖЕН» («SHALL»), «НЕ ДОЛЖЕН» («SHALL NOT»), «СЛЕДУЕТ» («SHOULD»),
«НЕ СЛЕДУЕТ» («SHOULD NOT»), «РЕКОМЕНДУЕТСЯ» («RECOMMENDED»), «МОЖЕТ» («MAY») и «ОПЦИОНАЛЬНО» («OPTIONAL») в данном документе
следует интерпретировать в соответствии с [RFC 2119][].

Финальные реализации МОГУТ расширять объекты дополнительной
функциональностью сверх предложенной, но ОБЯЗАНЫ в первую очередь реализовывать указанные
интерфейсы/функциональность.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

# 1. Спецификация

## 1.1 Введение

Кэширование — распространённый способ повышения производительности любого проекта,
что делает библиотеки кэширования одним из наиболее востребованных компонентов многих фреймворков и
библиотек. Совместимость на этом уровне означает, что библиотеки могут отказаться от
собственных реализаций кэширования и без труда опираться на ту, что предоставляется
фреймворком или другой специализированной библиотекой кэша.

PSR-6 уже решает эту задачу, однако делает это достаточно формально и многословно
применительно к самым простым случаям использования. Данный более простой подход направлен на построение
стандартизированного упрощённого интерфейса для типовых ситуаций. Он независим от
PSR-6, но спроектирован так, чтобы обеспечить максимально простую совместимость с PSR-6.

## 1.2 Определения

Определения понятий «Вызывающая библиотека», «Реализующая библиотека», TTL, «Срок истечения» и «Ключ»
скопированы из PSR-6, поскольку те же допущения остаются в силе.

*    **Вызывающая библиотека** (Calling Library) — библиотека или код, которым фактически нужны
услуги кэша. Эта библиотека будет использовать сервисы кэширования, реализующие интерфейсы
данного стандарта, но не будет знать ничего о конкретной реализации этих сервисов кэширования.

*    **Реализующая библиотека** (Implementing Library) — эта библиотека отвечает за реализацию
данного стандарта с целью предоставления сервисов кэширования любой Вызывающей библиотеке.
Реализующая библиотека ОБЯЗАНА предоставлять класс, реализующий интерфейс `Psr\SimpleCache\CacheInterface`.
Реализующие библиотеки ОБЯЗАНЫ поддерживать как минимум функциональность TTL, описанную
ниже, с гранулярностью до целых секунд.

*    **TTL** — Время жизни (Time To Live, TTL) элемента — это промежуток времени между
моментом сохранения элемента и моментом, когда он считается устаревшим. TTL обычно задаётся
целым числом, представляющим время в секундах, или объектом DateInterval.

* **Срок истечения** (Expiration) — фактический момент времени, когда элемент становится устаревшим. Он
  вычисляется прибавлением TTL к времени сохранения объекта.

  Элемент с TTL 300 секунд, сохранённый в 1:30:00, имеет срок истечения 1:35:00.

  Реализующие библиотеки МОГУТ инвалидировать элемент раньше запрошенного срока истечения,
  но ОБЯЗАНЫ считать элемент истёкшим, как только наступит его срок истечения. Если вызывающая
  библиотека запрашивает сохранение элемента без указания срока истечения или с указанием нулевого
  срока истечения или TTL, Реализующая библиотека МОЖЕТ использовать настроенную
  продолжительность по умолчанию. Если продолжительность по умолчанию не задана, Реализующая библиотека
  ОБЯЗАНА интерпретировать это как запрос на хранение элемента в кэше бессрочно или столько,
  сколько позволяет базовая реализация.

  Если передан отрицательный или нулевой TTL, элемент ОБЯЗАН быть удалён из кэша
  при его наличии, поскольку он уже является истёкшим.

*    **Ключ** (Key) — строка длиной не менее одного символа, однозначно идентифицирующая
элемент кэша. Реализующие библиотеки ОБЯЗАНЫ поддерживать ключи, состоящие из символов
`A-Z`, `a-z`, `0-9`, `_` и `.` в любом порядке в кодировке UTF-8, длиной до 64 символов.
Реализующие библиотеки МОГУТ поддерживать дополнительные символы, кодировки или бо́льшую длину,
но ОБЯЗАНЫ поддерживать как минимум указанный минимум. Библиотеки сами отвечают за экранирование
строк ключей в случае необходимости, однако ОБЯЗАНЫ возвращать оригинальную неизменённую строку ключа.
Следующие символы зарезервированы для будущих расширений и НЕ ДОЛЖНЫ поддерживаться
реализующими библиотеками: `{}()/\@:`

*    **Кэш** (Cache) — объект, реализующий интерфейс `Psr\SimpleCache\CacheInterface`.

*    **Промах кэша** (Cache Misses) — промах кэша возвращает null, поэтому обнаружить факт
хранения значения `null` невозможно. Это главное отличие от допущений PSR-6.

## 1.3 Кэш

Реализации МОГУТ предоставлять механизм задания пользователем TTL по умолчанию,
если он не указан для конкретного элемента кэша. Если пользователь не задал значение по умолчанию,
реализации ОБЯЗАНЫ использовать максимально допустимое значение, разрешённое базовой реализацией.
Если базовая реализация не поддерживает TTL, указанный пользователем TTL ОБЯЗАН молча игнорироваться.

## 1.4 Данные

Реализующие библиотеки ОБЯЗАНЫ поддерживать все сериализуемые типы данных PHP, в том числе:

*    **Строки** (Strings) — символьные строки произвольного размера в любой PHP-совместимой кодировке.
*    **Целые числа** (Integers) — целые числа любого размера, поддерживаемого PHP, вплоть до 64-битных знаковых.
*    **Числа с плавающей точкой** (Floats) — все знаковые значения с плавающей точкой.
*    **Булевы значения** (Booleans) — True и False.
*    **Null** — значение null (хотя при чтении его невозможно будет отличить от
промаха кэша).
*    **Массивы** (Arrays) — индексированные, ассоциативные и многомерные массивы произвольной глубины.
*    **Объекты** (Objects) — любой объект, поддерживающий сериализацию и десериализацию без потерь,
то есть такой, что `$o == unserialize(serialize($o))`. Объекты МОГУТ использовать
интерфейс `Serializable` PHP, магические методы `__sleep()` или `__wakeup()`
либо аналогичную языковую функциональность, если это уместно.

Все данные, переданные в Реализующую библиотеку, ОБЯЗАНЫ возвращаться в точно таком же виде,
включая тип переменной. То есть возвращать `(string) 5` при сохранённом значении `(int) 5` является ошибкой.
Реализующие библиотеки МОГУТ внутренне использовать функции PHP `serialize()`/`unserialize()`,
но не обязаны этого делать. Совместимость с ними используется лишь как базовый критерий приемлемых значений объектов.

Если по какой-либо причине невозможно вернуть точно сохранённое значение,
реализующие библиотеки ОБЯЗАНЫ вернуть промах кэша, а не повреждённые данные.

# 2. Интерфейсы

## 2.1 CacheInterface

Интерфейс кэша определяет наиболее базовые операции над набором записей кэша, включающие
базовое чтение, запись и удаление отдельных элементов кэша.

Кроме того, он содержит методы для работы с множеством записей кэша одновременно — запись, чтение и
удаление нескольких элементов за один вызов. Это удобно при большом количестве операций чтения/записи кэша
и позволяет выполнять их за один вызов к серверу кэша, существенно сокращая задержку.

Экземпляр CacheInterface соответствует единственному набору элементов кэша с единым пространством имён ключей
и является эквивалентом «Pool» в PSR-6. Разные экземпляры CacheInterface МОГУТ использовать одно и то же
хранилище данных, но ОБЯЗАНЫ быть логически независимы.

~~~php
<?php

namespace Psr\SimpleCache;

interface CacheInterface
{
    /**
     * Fetches a value from the cache.
     *
     * @param string $key     The unique key of this item in the cache.
     * @param mixed  $default Default value to return if the key does not exist.
     *
     * @return mixed The value of the item from the cache, or $default in case of cache miss.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function get($key, $default = null);

    /**
     * Persists data in the cache, uniquely referenced by a key with an optional expiration TTL time.
     *
     * @param string                 $key   The key of the item to store.
     * @param mixed                  $value The value of the item to store. Must be serializable.
     * @param null|int|\DateInterval $ttl   Optional. The TTL value of this item. If no value is sent and
     *                                      the driver supports TTL then the library may set a default value
     *                                      for it or let the driver take care of that.
     *
     * @return bool True on success and false on failure.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function set($key, $value, $ttl = null);

    /**
     * Delete an item from the cache by its unique key.
     *
     * @param string $key The unique cache key of the item to delete.
     *
     * @return bool True if the item was successfully removed. False if there was an error.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function delete($key);

    /**
     * Wipes clean the entire cache's keys.
     *
     * @return bool True on success and false on failure.
     */
    public function clear();

    /**
     * Obtains multiple cache items by their unique keys.
     *
     * @param iterable $keys    A list of keys that can obtained in a single operation.
     * @param mixed    $default Default value to return for keys that do not exist.
     *
     * @return iterable A list of key => value pairs. Cache keys that do not exist or are stale will have $default as value.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $keys is neither an array nor a Traversable,
     *   or if any of the $keys are not a legal value.
     */
    public function getMultiple($keys, $default = null);

    /**
     * Persists a set of key => value pairs in the cache, with an optional TTL.
     *
     * @param iterable               $values A list of key => value pairs for a multiple-set operation.
     * @param null|int|\DateInterval $ttl    Optional. The TTL value of this item. If no value is sent and
     *                                       the driver supports TTL then the library may set a default value
     *                                       for it or let the driver take care of that.
     *
     * @return bool True on success and false on failure.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $values is neither an array nor a Traversable,
     *   or if any of the $values are not a legal value.
     */
    public function setMultiple($values, $ttl = null);

    /**
     * Deletes multiple cache items in a single operation.
     *
     * @param iterable $keys A list of string-based keys to be deleted.
     *
     * @return bool True if the items were successfully removed. False if there was an error.
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if $keys is neither an array nor a Traversable,
     *   or if any of the $keys are not a legal value.
     */
    public function deleteMultiple($keys);

    /**
     * Determines whether an item is present in the cache.
     *
     * NOTE: It is recommended that has() is only to be used for cache warming type purposes
     * and not to be used within your live applications operations for get/set, as this method
     * is subject to a race condition where your has() will return true and immediately after,
     * another script can remove it, making the state of your app out of date.
     *
     * @param string $key The cache item key.
     *
     * @return bool
     *
     * @throws \Psr\SimpleCache\InvalidArgumentException
     *   MUST be thrown if the $key string is not a legal value.
     */
    public function has($key);
}
~~~

## 2.2 CacheException

~~~php

<?php

namespace Psr\SimpleCache;

/**
 * Interface used for all types of exceptions thrown by the implementing library.
 */
interface CacheException
{
}
~~~

## 2.3 InvalidArgumentException

~~~php
<?php

namespace Psr\SimpleCache;

/**
 * Exception interface for invalid cache arguments.
 *
 * When an invalid argument is passed, it must throw an exception which implements
 * this interface.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~
