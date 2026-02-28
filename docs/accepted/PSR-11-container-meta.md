---
description: "Этот документ описывает процесс и обсуждения, которые привели к созданию PSR для контейнера. Его цель — объяснить причины каждого принятого решения."
---

# Мета-документ контейнера


## 1. Введение

Этот документ описывает процесс и обсуждения, которые привели к созданию PSR для контейнера.
Его цель — объяснить причины каждого принятого решения.

## 2. Зачем это нужно?

Существуют десятки контейнеров внедрения зависимостей, и эти
DI-контейнеры имеют очень разные способы хранения элементов.

- Одни основаны на колбэках (Pimple, Laravel, ...)
- Другие основаны на конфигурации (Symfony, ZF, ...) в различных форматах
  (PHP-массивы, YAML-файлы, XML-файлы...)
- Некоторые могут использовать фабрики...
- Некоторые имеют PHP API для создания элементов (PHP-DI, ZF, Symfony, Mouf...)
- Некоторые поддерживают автоматическое связывание (Laravel, PHP-DI, ...)
- Другие могут связывать элементы на основе аннотаций (PHP-DI, JMS Bundle...)
- Некоторые имеют графический пользовательский интерфейс (Mouf...)
- Некоторые могут компилировать конфигурационные файлы в PHP-классы (Symfony, ZF...)
- Некоторые поддерживают псевдонимы...
- Некоторые могут использовать прокси для ленивой загрузки зависимостей...

Таким образом, если смотреть в целом, существует очень большое количество способов
решения проблемы DI, и, следовательно, большое число различных реализаций. Однако все
существующие DI-контейнеры отвечают на одну и ту же потребность: они предоставляют
приложению способ получить набор сконфигурированных объектов (как правило, сервисов).

Стандартизировав способ получения элементов из контейнера, фреймворки и
библиотеки, использующие PSR для контейнера, смогут работать с любым совместимым контейнером.
Это позволит конечным пользователям выбирать контейнер по собственному усмотрению.

## 3. Область применения
### 3.1. Цели

Цель PSR для контейнера — стандартизировать, как фреймворки и библиотеки используют
контейнер для получения объектов и параметров.

Важно различать два способа использования контейнера:

- конфигурирование элементов
- получение элементов

В большинстве случаев эти две стороны использует не одна и та же сторона.
Хотя конфигурированием элементов, как правило, занимаются конечные пользователи,
получением элементов для построения приложения обычно занимается фреймворк.

Именно поэтому данный интерфейс сосредоточен исключительно на том, как элементы
могут быть получены из контейнера.

### 3.2. Вне области применения

Способ установки элементов в контейнер и их конфигурирование выходят за рамки
данного PSR. Именно это делает реализацию контейнера уникальной. Одни
контейнеры вообще не имеют конфигурации (они используют автоматическое связывание), другие
основаны на PHP-коде, заданном через колбэки, третьи — на конфигурационных файлах... Этот стандарт
сосредоточен только на способе получения элементов.

Кроме того, соглашения об именовании элементов не входят в область применения данного
PSR. Действительно, если рассматривать соглашения об именовании, существует 2 стратегии:

- идентификатор является именем класса или именем интерфейса (используется преимущественно
  фреймворками с поддержкой автоматического связывания)
- идентификатор является общим именем (ближе к имени переменной), что
  преимущественно используется фреймворками, опирающимися на конфигурацию.

Обе стратегии имеют свои сильные и слабые стороны. Цель данного PSR
не в том, чтобы отдать предпочтение одному соглашению перед другим. Вместо этого пользователь может
использовать псевдонимы для устранения разрыва между двумя контейнерами с разными стратегиями именования.

## 4. Рекомендуемое использование: PSR для контейнера и Service Locator

PSR гласит:

> «пользователи НЕ ДОЛЖНЫ передавать контейнер в объект, чтобы объект
> мог получать *собственные зависимости*. Пользователи, поступающие таким образом, используют контейнер как Service Locator.
> Использование Service Locator в целом не рекомендуется.»

