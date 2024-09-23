# Общий интерфейс кэширования

Кэширование — это распространенный способ повысить производительность любого проекта,
что делает библиотеки кэширования одной из наиболее распространенных функций многих фреймворков и
библиотек. Это привело к ситуации, когда многие библиотеки используют свои собственные
библиотеки кэширования с различными уровнями функциональности.
Эти различия заставляют разработчиков изучать несколько систем,
которые могут обеспечивать или не обеспечивать необходимую им функциональность.
Кроме того, сами разработчики кэширующих библиотек сталкиваются с выбором между поддержкой
ограниченного числа фреймворков или созданием большого количества классов адаптеров.

Общий интерфейс для систем кэширования решит эти проблемы. Разработчики библиотек и фреймворков могут рассчитывать на
то, что системы кэширования работают так, как они
ожидают и разработчикам останется только реализовать
единый набор интерфейсов, а не целый набор адаптеров.

Слова «НЕОБХОДИМО» / «ДОЛЖНО» ("MUST"), «НЕДОПУСТИМО» ("MUST NOT"),
«ТРЕБУЕТСЯ» ("REQUIRED"), «НУЖНО» ("SHALL"), «НЕ ПОЗВОЛЯЕТСЯ» ("SHALL NOT"),
«СЛЕДУЕТ» ("SHOULD"), «НЕ СЛЕДУЕТ» ("SHOULD NOT"),
«РЕКОМЕНДУЕТСЯ» ("RECOMMENDED"), «МОЖЕТ» / «ВОЗМОЖНО» ("MAY") и
«НЕОБЯЗАТЕЛЬНО» ("OPTIONAL") в этом документе следует понимать так,
как это описано в [RFC-2119] (и его [переводе]).

[RFC-2119]:  http://www.ietf.org/rfc/rfc2119.txt
[переводе]: http://rfc.com.ru/rfc2119.htm

## Цель

Цель этого PSR — позволить разработчикам создавать библиотеки с поддержкой кэширования, которые
могут быть интегрированы в существующие структуры и системы без необходимости
кастомной разработки.

## Определения

* **Вызов библиотеки** — библиотека или код, которым действительно нужны службы кэширования.
  Эта библиотека будет использовать службы кэширования, реализующие
  стандартные интерфейсы.

* **Реализующая библиотека**. Эта библиотека отвечает за реализацию этого стандарта для предоставления кэширования любой
  вызывающей библиотеке.
  Реализующая библиотека ДОЛЖНА предоставлять классы, реализующие интерфейсы Cache\CacheItemPoolInterface и
  Cache\CacheItemInterface.
  Реализующие библиотеки ДОЛЖНЫ поддерживать минимальную функциональность TTL, как описано ниже, с точностью до целой
  секунды.

* **TTL** — время жизни (TTL (Time To Life)) элемента — это время между хранением этого элемента и моментом, когда он
  считается устаревшим. TTL обычно определяется целым числом, представляющим время в секундах, или объектом
  DateInterval.

* **Срок действия**— фактическое время, когда элемент устареет. Это время
  обычно рассчитывается путем добавления TTL ко времени хранения объекта, но также может быть явно задано с помощью
  объекта DateTime.
  Элемент с 300-секундным сроком жизни, сохраненный в 1:30:00, будет иметь срок действия 1:35:00. Реализующие библиотеки
  МОГУТ сократить срок действия элемента до запрошенного Срока действия, но ДОЛЖНЫ рассматривать элемент как
  просроченный после достижения его Срока действия. Если вызывающая библиотека запрашивает сохранение элемента, но не
  указывает время истечения срока действия или указывает нулевое время истечения срока действия или TTL, реализующая
  библиотека МОЖЕТ использовать настроенную продолжительность по умолчанию. Если продолжительность по умолчанию не
  установлена, реализующая библиотека ДОЛЖНА интерпретировать это как запрос на кэширование элемента навсегда или до тех
  пор, пока поддерживает базовая реализация.

