HTTP-клиент
===========


В этом документе описывается общий интерфейс для отправки HTTP-запросов и получения HTTP-ответов.

Ключевые слова «ДОЛЖЕН», «НЕ ДОЛЖЕН», «ТРЕБУЕТСЯ», «ДОЛЖЕН», «НЕ ДОЛЖЕН», «СЛЕДУЕТ», «НЕ ДОЛЖЕН», «РЕКОМЕНДУЕТСЯ», «МОЖЕТ» и «ДОПОЛНИТЕЛЬНО» в этом документе: интерпретировать, как описано в [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Цель

Цель этого PSR — позволить разработчикам создавать библиотеки, отделенные от реализаций HTTP-клиентов. Это сделает библиотеки более пригодными для повторного использования, поскольку уменьшит количество зависимостей и снизит вероятность конфликтов версий.

Вторая цель состоит в том, чтобы HTTP-клиенты могли быть заменены в соответствии с
[Принцип подстановки Лисков][Liskov]. Это означает, что все клиенты ДОЛЖНЫ вести себя одинаково при отправке запроса.

## Определения

* **Клиент** — Клиент — это библиотека, которая реализует эту спецификацию для отправки сообщений HTTP-запроса, совместимого с PSR-7, и возврата ответного сообщения HTTP, совместимого с PSR-7, в вызывающую библиотеку.
* **Библиотека вызовов**. Библиотека вызовов — это любой код, использующий Клиент. Он не реализует интерфейсы этой спецификации, но использует объект, который их реализует (Клиент).

## Клиент

Клиент — это объект, реализующий ClientInterface.

Клиент МОЖЕТ:

* Выбирать отправку измененного HTTP-запроса по сравнению с тем, который был предоставлен. Например, он может сжимать тело исходящего сообщения.
* Выбирать изменение полученного HTTP-ответа перед его возвратом в вызывающую библиотеку. Например, он может распаковать тело входящего сообщения.

Если Клиент решает изменить либо HTTP-запрос, либо HTTP-ответ, он ДОЛЖЕН гарантировать, что объект остается внутренне согласованным. Например, если Клиент решает распаковать тело сообщения, он ДОЛЖЕН также удалить заголовок «Content-Encoding» и настроить заголовок «Content-Length».

Обратите внимание, что в результате, поскольку [объекты PSR-7 неизменяемы](https://php-psr.ru/accepted/PSR-7-http-message-meta/#why-value-objects),
Вызывающая библиотека НЕ ДОЛЖНА предполагать, что объект, переданный в `ClientInterface::sendRequest()`, будет тем же самым объектом PHP, который фактически отправлен. Например, объект Request, возвращаемый исключением, МОЖЕТ быть другим объектом, чем тот, который был передан в sendRequest(), поэтому сравнение по ссылке (===) невозможно.

Клиент ДОЛЖЕН:

* Собрать заново многоэтапный ответ HTTP 1xx, чтобы в библиотеку вызовов возвращался действительный ответ HTTP с кодом состояния 200 или выше.

## Обработка ошибок

Клиент НЕ ДОЛЖЕН рассматривать правильно сформированный HTTP-запрос или HTTP-ответ как состояние ошибки. 
Например, коды состояния ответа в диапазоне 400 и 500 НЕ ДОЛЖНЫ вызывать исключение и ДОЛЖНЫ быть возвращены в вызывающую библиотеку как обычно.

Клиент ДОЛЖЕН генерировать экземпляр `Psr\Http\Client\ClientExceptionInterface` тогда и только тогда, когда он вообще не может отправить HTTP-запрос или если ответ HTTP не может быть преобразован в объект ответа PSR-7.

Если запрос не может быть отправлен, потому что сообщение запроса не является правильно сформированным HTTP-запросом или в нем отсутствует какая-либо важная часть информации (например, хост или метод), клиент ДОЛЖЕН выдать экземпляр `Psr\Http\Client\RequestExceptionInterface `.

Если запрос не может быть отправлен из-за какого-либо сбоя сети, включая тайм-аут, Клиент ДОЛЖЕН создать экземпляр `Psr\Http\Client\NetworkExceptionInterface`.

Клиенты МОГУТ генерировать более конкретные исключения, чем определенные здесь (например, TimeOutException или HostNotFoundException), при условии, что они реализуют соответствующий интерфейс, определенный выше.

## Интерфейсы

### ClientInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

interface ClientInterface
{
    /**
     * Отправляет запрос PSR-7 и возвращает ответ PSR-7.
     *
     * @param RequestInterface $request
     *
     * @return ResponseInterface
     *
     * @throws \Psr\Http\Client\ClientExceptionInterface Если при обработке запроса произошла ошибка.
     */
    public function sendRequest(RequestInterface $request): ResponseInterface;
}
```

### ClientExceptionInterface

```php
namespace Psr\Http\Client;

/**
 * Каждое исключение, связанное с HTTP-клиентом, ДОЛЖНО реализовывать этот интерфейс.
 */
interface ClientExceptionInterface extends \Throwable
{
}
```

### RequestExceptionInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * Исключение для случаев, когда запрос не выполнен.
 *
 * Примеры:
 *      - Запрос недействителен (например, отсутствует метод)
 *      - Ошибки запроса во время выполнения (например, поток тела не доступен для поиска)
 */
interface RequestExceptionInterface extends ClientExceptionInterface
{
    /**
     * Возвращает запрос.
     *
     * Объект запроса МОЖЕТ быть объектом, отличным от объекта, переданного в ClientInterface::sendRequest().
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

### NetworkExceptionInterface

```php
namespace Psr\Http\Client;

use Psr\Http\Message\RequestInterface;

/**
 * Бросается, когда запрос не может быть выполнен из-за проблем с сетью.
 *
 * Объект ответа отсутствует, поскольку это исключение генерируется, когда ответ не получен.
 *
 * Пример: имя целевого хоста не может быть разрешено или соединение не удалось.
 */
interface NetworkExceptionInterface extends ClientExceptionInterface
{
    /**
     * Возвращает запрос.
     *
     * Объект запроса МОЖЕТ быть объектом, отличным от объекта, переданного в ClientInterface::sendRequest().
     *
     * @return RequestInterface
     */
    public function getRequest(): RequestInterface;
}
```

[Liskov]: https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B8%D0%BD%D1%86%D0%B8%D0%BF_%D0%BF%D0%BE%D0%B4%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B8_%D0%9B%D0%B8%D1%81%D0%BA%D0%BE%D0%B2