```php
// This is not OK, you are using the container as a service locator
class BadExample
{
    public function __construct(ContainerInterface $container)
    {
        $this->db = $container->get('db');
    }
}

// Instead, please consider injecting directly the dependencies
class GoodExample
{
    public function __construct($db)
    {
        $this->db = $db;
    }
}
// You can then use the container to inject the $db object into your $goodExample object.
```

В случае `BadExample` не следует внедрять контейнер, потому что:

- это делает код **менее интероперабельным**: внедряя контейнер, вы вынуждены
  использовать контейнер, совместимый с PSR для контейнера. При другом подходе
  ваш код может работать с ЛЮБЫМ контейнером.
- вы вынуждаете разработчика называть элемент «db». Такое имя может
  конфликтовать с другим пакетом, предъявляющим те же ожидания к другому сервису.
- это затрудняет тестирование.
- из кода неочевидно, что класс `BadExample` будет нуждаться
  в сервисе «db». Зависимости скрыты.

Очень часто `ContainerInterface` будет использоваться другими пакетами. Для конечного
PHP-разработчика, работающего с фреймворком, маловероятно, что ему когда-либо придётся
использовать контейнеры или объявлять зависимость непосредственно от `ContainerInterface`.

Является ли использование PSR для контейнера в вашем коде хорошей практикой или нет, сводится к
пониманию того, являются ли получаемые объекты **зависимостями** объекта, обращающегося к
контейнеру. Вот ещё несколько примеров:

```php
class RouterExample
{
    // ...

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function getRoute($request)
    {
        $controllerName = $this->getContainerEntry($request->getUrl());
        // This is OK, the router is finding the matching controller entry, the controller is
        // not a dependency of the router
        $controller = $this->container->get($controllerName);
        // ...
    }
}
```

В этом примере роутер преобразует URL в имя элемента контроллера,
а затем получает контроллер из контейнера. Контроллер не является настоящей
зависимостью роутера. Как правило, если ваш объект *вычисляет*
имя элемента из списка элементов, которые могут варьироваться, такое использование вполне оправдано.

В виде исключения фабричные объекты, единственная цель которых — создавать и возвращать новые экземпляры, МОГУТ
использовать паттерн Service Locator. Фабрика ДОЛЖНА реализовывать интерфейс, чтобы её можно было
заменить другой фабрикой с тем же интерфейсом.

```php
// ok: a factory interface + implementation to create an object
interface FactoryInterface
{
    public function newInstance();
}

class ExampleFactory implements FactoryInterface
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function newInstance()
    {
        return new Example($this->container->get('db'));
    }
}
```

## 5. История