* **Ключ** — строка, состоящая как минимум из одного символа, которая однозначно идентифицирует
  кешированный элемент. Реализующие библиотеки ДОЛЖНЫ поддерживать ключи, состоящие из
  символов `A-Z`, `a-z`, `0-9`, `_` и `.` в любом порядке в кодировке UTF-8 и
  длиной до 64 символов. Реализующие библиотеки МОГУТ поддерживать дополнительные
  символы и кодировки или более длинные, но должны поддерживать по крайней мере этот
  минимум. Библиотеки несут ответственность за собственное экранирование строк ключей.
  В зависимости от ситуации ДОЛЖНА быть возможность возвращать исходную неизмененную строку ключа.
  Следующие символы зарезервированы для будущих расширений и НЕ ДОЛЖНЫ
  поддерживаться реализацией библиотек: `{}()/\@:`

* **Попадание** — попадание в кеш происходит, когда вызывающая библиотека запрашивает элемент по ключу, и для этого
  ключа найдено соответствующее значение, срок действия этого значения не истек, и это значение не является
  недействительным по какой-либо другой причине. Вызывающие библиотеки ДОЛЖНЫ проверять isHit() при всех вызовах get().

* **Промах** — Промах кэша противоположен попаданию в кэш. Промах кэша происходит, когда вызывающая библиотека
  запрашивает элемент по ключу, а это значение не найдено для этого ключа, или значение было найдено, но срок его
  действия истек, или значение недопустимо по какой-либо другой причине. Значение с истекшим сроком действия ДОЛЖНО
  всегда считаться промахом кэша.

* **Отложенное** — отложенное сохранение кэша указывает на то, что элемент кэша не может быть немедленно сохранен пулом.
  Объект пула МОЖЕТ задержать сохранение элемента отложенного кэша, чтобы воспользоваться преимуществами операций
  массового набора, поддерживаемых некоторыми механизмами хранения.
  Пул ДОЛЖЕН гарантировать, что любые элементы отложенного кэша в конечном итоге будут сохранены, а данные не будут
  потеряны, и МОЖЕТ сохранить их до вызывающей библиотеки.
  Когда вызывающая библиотека вызывает метод `commit()`, все незавершенные отложенные элементы ДОЛЖНЫ быть сохранены.
  Реализующая библиотека МОЖЕТ использовать любую подходящую логику для определения того, когда следует сохранять
  отложенные элементы, например, деструктор объекта, сохранение всех при вызове `save()`, проверку тайм-аута или
  максимального количества элементов или любую другую подходящую логику.
  Запросы на отложенный элемент кэша ДОЛЖНЫ возвращать отложенный, но еще не сохраненный элемент.

## Данные

Реализующие библиотеки ДОЛЖНЫ поддерживать все сериализуемые типы данных PHP, включая:

* **Strings** - Строки символов произвольного размера в любой PHP-совместимой кодировке.
* **Integers** - Все целые числа любого размера, поддерживаемые PHP, вплоть до 64-битного со знаком.
* **Floats** - Все значения с плавающей запятой со знаком.
* **Boolean** - `True` или `False`.
* **Null** - Фактическое значение `null`.
* **Arrays** - Индексированные, ассоциативные и многомерные массивы произвольной глубины.
* **Object** - Любой объект, поддерживающий сериализацию и десериализацию без потерь,
  например `$o == unserialize(serialize($o))`. Объекты МОГУТ использовать интерфейс PHP Serializable, магические
  методы `__sleep()` или `__wakeup()` или аналогичные функции языка, если это необходимо.

Все данные, переданные в реализующую библиотеку, ДОЛЖНЫ быть возвращены точно в том виде, в каком они были переданы.
Учитывается тип переменной. То есть возврат (string) 5 является ошибкой, если (int) 5 было переданным значением.
Реализующие библиотеки МОГУТ использовать PHP-функции `serialize()`/`unserialize()` внутри, но не обязаны это делать.
Совместимость с ними просто используется в качестве основы для допустимых значений объекта.

