HTTP-фабрики
==============


В этом документе описан общий стандарт для фабрик, создающих HTTP-объекты, соответствующие [PSR-7][psr7].

В PSR-7 не было рекомендаций по созданию объектов HTTP, что приводит к трудностям при необходимости создания новых объектов HTTP внутри компонентов, не привязанных к конкретной реализации PSR-7.

Интерфейсы, описанные в этом документе, описывают методы, с помощью которых можно создавать экземпляры объектов PSR-7.

Ключевые слова «ДОЛЖЕН», «НЕ ДОЛЖЕН», «ТРЕБУЕТСЯ», «ДОЛЖЕН», «НЕ ДОЛЖЕН», «СЛЕДУЕТ», «НЕ ДОЛЖЕН», «РЕКОМЕНДУЕТСЯ», «МОЖЕТ» и «ДОПОЛНИТЕЛЬНО» в этом документе: интерпретироваться, как описано в [RFC 2119][rfc2119].

[psr7]: /accepted/PSR-7-http-message/
[rfc2119]: https://tools.ietf.org/html/rfc2119

## 1. Спецификация

Фабрика HTTP — это метод, с помощью которого создается новый объект HTTP, определенный PSR-7. Фабрики HTTP ДОЛЖНЫ реализовать эти интерфейсы для каждого типа объекта, предоставляемого пакетом.

## 2. Интерфейсы

Следующие интерфейсы МОГУТ быть реализованы вместе в одном классе или в отдельных классах.

### 2.1 RequestFactoryInterface

Имеет возможность создавать клиентские запросы.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\UriInterface;

interface RequestFactoryInterface
{
    /**
     * Создает новый запрос
     *
     * @param string $method Метод HTTP, связанный с запросом.
     * @param UriInterface|string $uri URI, связанный с запросом.
     */
    public function createRequest(string $method, $uri): RequestInterface;
}
```

### 2.2 ResponseFactoryInterface

Имеет возможность создавать ответы.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ResponseInterface;

interface ResponseFactoryInterface
{
    /**
     * Создает новый ответ.
     *
     * @param int $code Код состояния HTTP. По умолчанию 200.
     * @param string $reasonPhrase Фраза причины, которую необходимо связать
      с кодом состояния в сгенерированном ответе. Если ничего не указано, 
      реализации МОГУТ использовать значения по умолчанию, предложенные 
      в спецификации HTTP.
     */
    public function createResponse(int $code = 200, string $reasonPhrase = ''): ResponseInterface;
}
```

### 2.3 ServerRequestFactoryInterface

Имеет возможность создавать запросы к серверу.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UriInterface;

interface ServerRequestFactoryInterface
{
    /**
     * Создает новый запрос к серверу.
     *
     * Обратите внимание, что параметры сервера принимаются точно такими, 
     какими они заданы — никакой синтаксический анализ/обработка заданных 
     значений не производится. В частности, не предпринимается никаких 
     попыток определить метод HTTP или URI, которые должны быть указаны явно.
     *
     * @param string $method Метод HTTP, связанный с запросом.
     * @param UriInterface|string $uri URI, связанный с запросом.
     * @param array $serverParams Массив параметров API сервера (SAPI), 
     с помощью которых можно заполнить сгенерированный экземпляр запроса.
     */
    public function createServerRequest(string $method, $uri, array $serverParams = []): ServerRequestInterface;
}
```

### 2.4 StreamFactoryInterface

Имеет возможность создавать потоки запросов и ответов.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;

interface StreamFactoryInterface
{
    /**
     * Создает новый поток из строки.
     *
     * Поток СЛЕДУЕТ создавать с использованием временного ресурса.
     *
     * @param string $content Строковое содержимое, которым будет заполнен поток.
     */
    public function createStream(string $content = ''): StreamInterface;

    /**
     * Создает поток из существующего файла.
     *
     * Файл ДОЛЖЕН быть открыт в заданном режиме, 
     который может быть любым режимом, поддерживаемым функцией fopen.
     *
     * `$filename` МОЖЕТ быть любой строкой, поддерживаемой `fopen()`.
     *
     * @param string $filename Имя файла или URI потока, который будет использоваться в качестве основы потока.
     * @param string $mode Режим открытия основного имени файла/потока.
     *
     * @throws \RuntimeException Если файл не открывается.
     * @throws \InvalidArgumentException Если режим недействителен.
     */
    public function createStreamFromFile(string $filename, string $mode = 'r'): StreamInterface;

    /**
     * Создает новый поток из существующего ресурса.
     *
     * Поток ДОЛЖЕН быть доступен для чтения и записи.
     *
     * @param resource $resource Ресурс PHP, который будет использоваться в качестве основы для потока.
     */
    public function createStreamFromResource($resource): StreamInterface;
}
```

Реализациям этого интерфейса СЛЕДУЕТ использовать временный поток при создании ресурсов из строк. 
РЕКОМЕНДУЕМЫЙ способ сделать это:

```php
$resource = fopen('php://temp', 'r+');
```

### 2.5 UploadedFileFactoryInterface

Имеет возможность создавать потоки для загружаемых файлов.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\StreamInterface;
use Psr\Http\Message\UploadedFileInterface;

interface UploadedFileFactoryInterface
{
    /**
     * Создает новый загруженный файл.
     *
     * Если размер не указан, он будет определен путем проверки размера потока.
     *
     * @link http://php.net/manual/features.file-upload.post-method.php
     * @link http://php.net/manual/features.file-upload.errors.php
     *
     * @param StreamInterface $stream Базовый поток, представляющий загруженное содержимое файла.
     * @param int $size Размер файла в байтах.
     * @param int $error Ошибка загрузки файла PHP.
     * @param string $clientFilename Имя файла, предоставленное клиентом, если таковое имеется.
     * @param string $clientMediaType Тип носителя, предоставленный клиентом, если таковой имеется.
     *
     * @throws \InvalidArgumentException Если файловый ресурс не доступен для чтения.
     */
    public function createUploadedFile(
        StreamInterface $stream,
        int $size = null,
        int $error = \UPLOAD_ERR_OK,
        string $clientFilename = null,
        string $clientMediaType = null
    ): UploadedFileInterface;
}
```

### 2.6 UriFactoryInterface

Имеет возможность создавать URI для запросов клиента и сервера.

```php
namespace Psr\Http\Message;

use Psr\Http\Message\UriInterface;

interface UriFactoryInterface
{
    /**
     * Create a new URI.
     *
     * @param string $uri URI для анализа.
     *
     * @throws \InvalidArgumentException Если данный URI не может быть проанализирован.
     */
    public function createUri(string $uri = '') : UriInterface;
}
```