До передачи PSR для контейнера в PHP-FIG, `ContainerInterface` был
впервые предложен в проекте [container-interop](https://github.com/container-interop/container-interop/).
Цель проекта состояла в создании полигона для реализации `ContainerInterface`
и в подготовке почвы для PSR контейнера.

В остальной части этого мета-документа вы найдёте частые ссылки на
`container-interop.`

## 6. Имя интерфейса

Имя интерфейса совпадает с тем, что обсуждалось для `container-interop`
(изменено только пространство имён, чтобы соответствовать другим PSR).
Оно было тщательно обсуждено в `container-interop` [[4]](#link_naming_discussion) и определено голосованием [[5]](#link_naming_vote).

Список рассмотренных вариантов с результатами голосования:

- `ContainerInterface`: +8
- `ProviderInterface`: +2
- `LocatorInterface`: 0
- `ReadableContainerInterface`: -5
- `ServiceLocatorInterface`: -6
- `ObjectFactory`: -6
- `ObjectStore`: -8
- `ConsumerInterface`: -9

## 7. Методы интерфейса

Выбор методов, которые должен содержать интерфейс, был сделан после статистического анализа существующих контейнеров [[6]](#link_statistical_analysis).

Краткие итоги анализа показали:

- все контейнеры предоставляют метод для получения элемента по его идентификатору
- большинство называют такой метод `get()`
- у всех контейнеров метод `get()` имеет 1 обязательный параметр типа string
- некоторые контейнеры имеют необязательный дополнительный аргумент у `get()`, но его назначение различается от контейнера к контейнеру
- большинство контейнеров предоставляют метод для проверки возможности вернуть элемент по его идентификатору
- большинство называют такой метод `has()`
- у всех контейнеров, предоставляющих `has()`, метод имеет ровно 1 параметр типа string
- большинство контейнеров выбрасывают исключение, а не возвращают null, когда элемент не найден в `get()`
- большинство контейнеров не реализуют `ArrayAccess`

Вопрос о включении методов для определения элементов обсуждался с самого начала проекта container-interop [[4]](#link_naming_discussion).
Было признано, что такие методы не относятся к описанному здесь интерфейсу, поскольку выходят за его рамки
(см. раздел «Цели»).

В результате `ContainerInterface` содержит два метода:

- `get()`, возвращающий произвольное значение, с одним обязательным строковым параметром. ДОЛЖЕН выбрасывать исключение, если элемент не найден.
- `has()`, возвращающий булево значение, с одним обязательным строковым параметром.

### 7.1. Количество параметров метода `get()`

Хотя `ContainerInterface` определяет только один обязательный параметр в `get()`, это не противоречит
существующим контейнерам с дополнительными необязательными параметрами. PHP позволяет реализации предлагать
больше параметров, если они являются необязательными, поскольку такая реализация *действительно* удовлетворяет интерфейсу.

Отличие от container-interop: [спецификация container-interop](https://github.com/container-interop/container-interop/blob/master/docs/ContainerInterface.md) гласила:

> Хотя `ContainerInterface` определяет только один обязательный параметр в `get()`, реализации МОГУТ принимать дополнительные необязательные параметры.

Это предложение было исключено из PSR-11, поскольку:

- это вытекает из принципов ООП в PHP и напрямую не связано с PSR-11
- мы не хотим поощрять реализующих добавлять дополнительные параметры, так как рекомендуем писать код против интерфейса, а не реализации

Тем не менее, некоторые реализации имеют дополнительные необязательные параметры — это технически допустимо. Такие реализации совместимы с PSR-11. [[11]](#link_get_optional_parameters)

### 7.2. Тип параметра `$id`

Тип параметра `$id` в методах `get()` и `has()` обсуждался в проекте container-interop.

Хотя во всех проанализированных контейнерах используется тип `string`, было предложено
разрешить любой тип (например, объекты), чтобы позволить контейнерам предлагать более расширенный API запросов.

В качестве примера рассматривалось использование контейнера как строителя объектов. Параметр `$id` в этом случае
был бы объектом, описывающим, как создать экземпляр.

Вывод обсуждения [[7]](#link_method_and_parameters_details) состоял в том, что это выходит за рамки
получения элементов из контейнера без знания о том, как контейнер их предоставляет, и больше подходит для фабрики.

### 7.3. Выбрасываемые исключения

Данный PSR предоставляет 2 интерфейса, предназначенных для реализации исключениями контейнера.

#### 7.3.1 Базовое исключение

`Psr\Container\ContainerExceptionInterface` — базовый интерфейс. Его СЛЕДУЕТ реализовывать в пользовательских исключениях, выбрасываемых непосредственно контейнером.

Ожидается, что любое исключение, относящееся к предметной области контейнера, реализует `ContainerExceptionInterface`. Несколько примеров:

- если контейнер использует конфигурационный файл и этот файл некорректен, контейнер МОЖЕТ выбрасывать `InvalidFileException`, реализующий `ContainerExceptionInterface`.
- если обнаруживается циклическая зависимость между зависимостями, контейнер МОЖЕТ выбрасывать `CyclicDependencyException`, реализующий `ContainerExceptionInterface`.

Однако если исключение выбрасывается кодом вне области контейнера (например, исключение при создании экземпляра элемента), контейнер не обязан оборачивать это исключение в пользовательское исключение, реализующее `ContainerExceptionInterface`.

Полезность базового интерфейса исключения ставилась под сомнение: это не то исключение, которое обычно перехватывают [[8]](#link_base_exception_usefulness).

Тем не менее большинство членов PHP-FIG сочли это лучшей практикой. Базовые интерфейсы исключений реализованы в предыдущих PSR и в нескольких проектах-участниках. Базовый интерфейс исключения был поэтому сохранён.

#### 7.3.2 Исключение «не найдено»

Вызов метода `get` с несуществующим идентификатором ДОЛЖЕН выбрасывать исключение, реализующее `Psr\Container\NotFoundExceptionInterface`.

Для заданного идентификатора:

- если метод `has` возвращает `false`, то метод `get` ДОЛЖЕН выбрасывать `Psr\Container\NotFoundExceptionInterface`.
- если метод `has` возвращает `true`, это не означает, что метод `get` завершится успешно и не выбросит исключение. Он даже МОЖЕТ выбросить `Psr\Container\NotFoundExceptionInterface`, если отсутствует одна из зависимостей запрошенного элемента.

Таким образом, когда пользователь перехватывает `Psr\Container\NotFoundExceptionInterface`, это МОЖЕТ означать [[9]](#link_not_found_behaviour):

- запрошенный элемент не существует (неверный запрос)
- или зависимость запрошенного элемента не существует (т. е. контейнер неправильно сконфигурирован)

Пользователь, однако, может легко разграничить эти случаи с помощью вызова `has`.

В псевдокоде:

```php
if (!$container->has($id)) {
    // The requested instance does not exist
    return;
}
try {
    $entry = $container->get($id);
} catch (NotFoundExceptionInterface $e) {
    // Since the requested entry DOES exist, a NotFoundExceptionInterface means that the container is misconfigured and a dependency is missing.
}
```

## 8. Реализации

На момент написания следующие проекты уже реализуют и/или потребляют версию интерфейса из `container-interop`.

### Реализующие

- [Acclimate](https://github.com/jeremeamia/acclimate-container)
- [Aura.DI](https://github.com/auraphp/Aura.Di)
- [dcp-di](https://github.com/estelsmith/dcp-di)
- [League Container](https://github.com/thephpleague/container)
- [Mouf](http://mouf-php.com)
- [Njasm Container](https://github.com/njasm/container)
- [PHP-DI](http://php-di.org)
- [PimpleInterop](https://github.com/moufmouf/pimple-interop)
- [XStatic](https://github.com/jeremeamia/xstatic)
- [Zend ServiceManager](https://github.com/zendframework/zend-servicemanager)

### Промежуточные пакеты

- [Alias-Container](https://github.com/thecodingmachine/alias-container)
- [Prefixer-Container](https://github.com/thecodingmachine/prefixer-container)

### Потребители

- [Behat](https://github.com/Behat/Behat)
- [interop.silex.di](https://github.com/thecodingmachine/interop.silex.di)
- [mindplay/middleman](https://github.com/mindplay-dk/middleman)
- [PHP-DI Invoker](https://github.com/PHP-DI/Invoker)
- [Prophiler](https://github.com/fabfuel/prophiler)
- [Silly](https://github.com/mnapoli/silly)
- [Slim](https://github.com/slimphp/Slim)
- [Splash](http://mouf-php.com/packages/mouf/mvc.splash-common/version/8.0-dev/README.md)
- [Zend Expressive](https://github.com/zendframework/zend-expressive)

Этот список не является исчерпывающим и приводится лишь как пример, демонстрирующий значительный интерес к данному PSR.

## 9. Участники

### 9.1 Редакторы

* [Matthieu Napoli](https://github.com/mnapoli)
* [David Négrier](https://github.com/moufmouf)

### 9.2 Спонсоры

* [Matthew Weier O'Phinney](https://github.com/weierophinney) (Координатор)
* [Korvin Szanto](https://github.com/KorvinSzanto)

### 9.3 Участники обсуждений

Здесь перечислены все люди, принимавшие участие в обсуждениях или голосованиях (в container-interop и в ходе миграции на PSR-11), в алфавитном порядке:

* [Alexandru Pătrănescu](https://github.com/drealecs)
* [Amy Stephen](https://github.com/AmyStephen)
* [Ben Peachey](https://github.com/potherca)
* [David Négrier](https://github.com/moufmouf)
* [Don Gilbert](https://github.com/dongilbert)
* [Jason Judge](https://github.com/judgej)
* [Jeremy Lindblom](https://github.com/jeremeamia)
* [Larry Garfield](https://github.com/crell)
* [Marco Pivetta](https://github.com/Ocramius)
* [Matthieu Napoli](https://github.com/mnapoli)
* [Nelson J Morais](https://github.com/njasm)
* [Paul M. Jones](https://github.com/pmjones)
* [Phil Sturgeon](https://github.com/philsturgeon)
* [Stephan Hochdörfer](https://github.com/shochdoerfer)
* [Taylor Otwell](https://github.com/taylorotwell)

## 10. Ссылки

1. [Обсуждение PSR для контейнера и Service Locator](https://groups.google.com/forum/#!topic/php-fig/pyTXRvLGpsw)
1. [`ContainerInterface.php` из container-interop](https://github.com/container-interop/container-interop/blob/master/src/Interop/Container/ContainerInterface.php)
1. [Список всех задач](https://github.com/container-interop/container-interop/issues?labels=ContainerInterface&milestone=&page=1&state=closed)
1. <a name="link_naming_discussion"></a>[Обсуждение имени интерфейса и области применения container-interop](https://github.com/container-interop/container-interop/issues/1)
1. <a name="link_naming_vote"></a>[Голосование за имя интерфейса](https://github.com/container-interop/container-interop/wiki/%231-interface-name:-Vote)
1. <a name="link_statistical_analysis"></a>[Статистический анализ имён методов существующих контейнеров](https://gist.github.com/mnapoli/6159681)
1. <a name="link_method_and_parameters_details"></a>[Обсуждение имён методов и параметров](https://github.com/container-interop/container-interop/issues/6)
1. <a name="link_base_exception_usefulness"></a>[Обсуждение полезности базового интерфейса исключения](https://groups.google.com/forum/#!topic/php-fig/_vdn5nLuPBI)
1. <a name="link_not_found_behaviour"></a>[Обсуждение поведения `NotFoundExceptionInterface`](https://groups.google.com/forum/#!topic/php-fig/I1a2Xzv9wN8)
1. <a name="link_get_optional_parameters"></a>Обсуждение необязательных параметров get [в container-interop](https://github.com/container-interop/container-interop/issues/6) и в [списке рассылки PHP-FIG](https://groups.google.com/forum/#!topic/php-fig/zY6FAG4-oz8)

## 11. Исправления

## Добавление типов

Версия 1.1 пакета `psr/container` включает скалярные типы параметров. Версия
2.0 пакета включает типы возвращаемых значений. Данная структура использует поддержку
ковариантности PHP 7.2, чтобы обеспечить поэтапный процесс обновления.

Реализующие МОГУТ добавлять типы возвращаемых значений в собственные пакеты по своему усмотрению
при условии, что:

- типы возвращаемых значений совпадают с теми, что указаны в пакете версии 2.0.
- реализация указывает минимальную версию PHP 7.2.0 или выше.

Реализующие МОГУТ добавлять типы параметров в собственные пакеты в новом мажорном
выпуске — одновременно с добавлением типов возвращаемых значений или в последующем
выпуске — при условии, что:

- типы параметров совпадают с теми, что указаны в пакете версии 1.1.
- реализация указывает минимальную версию PHP 7.2.0.
- реализация зависит от `"psr/container": "^1.1 || ^2.0"`, чтобы исключить
  нетипизированную версию 1.0.

Реализующим рекомендуется, но не требуется, как можно скорее переходить
своими пакетами на версию 2.0 пакета.
