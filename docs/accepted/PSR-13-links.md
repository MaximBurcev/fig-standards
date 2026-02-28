# Интерфейсы определения ссылок



Гипермедиа-ссылки становятся всё более важной частью веба — как в контексте HTML,
так и в различных форматах API. Однако не существует единого общепринятого формата гипермедиа
и единого способа представления ссылок между форматами.

Данная спецификация ставит целью предоставить PHP-разработчикам простой и универсальный способ
представления гипермедиа-ссылки независимо от используемого формата сериализации. Это, в свою
очередь, позволяет системе сериализовать ответ с гипермедиа-ссылками в один или несколько
проводных форматов независимо от процесса принятия решения о том, какими эти ссылки должны быть.

Ключевые слова «ОБЯЗАН» («MUST»), «НЕ ДОЛЖЕН» («MUST NOT»), «ТРЕБУЕТСЯ» («REQUIRED»),
«ОБЯЗАН» («SHALL»), «НЕ ОБЯЗАН» («SHALL NOT»), «СЛЕДУЕТ» («SHOULD»),
«НЕ СЛЕДУЕТ» («SHOULD NOT»), «РЕКОМЕНДУЕТСЯ» («RECOMMENDED»), «МОЖЕТ» («MAY»)
и «НЕОБЯЗАТЕЛЬНО» («OPTIONAL») в данном документе следует интерпретировать
так, как описано в [RFC 2119](http://tools.ietf.org/html/rfc2119).

### Ссылки

- [RFC 2119](http://tools.ietf.org/html/rfc2119)
- [RFC 4287](https://tools.ietf.org/html/rfc4287)
- [RFC 5988](https://tools.ietf.org/html/rfc5988)
- [RFC 6570](https://tools.ietf.org/html/rfc6570)
- [Реестр отношений ссылок IANA](http://www.iana.org/assignments/link-relations/link-relations.xhtml)
- [Список отношений Microformats](http://microformats.org/wiki/existing-rel-values#HTML5_link_type_extensions)

## 1. Спецификация

### 1.1 Базовые ссылки

Гипермедиа-ссылка состоит как минимум из:
- URI, представляющего целевой ресурс, на который делается ссылка.
- Отношения, определяющего, как целевой ресурс соотносится с источником.

У ссылки могут существовать и другие атрибуты в зависимости от используемого формата. Поскольку
дополнительные атрибуты не стандартизированы и не являются универсальными, данная спецификация
не ставит целью их стандартизацию.

В целях настоящей спецификации применяются следующие определения.

*    **Реализующий объект** — объект, реализующий один из интерфейсов, определённых данной
спецификацией.

*    **Сериализатор** — библиотека или иная система, которая принимает один или несколько объектов
ссылок и создаёт их сериализованное представление в некотором определённом формате.

### 1.2 Атрибуты

Все ссылки МОГУТ включать ноль или более дополнительных атрибутов помимо URI и отношения.
Формального реестра допустимых значений не существует, и валидность значений
зависит от контекста и нередко от конкретного формата сериализации. Часто встречающиеся
значения — `hreflang`, `title` и `type`.

Сериализаторы МОГУТ опускать атрибуты объекта ссылки, если этого требует формат сериализации.
Однако сериализаторы СЛЕДУЕТ кодировать все предоставленные атрибуты там, где это возможно,
чтобы допускать расширения пользователем, если только определение формата сериализации
не препятствует этому.

Некоторые атрибуты (часто `hreflang`) могут появляться в своём контексте более одного раза.
Поэтому значение атрибута МОЖЕТ быть массивом значений, а не единственным значением. Сериализаторы
МОГУТ кодировать такой массив в любом формате, подходящем для данного формата сериализации
(например, в виде списка, разделённого пробелами, запятыми и т. д.). Если атрибут не допускает
нескольких значений в конкретном контексте, сериализаторы ДОЛЖНЫ использовать первое
предоставленное значение и игнорировать все последующие.

Если значение атрибута является булевым `true`, сериализаторы МОГУТ использовать сокращённые
формы там, где это уместно и поддерживается форматом сериализации. Например, HTML допускает
атрибуты без значения, когда само присутствие атрибута имеет булевый смысл. Это правило
применяется тогда и только тогда, когда атрибут равен булевому `true`, но не для любого другого
«истинного» значения в PHP, например целого числа 1.

Если значение атрибута является булевым `false`, сериализаторы СЛЕДУЕТ полностью опускать атрибут,
если только это не изменяет семантического смысла результата. Это правило применяется тогда и
только тогда, когда атрибут равен булевому `false`, но не для любого другого «ложного» значения
в PHP, например целого числа 0.

### 1.3 Отношения

Отношения ссылок определяются как строки и представляют собой либо простое ключевое слово
в случае публично определённого отношения, либо абсолютный URI в случае приватного отношения.

Если используется простое ключевое слово, оно СЛЕДУЕТ соответствовать одному из записей реестра
IANA по адресу:

http://www.iana.org/assignments/link-relations/link-relations.xhtml

Опционально МОЖЕТ использоваться реестр microformats.org, однако это может быть неприемлемо
в каждом контексте:

http://microformats.org/wiki/existing-rel-values

Отношение, не определённое ни в одном из перечисленных реестров или аналогичном публичном реестре,
считается «приватным», то есть специфичным для конкретного приложения или варианта использования.
Такие отношения ДОЛЖНЫ использовать абсолютный URI.

## 1.4 Шаблоны ссылок

[RFC 6570](https://tools.ietf.org/html/rfc6570) определяет формат шаблонов URI, то есть
шаблон URI, который предполагается заполнить значениями, предоставляемыми клиентским инструментом.
Некоторые форматы гипермедиа поддерживают шаблонные ссылки, другие — нет и могут иметь особый
способ обозначения того, что ссылка является шаблоном. Сериализатор для формата, не поддерживающего
шаблоны URI, ДОЛЖЕН игнорировать любые шаблонные ссылки, с которыми он сталкивается.

## 1.5 Расширяемые провайдеры

В некоторых случаях провайдер ссылок может нуждаться в возможности добавления к нему
дополнительных ссылок. В других случаях провайдер ссылок по необходимости является
доступным только для чтения, а ссылки формируются во время выполнения из какого-либо
другого источника данных. По этой причине модифицируемые провайдеры выделены в
дополнительный интерфейс, который может быть реализован опционально.

Кроме того, некоторые объекты провайдера ссылок, например объекты PSR-7 Response, по своей
конструкции являются неизменяемыми. Это означает, что методы добавления ссылок к ним
«на месте» были бы несовместимы с этим подходом. Поэтому единственный метод интерфейса
`EvolvableLinkProviderInterface` требует возврата нового объекта, идентичного оригинальному,
но с включённым дополнительным объектом ссылки.

## 1.6 Расширяемые объекты ссылок

Объекты ссылок в большинстве случаев являются объектами-значениями. Поэтому возможность их
изменения по той же схеме, что и объектов-значений PSR-7, является полезной опцией. По этой
причине предусмотрен дополнительный интерфейс `EvolvableLinkInterface`, предоставляющий методы
для создания новых экземпляров объекта с одним изменением. Та же модель используется в PSR-7
и, благодаря механизму copy-on-write в PHP, остаётся эффективной с точки зрения использования
процессора и памяти.

Метода расширения для шаблонных значений не предусмотрено, поскольку шаблонное значение ссылки
определяется исключительно значением href. Оно НЕ ДОЛЖНО устанавливаться независимо, а
должно выводиться из того, является ли значение href шаблоном ссылки RFC 6570.

## 2. Пакет

Описанные интерфейсы и классы предоставляются в составе пакета
[psr/link](https://packagist.org/packages/psr/link).

## 3. Интерфейсы

### 3.1 `Psr\Link\LinkInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * A readable link object.
 */
interface LinkInterface
{
    /**
     * Returns the target of the link.
     *
     * The target link must be one of:
     * - An absolute URI, as defined by RFC 5988.
     * - A relative URI, as defined by RFC 5988. The base of the relative link
     *     is assumed to be known based on context by the client.
     * - A URI template as defined by RFC 6570.
     *
     * If a URI template is returned, isTemplated() MUST return True.
     *
     * @return string
     */
    public function getHref();

    /**
     * Returns whether or not this is a templated link.
     *
     * @return bool
     *   True if this link object is templated, False otherwise.
     */
    public function isTemplated();

    /**
     * Returns the relationship type(s) of the link.
     *
     * This method returns 0 or more relationship types for a link, expressed
     * as an array of strings.
     *
     * @return string[]
     */
    public function getRels();

    /**
     * Returns a list of attributes that describe the target URI.
     *
     * @return array
     *   A key-value list of attributes, where the key is a string and the value
     *  is either a PHP primitive or an array of PHP strings. If no values are
     *  found an empty array MUST be returned.
     */
    public function getAttributes();
}
~~~

### 3.2 `Psr\Link\EvolvableLinkInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * An evolvable link value object.
 */
interface EvolvableLinkInterface extends LinkInterface
{
    /**
     * Returns an instance with the specified href.
     *
     * @param string $href
     *   The href value to include. It must be one of:
     *     - An absolute URI, as defined by RFC 5988.
     *     - A relative URI, as defined by RFC 5988. The base of the relative link
     *       is assumed to be known based on context by the client.
     *     - A URI template as defined by RFC 6570.
     *     - An object implementing __toString() that produces one of the above
     *       values.
     *
     * An implementing library SHOULD evaluate a passed object to a string
     * immediately rather than waiting for it to be returned later.
     *
     * @return static
     */
    public function withHref($href);

    /**
     * Returns an instance with the specified relationship included.
     *
     * If the specified rel is already present, this method MUST return
     * normally without errors, but without adding the rel a second time.
     *
     * @param string $rel
     *   The relationship value to add.
     * @return static
     */
    public function withRel($rel);

    /**
     * Returns an instance with the specified relationship excluded.
     *
     * If the specified rel is already not present, this method MUST return
     * normally without errors.
     *
     * @param string $rel
     *   The relationship value to exclude.
     * @return static
     */
    public function withoutRel($rel);

    /**
     * Returns an instance with the specified attribute added.
     *
     * If the specified attribute is already present, it will be overwritten
     * with the new value.
     *
     * @param string $attribute
     *   The attribute to include.
     * @param string $value
     *   The value of the attribute to set.
     * @return static
     */
    public function withAttribute($attribute, $value);

    /**
     * Returns an instance with the specified attribute excluded.
     *
     * If the specified attribute is not present, this method MUST return
     * normally without errors.
     *
     * @param string $attribute
     *   The attribute to remove.
     * @return static
     */
    public function withoutAttribute($attribute);
}
~~~

#### 3.2 `Psr\Link\LinkProviderInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * A link provider object.
 */
interface LinkProviderInterface
{
    /**
     * Returns an iterable of LinkInterface objects.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * are available, an empty array or \Traversable MUST be returned.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinks();

    /**
     * Returns an iterable of LinkInterface objects that have a specific relationship.
     *
     * The iterable may be an array or any PHP \Traversable object. If no links
     * with that relationship are available, an empty array or \Traversable MUST be returned.
     *
     * @return LinkInterface[]|\Traversable
     */
    public function getLinksByRel($rel);
}
~~~

#### 3.3 `Psr\Link\EvolvableLinkProviderInterface`

~~~php
<?php

namespace Psr\Link;

/**
 * An evolvable link provider value object.
 */
interface EvolvableLinkProviderInterface extends LinkProviderInterface
{
    /**
     * Returns an instance with the specified link included.
     *
     * If the specified link is already present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   A link object that should be included in this collection.
     * @return static
     */
    public function withLink(LinkInterface $link);

    /**
     * Returns an instance with the specified link removed.
     *
     * If the specified link is not present, this method MUST return normally
     * without errors. The link is present if $link is === identical to a link
     * object already in the collection.
     *
     * @param LinkInterface $link
     *   The link to remove.
     * @return static
     */
    public function withoutLink(LinkInterface $link);
}
~~~

Начиная с [psr/link версии 1.1](https://packagist.org/packages/psr/link#1.1.0), в приведённые интерфейсы были добавлены подсказки типов для аргументов.
Начиная с [psr/link версии 2.0](https://packagist.org/packages/psr/link#2.0.0), в приведённые интерфейсы были добавлены подсказки типов для возвращаемых значений. Ссылки на `array|\Traversable` заменены на `iterable`.
