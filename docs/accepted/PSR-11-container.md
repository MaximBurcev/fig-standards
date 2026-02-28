# Интерфейс контейнера



Этот документ описывает общий интерфейс для контейнеров внедрения зависимостей.

Цель `ContainerInterface` — стандартизировать способ, которым фреймворки и библиотеки используют
контейнер для получения объектов и параметров (далее в этом документе именуемых *элементами*).

Ключевые слова «ДОЛЖЕН» ("MUST"), «НЕ ДОЛЖЕН» ("MUST NOT"), «ТРЕБУЕТСЯ» ("REQUIRED"),
«НУЖНО» ("SHALL"), «НЕ ПОЗВОЛЯЕТСЯ» ("SHALL NOT"), «СЛЕДУЕТ» ("SHOULD"),
«НЕ СЛЕДУЕТ» ("SHOULD NOT"), «РЕКОМЕНДУЕТСЯ» ("RECOMMENDED"), «МОЖЕТ» ("MAY")
и «НЕОБЯЗАТЕЛЬНО» ("OPTIONAL") в этом документе следует понимать так,
как это описано в [RFC 2119][].

Слово `реализующий` в этом документе следует интерпретировать как того,
кто реализует `ContainerInterface` в библиотеке или фреймворке, связанном с внедрением зависимостей.
Пользователи контейнеров внедрения зависимостей (DIC) именуются `пользователь`.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

## 1. Спецификация

### 1.1 Основы

#### 1.1.1 Идентификаторы элементов

Идентификатор элемента — это любая допустимая в PHP строка длиной не менее одного символа,
однозначно идентифицирующая элемент внутри контейнера. Идентификатор элемента является
непрозрачной строкой, поэтому вызывающий код НЕ ДОЛЖЕН предполагать, что структура строки
несёт какой-либо семантический смысл.

#### 1.1.2 Чтение из контейнера

- `Psr\Container\ContainerInterface` предоставляет два метода: `get` и `has`.

- `get` принимает один обязательный параметр: идентификатор элемента, который ДОЛЖЕН быть строкой.
  `get` может возвращать что угодно (*mixed*-значение) или выбрасывать `NotFoundExceptionInterface`,
  если идентификатор неизвестен контейнеру. Два последовательных вызова `get` с одним и тем же
  идентификатором СЛЕДУЕТ возвращать одно и то же значение. Однако в зависимости от конструкции
  `реализующего` и/или конфигурации `пользователя` могут возвращаться разные значения, поэтому
  `пользователь` НЕ ДОЛЖЕН рассчитывать на получение одного и того же значения при двух последовательных вызовах.

- `has` принимает один параметр: идентификатор элемента, который ДОЛЖЕН быть строкой.
  `has` ДОЛЖЕН возвращать `true`, если идентификатор элемента известен контейнеру, и `false` — если нет.
  Если `has($id)` возвращает false, то `get($id)` ДОЛЖЕН выбрасывать `NotFoundExceptionInterface`.

### 1.2 Исключения

Исключения, выбрасываемые непосредственно контейнером, СЛЕДУЕТ реализовывать через
[`Psr\Container\ContainerExceptionInterface`](#container-exception).

Вызов метода `get` с несуществующим идентификатором ДОЛЖЕН выбрасывать
[`Psr\Container\NotFoundExceptionInterface`](#not-found-exception).

### 1.3 Рекомендуемое использование

Пользователи НЕ ДОЛЖНЫ передавать контейнер в объект, чтобы объект мог извлекать *собственные зависимости*.
Это означает, что контейнер используется как [Service Locator](https://en.wikipedia.org/wiki/Service_locator_pattern)
— паттерн, который в целом не рекомендуется.

Подробнее см. раздел 4 мета-документа.

## 2. Пакет

Описанные интерфейсы, классы и соответствующие исключения предоставляются в составе пакета
[psr/container](https://packagist.org/packages/psr/container).

Пакеты, предоставляющие реализацию PSR-контейнера, ДОЛЖНЫ объявлять, что они предоставляют `psr/container-implementation` `1.0.0`.

Проекты, требующие реализацию, ДОЛЖНЫ указывать зависимость `psr/container-implementation` `1.0.0`.

## 3. Интерфейсы

<a name="container-interface"></a>
### 3.1. `Psr\Container\ContainerInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Describes the interface of a container that exposes methods to read its entries.
 */
interface ContainerInterface
{
    /**
     * Finds an entry of the container by its identifier and returns it.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @throws NotFoundExceptionInterface  No entry was found for **this** identifier.
     * @throws ContainerExceptionInterface Error while retrieving the entry.
     *
     * @return mixed Entry.
     */
    public function get($id);

    /**
     * Returns true if the container can return an entry for the given identifier.
     * Returns false otherwise.
     *
     * `has($id)` returning true does not mean that `get($id)` will not throw an exception.
     * It does however mean that `get($id)` will not throw a `NotFoundExceptionInterface`.
     *
     * @param string $id Identifier of the entry to look for.
     *
     * @return bool
     */
    public function has($id);
}
~~~


Начиная с [psr/container версии 1.1](https://packagist.org/packages/psr/container#1.1.0),
в вышеуказанный интерфейс добавлены подсказки типов для аргументов.

Начиная с [psr/container версии 2.0](https://packagist.org/packages/psr/container#2.0.0),
в вышеуказанный интерфейс добавлены подсказки типов для возвращаемых значений (но только для метода
`has()`).

<a name="container-exception"></a>
### 3.2. `Psr\Container\ContainerExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * Base interface representing a generic exception in a container.
 */
interface ContainerExceptionInterface
{
}
~~~

<a name="not-found-exception"></a>
### 3.3. `Psr\Container\NotFoundExceptionInterface`

~~~php
<?php
namespace Psr\Container;

/**
 * No entry was found in the container.
 */
interface NotFoundExceptionInterface extends ContainerExceptionInterface
{
}
~~~