Если по какой-либо причине невозможно вернуть точное сохраненное значение, реализующие библиотеки ДОЛЖНЫ ответить
промахом кэша, а не поврежденными данными.

## Ключевые понятия

### Пул

Пул представляет собой набор элементов в системе кэширования. Пул — это логический репозиторий всех содержащихся в нем
элементов.
Все кэшируемые элементы извлекаются из пула как объект Элемент, и все взаимодействие кэшированных объектов происходит
через пул.

### Элементы

Элемент представляет одну пару ключ/значение в пуле. Ключ является первичным уникальным идентификатором Элемента и
ДОЛЖЕН быть неизменным. Значение МОЖЕТ быть изменено в любое время.

## Обработка ошибок

Хотя кэширование часто является важной частью производительности приложения, оно никогда не должно быть критической
частью функциональности приложения. Таким образом, ошибка в системе кэширования НЕ ДОЛЖНА приводить к сбою приложения.
По этой причине реализующие библиотеки НЕ ДОЛЖНЫ генерировать исключения, кроме тех, которые определены интерфейсом, и
ДОЛЖНЫ перехватывать любые ошибки или исключения, вызванные базовым хранилищем данных, и не допускать их появления.

Реализующая библиотека ДОЛЖНА регистрировать такие ошибки или иным образом сообщать о них администратору.

Если вызывающая библиотека запрашивает удаление одного или нескольких элементов или очистку пула, это НЕ ДОЛЖНО
считаться ошибкой, если указанный ключ не существует. Пост-условие такое же (ключ не существует или пул пуст), поэтому
нет ошибки.

## Интерфейсы

### Интерфейс CacheItemInterface

CacheItemInterface определяет Элемент внутри системы кэширования. Каждый Элемент ДОЛЖЕН быть связан с определенным
ключом, который может быть установлен в соответствии с системой реализации и обычно передается
объектом `Cache\CacheItemPoolInterface`.

Объект `Cache\CacheItemInterface` инкапсулирует хранение и извлечение элементов кэша. Каждый `Cache\CacheItemInterface`
создается объектом `Cache\CacheItemPoolInterface`, который отвечает за любую требуемую настройку, а также связывает
объект с уникальным ключом. Объекты `Cache\CacheItemInterface` ДОЛЖНЫ иметь возможность хранить и извлекать любой тип
значения PHP, определенный в разделе данных этого документа.

Вызов библиотек НЕ ДОЛЖЕН создавать экземпляры самих Элементов. Они могут быть запрошены только из Пула с помощью
метода `getItem()`. Вызывающие библиотеки НЕ ДОЛЖНЫ предполагать, что элемент, созданный одной реализующей библиотекой,
совместим с пулом из другой реализующей библиотеки.

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterface определяет интерфейс для взаимодействия с объектами внутри кеша.
 */
interface CacheItemInterface
{
    /**
     * Возвращает ключ для текущего элемента кэша.
     *
     * Ключ загружается реализующей библиотекой, но при необходимости он должен 
     * быть доступен вызывающим объектам более высокого уровня.
     *
     * @return string
     *   Ключ для этого элемента кэша.
     */
    public function getKey();

    /**
     * Извлекает значение элемента из кэша, связанного с ключом этого объекта.
     *
     * Возвращаемое значение должно быть идентично значению, изначально сохраненному функцией set().
     *
     * Если isHit() возвращает false, этот метод ДОЛЖЕН возвращать null. 
     * Обратите внимание, что значение null является корректным кэшированным значением, 
     * поэтому метод isHit() СЛЕДУЕТ использовать, чтобы различать 
     * «значение null было найдено» и «значение не найдено».
     *
     * @return mixed
     *   Значение, соответствующее ключу этого элемента кэша, или null, если не найдено.
     */
    public function get();

