Обработчики HTTP-запросов на стороне сервера
============================================


Данный документ описывает общие интерфейсы для обработчиков HTTP-запросов на стороне сервера («обработчики запросов») и компонентов промежуточного программного обеспечения HTTP-сервера («middleware»), использующих HTTP-сообщения, описанные в [PSR-7][psr7] или последующих заменяющих PSR.

Обработчики HTTP-запросов являются фундаментальной частью любого веб-приложения. Серверный код получает сообщение-запрос, обрабатывает его и формирует сообщение-ответ. HTTP middleware — это способ вынести общую обработку запросов и ответов за пределы прикладного уровня.

Интерфейсы, описанные в данном документе, являются абстракциями для обработчиков запросов и middleware.

_Примечание: все ссылки на «обработчики запросов» и «middleware» относятся исключительно к обработке **серверных запросов**._

Ключевые слова «MUST», «MUST NOT», «REQUIRED», «SHALL», «SHALL NOT», «SHOULD», «SHOULD NOT», «RECOMMENDED», «MAY» и «OPTIONAL» в данном документе следует интерпретировать в соответствии с [RFC 2119][rfc2119].

[psr7]: https://www.php-fig.org/psr/psr-7/
[rfc2119]: http://tools.ietf.org/html/rfc2119

### Ссылки

- [PSR-7][psr7]
- [RFC 2119][rfc2119]

## 1. Спецификация

### 1.1 Обработчики запросов

Обработчик запросов — это отдельный компонент, который обрабатывает запрос и формирует ответ в соответствии с PSR-7.

Обработчик запросов МОЖЕТ выбрасывать исключение, если условия запроса не позволяют сформировать ответ. Тип исключения не определён.

Обработчики запросов, использующие данный стандарт, ОБЯЗАНЫ реализовывать следующий интерфейс:

- `Psr\Http\Server\RequestHandlerInterface`

### 1.2 Middleware

Компонент middleware — это отдельный компонент, участвующий, как правило совместно с другими компонентами middleware, в обработке входящего запроса и формировании результирующего ответа в соответствии с PSR-7.

Компонент middleware МОЖЕТ создавать и возвращать ответ без делегирования обработчику запросов, если для этого выполнены достаточные условия.

Middleware, использующее данный стандарт, ОБЯЗАНО реализовывать следующий интерфейс:

- `Psr\Http\Server\MiddlewareInterface`

### 1.3 Формирование ответов

РЕКОМЕНДУЕТСЯ, чтобы любой middleware или обработчик запросов, формирующий ответ, содержал либо прототип PSR-7 `ResponseInterface`, либо фабрику, способную создавать экземпляр `ResponseInterface`, во избежание зависимости от конкретной реализации HTTP-сообщений.

### 1.4 Обработка исключений

РЕКОМЕНДУЕТСЯ, чтобы любое приложение, использующее middleware, включало компонент, перехватывающий исключения и преобразующий их в ответы. Данный middleware СЛЕДУЕТ выполнять первым и оборачивать всю дальнейшую обработку, чтобы гарантировать формирование ответа в любом случае.

## 2. Интерфейсы

### 2.1 Psr\Http\Server\RequestHandlerInterface

Следующий интерфейс ОБЯЗАН быть реализован обработчиками запросов.

```php
namespace Psr\Http\Server;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

/**
 * Handles a server request and produces a response.
 *
 * An HTTP request handler process an HTTP request in order to produce an
 * HTTP response.
 */
interface RequestHandlerInterface
{
    /**
     * Handles a request and produces a response.
     *
     * May call other collaborating code to generate the response.
     */
    public function handle(ServerRequestInterface $request): ResponseInterface;
}
```

### 2.2 Psr\Http\Server\MiddlewareInterface

Следующий интерфейс ОБЯЗАН быть реализован совместимыми компонентами middleware.

```php
namespace Psr\Http\Server;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

/**
 * Participant in processing a server request and response.
 *
 * An HTTP middleware component participates in processing an HTTP message:
 * by acting on the request, generating the response, or forwarding the
 * request to a subsequent middleware and possibly acting on its response.
 */
interface MiddlewareInterface
{
    /**
     * Process an incoming server request.
     *
     * Processes an incoming server request in order to produce a response.
     * If unable to produce the response itself, it may delegate to the provided
     * request handler to do so.
     */
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface;
}
```