    /**
     * Подтверждает, что поиск элемента кэша привел к попаданию в кэш.
     *
     * Примечание. Этот метод НЕ ДОЛЖЕН иметь "состояние гонки" между вызовом isHit() и вызовом get().
     *
     * @return bool
     *   Истинно, если запрос привел к попаданию в кэш. Ложь в противном случае.
     */
    public function isHit();

    /**
     * Задает значение, представленное этим элементом кэша.
     *
     * Аргумент $value может быть любым элементом, который может быть сериализован PHP, 
     * хотя метод сериализации оставлен на усмотрение библиотеки реализации.
     *
     * @param mixed $value
     *   Сохраняемое сериализуемое значение.
     *
     * @return static
     *   Вызванный объект.
     */
    public function set($value);

    /**
     * Устанавливает время истечения срока действия для этого элемента кэша.
     *
     * @param \DateTimeInterface|null $expiration
     * Момент времени, после которого элемент ДОЛЖЕН считаться просроченным. 
     * Если null передается явно, МОЖЕТ быть использовано значение по умолчанию. 
     * Если ничего не установлено, значение должно храниться постоянно или до тех пор, 
     * пока позволяет реализация.
     *
     * @return static
     *   Вызываемый объект.
     */
    public function expiresAt($expiration);

    /**
     * Устанавливает время истечения срока действия для этого элемента кэша.
     *
     * @param int|\DateInterval|null $time
     *   Период времени от настоящего, после которого элемент ДОЛЖЕН считаться просроченным. 
     * Под целочисленным параметром понимается время в секундах до истечения срока действия. 
     * Если null передается явно, МОЖЕТ быть использовано значение по умолчанию. 
     * Если ничего не установлено, значение должно храниться постоянно или до тех пор, 
     * пока позволяет реализация.
     *
     * @return static
     *   Вызываемый объект.
     */
    public function expiresAfter($time);

}
~~~

### Интерфейс CacheItemPoolInterface

Основная цель Cache\CacheItemPoolInterface — принять ключ из библиотеки вызовов и вернуть связанный объект Cache\CacheItemInterface. Это также основная точка взаимодействия со всем Пулом кеша. 
Вся настройка и инициализация пула возложена на реализующую библиотеку.

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterface создает объекты CacheItemInterface.
 */
interface CacheItemPoolInterface
{
    /**
     * Возвращает элемент кэша, представляющий указанный ключ.
     *
     * Этот метод всегда должен возвращать объект CacheItemInterface, даже в случае промаха кеша. 
     * Он НЕ ДОЛЖЕН возвращать значение null.
     *
     * @param string $key
     *   Ключ, для которого необходимо вернуть соответствующий элемент кэша.
     *
     * @throws InvalidArgumentException
     *   Если строка $key не является допустимым значением, 
     *   НЕОБХОДИМО создать исключение \Psr\Cache\InvalidArgumentException.
     *
     * @return CacheItemInterface
     *   Соответствующий элемент кэша.
     */
    public function getItem($key);

    /**
     * Возвращает набор элементов кэша.
     *
     * @param string[] $keys
     *   Индексированный массив ключей элементов для извлечения.
     *
     * @throws InvalidArgumentException
     *   Если какой-либо из ключей в $keys не является допустимым значением, 
     *   НЕОБХОДИМО создать исключение \Psr\Cache\InvalidArgumentException.
     *
     * @return array|\Traversable
     *   Проходимая коллекция элементов кэша с ключами кэша каждого элемента. 
     *   Элемент кэша будет возвращен для каждого ключа, даже если этот ключ не найден. 
     *   Однако, если ключи не указаны, вместо этого ДОЛЖЕН быть возвращен пустой обходной объект.
     */
    public function getItems(array $keys = array());

    /**
     * Подтверждает, содержит ли кэш указанный элемент кэша.
     *
     * Примечание. Этот метод МОЖЕТ не извлекать кэшированное значение из соображений 
     * производительности. Это может привести к "состоянию гонки" с CacheItemInterface::get(). 
     * Чтобы избежать такой ситуации, используйте вместо этого CacheItemInterface::isHit().
     *
     * @param string $key
     *   Ключ, существование которого необходимо проверить.
     *
     * @throws InvalidArgumentException
     *   Если строка $key не является допустимым значением, 
     *   НЕОБХОДИМО создать исключение \Psr\Cache\InvalidArgumentException.
     *
     * @return bool
     *   True, если элемент существует в кеше, иначе False.
     */
    public function hasItem($key);

    /**
     * Удаляет все элементы в пуле.
     *
     * @return bool
     *   True, если пул был успешно очищен. False, если произошла ошибка.
     */
    public function clear();

    /**
     * Удаляет элемент из пула.
     *
     * @param string $key
     *   Ключ для удаления.
     *
     * @throws InvalidArgumentException
     *   Если строка $key не является допустимым значением, 
     *   НЕОБХОДИМО создать исключение \Psr\Cache\InvalidArgumentException.
     *
     * @return bool
     *   True, если элемент был успешно удален. False, если произошла ошибка.
     */
    public function deleteItem($key);

    /**
     * Удаляет несколько элементов из пула.
     *
     * @param string[] $keys
     *   Массив ключей, которые следует удалить из пула.
     *
     * @throws InvalidArgumentException
     *   Если какой-либо из ключей в $keys не является допустимым значением, 
     *   НЕОБХОДИМО создать исключение \Psr\Cache\InvalidArgumentException.
     *
     * @return bool
     *   True, если элементы были успешно удалены. False, если произошла ошибка.
     */
    public function deleteItems(array $keys);

    /**
     * Немедленно сохраняет элемент кэша.
     *
     * @param CacheItemInterface $item
     *   Элемент кэша для сохранения.
     *
     * @return bool
     *   True, если элемент был успешно сохранен. False, если произошла ошибка.
     */
    public function save(CacheItemInterface $item);

    /**
     * Устанавливает элемент кэша для сохранения позже.
     *
     * @param CacheItemInterface $item
     *    Элемент кэша для сохранения.
     *
     * @return bool
     *   False, если элемент не может быть поставлен в очередь или попытка сохранения не удалась. True в противном случае.
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * Сохраняет любые элементы отложенного кэша.
     *
     * @return bool
     *   False, если элемент не может быть поставлен в очередь или попытка сохранения не удалась. True в противном случае.
     */
    public function commit();
}
~~~

### Интерфейс CacheException

Этот интерфейс предназначен для использования при возникновении критических ошибок,
включая, помимо прочего, *настройку кеша*, например подключение к серверу кеша или предоставление неверных учетных данных.

Любое исключение, создаваемое реализующей библиотекой, ДОЛЖНО реализовывать этот интерфейс.

~~~php
<?php

namespace Psr\Cache;

/**
 * Интерфейс для всех исключений, создаваемых реализующей библиотекой.
 */
interface CacheException
{
}
~~~

### Интерфейс InvalidArgumentException

~~~php
<?php

namespace Psr\Cache;

/**
 * Интерфейс исключения для недопустимых аргументов кеша.
 *
 * Каждый раз, когда в метод передается недопустимый аргумент, 
 * он должен генерировать класс исключения, который реализует Psr\Cache\InvalidArgumentException.
 */
interface InvalidArgumentException extends CacheException
{
}
~~~

Начиная с версии [psr/cache 2.0](https://packagist.org/packages/psr/cache#2.0.0), вышеуказанные интерфейсы были обновлены для добавления подсказки типа аргумента.
Начиная с версии [psr/cache 3.0](https://packagist.org/packages/psr/cache#3.0.0), вышеуказанные интерфейсы были обновлены для добавления подсказки возвращаемого типа. Ссылки на `array|\Traversable` были заменены на `iterable`.

