# Метадокумент HTTP-сообщений


## 1. Краткое содержание

Целью этого предложения является предоставление набора общих интерфейсов для HTTP-сообщений, как описано в [RFC 7230](http://tools.ietf.org/html/rfc7230) и [RFC 7231](http://tools. ietf.org/html/rfc7231) и URI, как описано в [RFC 3986](http://tools.ietf.org/html/rfc3986) (в контексте HTTP-сообщений).

- RFC 7230: http://www.ietf.org/rfc/rfc7230.txt
- RFC 7231: http://www.ietf.org/rfc/rfc7231.txt
- RFC 3986: http://www.ietf.org/rfc/rfc3986.txt

Все сообщения HTTP состоят из используемой версии протокола HTTP, заголовков и тела сообщения. _Request_ основывается на сообщении и включает метод HTTP, использованный для выполнения запроса, и URI, к которому выполняется запрос. _Ответ_ включает код состояния HTTP и фразу причины.

В PHP сообщения HTTP используются в двух контекстах:

- Чтобы отправить HTTP-запрос через расширение `ext/curl`, собственный уровень потока PHP и т. д. и обработать полученный HTTP-ответ. Другими словами, HTTP-сообщения используются при использовании PHP в качестве _HTTP-клиента_.
- Для обработки входящего HTTP-запроса на сервер и возврата HTTP-ответа клиенту, сделавшему запрос. PHP может использовать HTTP-сообщения при использовании в качестве «серверного приложения» для выполнения HTTP-запросов.

В этом предложении представлен API для полного описания всех частей различных HTTP-сообщений в PHP.

## 2. HTTP-сообщения в PHP

PHP не имеет встроенной поддержки HTTP-сообщений.

### Поддержка HTTP на стороне клиента

PHP поддерживает отправку HTTP-запросов с помощью нескольких механизмов:

- [Потоки](http://php.net/streams)
- [Клиентская библиотека работы с URL](http://php.net/curl)
- [ext/http](http://php.net/http) (v2 также пытается обеспечить поддержку на стороне сервера.)

Потоки PHP — это наиболее удобный и повсеместный способ отправки HTTP-запросов, но они налагают ряд ограничений в отношении правильной настройки поддержки SSL и предоставляют громоздкий интерфейс для настройки таких вещей, как заголовки. cURL предоставляет полный и расширенный набор функций, но, поскольку он не является расширением по умолчанию, часто отсутствует. Расширение http страдает той же проблемой, что и cURL, а также тем фактом, что у него традиционно гораздо меньше примеров использования.

Большинство современных клиентских библиотек HTTP имеют тенденцию абстрагировать реализацию, чтобы гарантировать, что они могут работать в любой среде, в которой они выполняются, и на любом из вышеперечисленных уровней.

### Поддержка HTTP на стороне сервера

PHP использует API-интерфейсы сервера (SAPI) для интерпретации входящих HTTP-запросов, маршалирования входных данных и передачи обработки сценариям. Исходный дизайн SAPI отражал [Common Gateway Interface (http://www.w3.org/CGI/), который маршалировал данные запроса и помещал их в переменные среды перед передачей делегирования сценарию; Затем сценарий извлекает данные из переменных среды, чтобы обработать запрос и вернуть ответ.

Конструкция PHP SAPI абстрагирует общие источники входных данных, такие как файлы cookie, аргументы строки запроса и содержимое POST в кодировке URL-адреса, через суперглобальные переменные ($_COOKIE`, `$_GET` и `$_POST` соответственно), обеспечивая уровень удобства для веб-сайтов. Разработчики.

Что касается ответа на уравнение, PHP изначально разрабатывался как язык шаблонов и позволяет смешивать HTML и PHP; любые части HTML файла немедленно сбрасываются в выходной буфер. Современные приложения и платформы избегают этой практики, поскольку это может привести к проблемам с отправкой строки состояния и/или заголовков ответа; они имеют тенденцию объединять все заголовки и содержимое и выдавать их сразу после завершения всей остальной обработки приложения. Особое внимание необходимо уделять тому, чтобы отчеты об ошибках и другие действия, отправляющие содержимое в выходной буфер, не очищали выходной буфер.

## 3. Зачем?

Сообщения HTTP используются во многих проектах PHP — как на клиентах, так и на серверах. В каждом случае мы наблюдаем одну или несколько из следующих закономерностей или ситуаций:

1. Проекты напрямую используют суперглобальные переменные PHP.
2. Проекты будут создавать реализации с нуля.
3. Для проектов может потребоваться определенная клиент-серверная библиотека HTTP, обеспечивающая реализацию HTTP-сообщений.
4. Проекты могут создавать адаптеры для распространенных реализаций HTTP-сообщений.

В качестве примеров:

1. Практически любое приложение, разработка которого началась до появления фреймворков, включая ряд очень популярных систем CMS, форумов и корзин покупок, исторически использовало суперглобальные переменные.
2. Каждый из таких фреймворков, как Symfony и Zend Framework, определяет компоненты HTTP, которые составляют основу их уровней MVC; даже небольшие специализированные библиотеки, такие как oauth2-server-php, предоставляют и требуют своих собственных реализаций HTTP-запросов/ответов. Guzzle, Buzz и другие реализации HTTP-клиентов также создают свои собственные реализации HTTP-сообщений.
3. Такие проекты, как Silex, Stack и Drupal 8, жестко зависят от HTTP-ядра Symfony. Любой SDK, созданный на Guzzle, имеет жесткие требования к реализации HTTP-сообщений Guzzle.
4. Такие проекты, как Geocoder, создают избыточные [адаптеры для общих библиотек](https://github.com/geocoder-php/Geocoder/tree/6a729c6869f55ad55ae641c74ac9ce7731635e6e/src/Geocoder/HttpAdapter).

Прямое использование суперглобальных переменных вызывает ряд проблем. Во-первых, они изменяемы, что позволяет библиотекам и коду изменять значения и, таким образом, изменять состояние приложения. Кроме того, суперглобальные переменные усложняют и делают интеграционное тестирование сложным и хрупким, что приводит к ухудшению качества кода.

В нынешней экосистеме фреймворков, реализующих абстракции HTTP-сообщений, конечным результатом является то, что проекты не способны к взаимодействию или перекрестному опылению. Чтобы использовать код, предназначенный для одной платформы, из другой, первым делом необходимо построить мостовой уровень между реализациями HTTP-сообщений. На стороне клиента, если в конкретной библиотеке нет адаптера, который вы могли бы использовать, вам необходимо соединить пары запрос/ответ, если вы хотите использовать адаптер из другой библиотеки.

Наконец, когда дело доходит до ответов на стороне сервера, PHP действует по-своему: любой контент, созданный до вызова `header()`, приведет к тому, что этот вызов станет неактивным; в зависимости от настроек отчетов об ошибках это часто может означать, что заголовки и/или статус ответа отправляются неправильно. Один из способов обойти эту проблему — использовать функции буферизации вывода PHP, но вложение выходных буферов может стать проблематичным и затруднить отладку. Таким образом, фреймворки и приложения имеют тенденцию создавать абстракции ответов для агрегирования заголовков и контента, которые могут быть отправлены одновременно, и эти абстракции часто несовместимы.

Таким образом, цель этого предложения — абстрагировать интерфейсы запросов и ответов как на стороне клиента, так и на стороне сервера, чтобы обеспечить совместимость между проектами. Если проекты реализуют эти интерфейсы, можно предположить разумный уровень совместимости при использовании кода из разных библиотек.

Следует отметить, что целью этого предложения не является устаревание текущих интерфейсов, используемых существующими библиотеками PHP. Это предложение направлено на обеспечение совместимости между пакетами PHP с целью описания HTTP-сообщений.

## 4. Суть

### 4.1 Цели

* Предоставить интерфейсы, необходимые для описания HTTP-сообщений.
* Сосредоточьтесь на практическом применении и удобстве использования.
* Определите интерфейсы для моделирования всех элементов HTTP-сообщения и спецификаций URI.
* Убедитесь, что API не накладывает произвольные ограничения на HTTP-сообщения. Например, некоторые тела HTTP-сообщений могут быть слишком большими для хранения в памяти, поэтому мы должны это учитывать.
* Предоставляйте полезные абстракции как для обработки входящих запросов к серверным приложениям, так и для отправки исходящих запросов в HTTP-клиентах.

### 4.2 Не является целью

* Это предложение не предполагает использования всех клиентских HTTP-библиотек или серверных библиотек.
  фреймворки, чтобы изменить их интерфейсы в соответствии с ними. Это строго предназначено для совместимости.
* Хотя восприятие того, что является деталями реализации, а что нет, у каждого разное, это предложение не должно навязывать детали реализации. Поскольку RFC 7230, 7231 и 3986 не требуют какой-либо конкретной реализации, потребуется определенное количество изобретений для описания интерфейсов сообщений HTTP в PHP.

## 5. Дизайнерские решения

### Дизайн сообщения

MessageInterface предоставляет средства доступа к элементам, общим для всех HTTP-сообщений, независимо от того, предназначены ли они для запросов или ответов. Эти элементы включают в себя:

- Версия протокола HTTP (например, «1.0», «1.1»)
- HTTP-заголовки
- Тело HTTP-сообщения

Для описания запросов и ответов используются более конкретные интерфейсы, а точнее контекст каждого из них (клиентская или серверная сторона). Эти подразделения частично вдохновлены существующим использованием PHP, а также другими языками, такими как [Rack] Ruby (https://rack.github.io),
Python [WSGI](https://www.python.org/dev/peps/pep-0333/),
[http-пакет] Go(http://golang.org/pkg/net/http/),
[http-модуль] узла (http://nodejs.org/api/http.html) и т. д.

### Почему в сообщениях есть методы заголовков, а не в пакете заголовков?

Само сообщение является контейнером для заголовков (а также других свойств сообщения). То, как они представлены внутри, является деталью реализации, но единообразный доступ к заголовкам является обязанностью сообщения.

### Почему URI представлены как объекты?

URI — это значения, идентичность которых определяется значением, и поэтому их следует моделировать как объекты значений.

Кроме того, URI содержат множество сегментов, к которым можно обращаться много раз в одном запросе, и для определения которых потребуется анализ URI (например, с помощью `parse_url()`). Моделирование URI как объектов значений позволяет выполнять синтаксический анализ только один раз и упрощает доступ к отдельным сегментам. Это также обеспечивает удобство в клиентских приложениях, позволяя пользователям создавать новые экземпляры базового экземпляра URI только с изменяющимися сегментами (например, обновляя только путь).

### Почему в интерфейсе запроса есть методы для работы с целью запроса И создания URI?

В RFC 7230 строка запроса подробно описана как содержащая «цель запроса». Из четырех форм запроса-цели только одна является URI, соответствующей RFC 3986; наиболее распространенной формой является origin-form, которая представляет URI без схемы или информации о полномочиях. Более того, поскольку все формы действительны для запросов, предложение должно учитывать каждую из них.

Таким образом, `RequestInterface` имеет методы, относящиеся к цели запроса. По умолчанию он будет использовать составной URI для представления цели запроса в исходной форме и, при отсутствии экземпляра URI, возвращать строку «/». Другой метод, withRequestTarget(), позволяет указать экземпляр с определенной целью запроса, позволяя пользователям создавать запросы, использующие одну из других допустимых форм цели запроса.

URI сохраняется как отдельный элемент запроса по ряду причин. Как для клиентов, так и для серверов обычно требуется знание абсолютного URI. В случае клиентов URI и, в частности, сведения о схеме и полномочиях необходимы для фактического TCP-соединения. Для серверных приложений полный URI часто требуется для проверки запроса или маршрутизации к соответствующему обработчику.

### Почему Объекты Значения?

В предложении сообщения и URI моделируются как [объекты значений](http://en.wikipedia.org/wiki/Value_object).

Сообщения — это значения, где идентификатор — это совокупность всех частей сообщения; изменение любого аспекта сообщения по сути является новым сообщением.
Это само определение объекта значения. Практика, при которой изменения приводят к созданию нового экземпляра, называется [неизменяемостью](https://ru.wikipedia.org/wiki/%D0%9D%D0%B5%D0%B8%D0%B7%D0%BC%D0%B5%D0%BD%D1%8F%D0%B5%D0%BC%D1%8B%D0%B9_%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82) и представляет собой функцию, предназначенную для обеспечения целостности данного значения.

В предложении также признается, что большинству клиентов и серверных приложений потребуется возможность легко обновлять аспекты сообщений, и поэтому предоставляются методы интерфейса, которые будут создавать новые экземпляры сообщений с обновлениями. Обычно к ним добавляются префиксы «с» или «без».

Объекты-значения предоставляют несколько преимуществ при моделировании HTTP-сообщений:

- Изменения в состоянии URI не могут изменить запрос, составляющий экземпляр URI.
- Изменения в заголовках не могут изменить составляющее их сообщение.

По сути, моделирование HTTP-сообщений как объектов значений обеспечивает целостность состояния сообщения и предотвращает необходимость двунаправленных зависимостей, которые часто могут выйти из синхронизации или привести к проблемам отладки или производительности.

Для HTTP-клиентов они позволяют потребителям создавать базовый запрос с такими данными, как базовый URI и необходимые заголовки, без необходимости создавать новый запрос или сбрасывать состояние запроса для каждого сообщения, отправляемого клиентом:

~~~php
$uri = new Uri('http://api.example.com');
$baseRequest = new Request($uri, null, [
    'Authorization' => 'Bearer ' . $token,
    'Accept'        => 'application/json',
]);

$request = $baseRequest->withUri($uri->withPath('/user'))->withMethod('GET');
$response = $client->send($request);

// получаем идентификатор пользователя из $response

$body = new StringStream(json_encode(['tasks' => [
    'Code',
    'Coffee',
]]));
$request = $baseRequest
    ->withUri($uri->withPath('/tasks/user/' . $userId))
    ->withMethod('POST')
    ->withHeader('Content-Type', 'application/json')
    ->withBody($body);
$response = $client->send($request)

// Не нужно перезаписывать заголовки или тело!
$request = $baseRequest->withUri($uri->withPath('/tasks'))->withMethod('GET');
$response = $client->send($request);
~~~

На стороне сервера разработчикам необходимо:

- Десериализовать тело сообщения запроса.
- Расшифровать файлы cookie HTTP.
- Напишите в ответ.

Эти операции также можно выполнять с объектами-значениями, что дает ряд преимуществ:

- Исходное состояние запроса может быть сохранено для извлечения любым потребителем.
- Состояние ответа по умолчанию может быть создано с использованием заголовков и/или тела сообщения по умолчанию.

Сегодня большинство популярных PHP-фреймворков имеют полностью изменяемые HTTP-сообщения. Основные изменения, необходимые для использования объектов истинной ценности:

- Вместо вызова методов установки или установки общедоступных свойств, мутатор
  Будут вызываться методы и присваиваться результат.
- Разработчики должны уведомить приложение об изменении состояния.

Например, в Zend Framework 2 вместо следующего:

~~~php
function (MvcEvent $e)
{
    $response = $e->getResponse();
    $response->setHeaderLine('x-foo', 'bar');
}
~~~

сейчас бы написали:

~~~php
function (MvcEvent $e)
{
    $response = $e->getResponse();
    $e->setResponse(
        $response->withHeader('x-foo', 'bar')
    );
}
~~~

Вышеупомянутое объединяет назначение и уведомление в одном вызове.

Побочным преимуществом этой практики является возможность явного внесения любых изменений в состояние приложения.

### Новые экземпляры vs возврат $this

Одно наблюдение, сделанное в отношении различных методов `with*()`, заключается в том, что они, вероятно, могут безопасно `вернуть $this;`, если представленный аргумент не приведет к изменению значения. Одним из оснований для этого является производительность (поскольку это не приведет к операции клонирования).

Различные интерфейсы были написаны с многословием, указывающим, что неизменность ДОЛЖНА быть сохранена, но указывают только на то, что должен быть возвращен «экземпляр», содержащий новое состояние. Поскольку экземпляры, представляющие одно и то же значение, считаются равными, возврат $this функционально эквивалентен и, следовательно, разрешен.

### Использование потоков вместо X

`MessageInterface` uses a body value that must implement `StreamInterface`. This
design decision was made so that developers can send and receive (and/or receive
and send) HTTP messages that contain more data than can practically be stored in
memory while still allowing the convenience of interacting with message bodies
as a string. While PHP provides a stream abstraction by way of stream wrappers,
stream resources can be cumbersome to work with: stream resources can only be
cast to a string using `stream_get_contents()` or manually reading the remainder
of a string. Adding custom behavior to a stream as it is consumed or populated
requires registering a stream filter; however, stream filters can only be added
to a stream after the filter is registered with PHP (i.e., there is no stream
filter autoloading mechanism).

The use of a well- defined stream interface allows for the potential of
flexible stream decorators that can be added to a request or response
pre-flight to enable things like encryption, compression, ensuring that the
number of bytes downloaded reflects the number of bytes reported in the
`Content-Length` of a response, etc. Decorating streams is a well-established
[pattern in the Java](http://docs.oracle.com/javase/7/docs/api/java/io/package-tree.html)
and [Node](http://nodejs.org/api/stream.html#stream_class_stream_transform_1)
communities that allows for very flexible streams.

The majority of the `StreamInterface` API is based on
[Python's io module](http://docs.python.org/3.1/library/io.html), which provides
a practical and consumable API. Instead of implementing stream
capabilities using something like a `WritableStreamInterface` and
`ReadableStreamInterface`, the capabilities of a stream are provided by methods
like `isReadable()`, `isWritable()`, etc. This approach is used by Python,
[C#, C++](http://msdn.microsoft.com/en-us/library/system.io.stream.aspx),
[Ruby](http://www.ruby-doc.org/core-2.0.0/IO.html),
[Node](http://nodejs.org/api/stream.html), and likely others.

#### What if I just want to return a file?

In some cases, you may want to return a file from the filesystem. The typical
way to do this in PHP is one of the following:

~~~php
readfile($filename);

stream_copy_to_stream(fopen($filename, 'r'), fopen('php://output', 'w'));
~~~

Note that the above omits sending appropriate `Content-Type` and
`Content-Length` headers; the developer would need to emit these prior to
calling the above code.

The equivalent using HTTP messages would be to use a `StreamInterface`
implementation that accepts a filename and/or stream resource, and to provide
this to the response instance. A complete example, including setting appropriate
headers:

~~~php
// where Stream is a concrete StreamInterface:
$stream   = new Stream($filename);
$finfo    = new finfo(FILEINFO_MIME);
$response = $response
    ->withHeader('Content-Type', $finfo->file($filename))
    ->withHeader('Content-Length', (string) filesize($filename))
    ->withBody($stream);
~~~

Emitting this response will send the file to the client.

#### What if I want to directly emit output?

Directly emitting output (e.g. via `echo`, `printf`, or writing to the
`php://output` stream) is generally only advisable as a performance optimization
or when emitting large data sets. If it needs to be done and you still wish
to work in an HTTP message paradigm, one approach would be to use a
callback-based `StreamInterface` implementation, per [this
example](https://github.com/phly/psr7examples#direct-output). Wrap any code
emitting output directly in a callback, pass that to an appropriate
`StreamInterface` implementation, and provide it to the message body:

~~~php
$output = new CallbackStream(function () use ($request) {
    printf("The requested URI was: %s<br>\n", $request->getUri());
    return '';
});
return (new Response())
    ->withHeader('Content-Type', 'text/html')
    ->withBody($output);
~~~

#### What if I want to use an iterator for content?

Ruby's Rack implementation uses an iterator-based approach for server-side
response message bodies. This can be emulated using an HTTP message paradigm via
an iterator-backed `StreamInterface` approach, as [detailed in the
psr7examples repository](https://github.com/phly/psr7examples#iterators-and-generators).

### Why are streams mutable?

The `StreamInterface` API includes methods such as `write()` which can
change the message content -- which directly contradicts having immutable
messages.

The problem that arises is due to the fact that the interface is intended to
wrap a PHP stream or similar. A write operation therefore will proxy to writing
to the stream. Even if we made `StreamInterface` immutable, once the stream
has been updated, any instance that wraps that stream will also be updated --
making immutability impossible to enforce.

Our recommendation is that implementations use read-only streams for
server-side requests and client-side responses.

### Rationale for ServerRequestInterface

The `RequestInterface` and `ResponseInterface` have essentially 1:1
correlations with the request and response messages described in
[RFC 7230](http://www.ietf.org/rfc/rfc7230.txt). They provide interfaces for
implementing value objects that correspond to the specific HTTP message types
they model.

For server-side applications there are other considerations for
incoming requests:

- Access to server parameters (potentially derived from the request, but also
  potentially the result of server configuration, and generally represented
  via the `$_SERVER` superglobal; these are part of the PHP Server API (SAPI)).
- Access to the query string arguments (usually encapsulated in PHP via the
  `$_GET` superglobal).
- Access to the parsed body (i.e., data deserialized from the incoming request
  body; in PHP, this is typically the result of POST requests using
  `application/x-www-form-urlencoded` content types, and encapsulated in the
  `$_POST` superglobal, but for non-POST, non-form-encoded data, could be
  an array or an object).
- Access to uploaded files (encapsulated in PHP via the `$_FILES` superglobal).
- Access to cookie values (encapsulated in PHP via the `$_COOKIE` superglobal).
- Access to attributes derived from the request (usually, but not limited to,
  those matched against the URL path).

Uniform access to these parameters increases the viability of interoperability
between frameworks and libraries, as they can now assume that if a request
implements `ServerRequestInterface`, they can get at these values. It also
solves problems within the PHP language itself:

- Until 5.6.0, `php://input` was read-once; as such, instantiating multiple
  request instances from multiple frameworks/libraries could lead to
  inconsistent state, as the first to access `php://input` would be the only
  one to receive the data.
- Unit testing against superglobals (e.g., `$_GET`, `$_FILES`, etc.) is
  difficult and typically brittle. Encapsulating them inside the
  `ServerRequestInterface` implementation eases testing considerations.

### Why "parsed body" in the ServerRequestInterface?

Arguments were made to use the terminology "BodyParams", and require the value
to be an array, with the following rationale:

- Consistency with other server-side parameter access.
- `$_POST` is an array, and the 80% use case would target that superglobal.
- A single type makes for a strong contract, simplifying usage.

The main argument is that if the body parameters are an array, developers have
predictable access to values:

~~~php
$foo = isset($request->getBodyParams()['foo'])
    ? $request->getBodyParams()['foo']
    : null;
~~~

The argument for using "parsed body" was made by examining the domain. A message
body can contain literally anything. While traditional web applications use
forms and submit data using POST, this is a use case that is quickly being
challenged in current web development trends, which are often API-centric, and
thus use alternate request methods (notably PUT and PATCH), as well as
non-form-encoded content (generally JSON or XML) that _can_ be coerced to arrays
in many cases, but in many cases also _cannot_ or _should not_.

If forcing the property representing the parsed body to be only an array,
developers then need a shared convention about where to put the results of
parsing the body. These might include:

- A special key under the body parameters, such as `__parsed__`.
- A specially named attribute, such as `__body__`.

The end result is that a developer now has to look in multiple locations:

~~~php
$data = $request->getBodyParams();
if (isset($data['__parsed__']) && is_object($data['__parsed__'])) {
    $data = $data['__parsed__'];
}

// or:
$data = $request->getBodyParams();
if ($request->hasAttribute('__body__')) {
    $data = $request->getAttribute('__body__');
}
~~~

The solution presented is to use the terminology "ParsedBody", which implies
that the values are the results of parsing the message body. This also means
that the return value _will_ be ambiguous; however, because this is an attribute
of the domain, this is also expected. As such, usage will become:

~~~php
$data = $request->getParsedBody();
if (! $data instanceof \stdClass) {
    // raise an exception!
}
// otherwise, we have what we expected
~~~

This approach removes the limitations of forcing an array, at the expense of
ambiguity of return value. Considering that the other suggested solutions —
pushing the parsed data into a special body parameter key or into an attribute —
also suffer from ambiguity, the proposed solution is simpler as it does not
require additions to the interface specification. Ultimately, the ambiguity
enables the flexibility required when representing the results of parsing the
body.

### Why is no functionality included for retrieving the "base path"?

Many frameworks provide the ability to get the "base path," usually considered
the path up to and including the front controller. As an example, if the
application is served at `http://example.com/b2b/index.php`, and the current URI
used to request it is `http://example.com/b2b/index.php/customer/register`, the
functionality to retrieve the base path would return `/b2b/index.php`. This value
can then be used by routers to strip that path segment prior to attempting a
match.

This value is often also then used for URI generation within applications;
parameters will be passed to the router, which will generate the path, and
prefix it with the base path in order to return a fully-qualified URI. Other
tools — typically view helpers, template filters, or template functions — are
used to resolve a path relative to the base path in order to generate a URI for
linking to resources such as static assets.

On examination of several different implementations, we noticed the following:

- The logic for determining the base path varies widely between implementations.
  As an example, compare the [logic in ZF2](https://github.com/zendframework/zf2/blob/release-2.3.7/library/Zend/Http/PhpEnvironment/Request.php#L477-L575)
  to the [logic in Symfony 2](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/HttpFoundation/Request.php#L1858-L1877).
- Most implementations appear to allow manual injection of a base path to the
  router and/or any facilities used for URI generation.
- The primary use cases — routing and URI generation — typically are the only
  consumers of the functionality; developers usually do not need to be aware
  of the base path concept as other objects take care of that detail for them.
  As examples:
  - A router will strip off the base path for you during routing; you do not
    need to pass the modified path to the router.
  - View helpers, template filters, etc. typically are injected with a base path
    prior to invocation. Sometimes this is manually done, though more often it
    is the result of framework wiring.
- All sources necessary for calculating the base path *are already in the
  `RequestInterface` instance*, via server parameters and the URI instance.

Our stance is that base path detection is framework and/or application
specific, and the results of detection can be easily injected into objects that
need it, and/or calculated as needed using utility functions and/or classes from
the `RequestInterface` instance itself.

### Why does getUploadedFiles() return objects instead of arrays?

`getUploadedFiles()` returns a tree of `Psr\Http\Message\UploadedFileInterface`
instances. This is done primarily to simplify specification: instead of
requiring paragraphs of implementation specification for an array, we specify an
interface.

Additionally, the data in an `UploadedFileInterface` is normalized to work in
both SAPI and non-SAPI environments. This allows the creation of processes to parse
the message body manually and assign contents to streams without first writing
to the filesystem, while still allowing proper handling of file uploads in SAPI
environments.

### What about "special" header values?

A number of header values contain unique representation requirements which can
pose problems both for consumption as well as generation; in particular, cookies
and the `Accept` header.

This proposal does not provide any special treatment of any header types. The
base `MessageInterface` provides methods for header retrieval and setting, and
all header values are, in the end, string values.

Developers are encouraged to write commodity libraries for interacting with
these header values, either for the purposes of parsing or generation. Users may
then consume these libraries when needing to interact with those values.
Examples of this practice already exist in libraries such as
[willdurand/Negotiation](https://github.com/willdurand/Negotiation) and
[Aura.Accept](https://github.com/auraphp/Aura.Accept). So long as the object
has functionality for casting the value to a string, these objects can be
used to populate the headers of an HTTP message.

## 6. People

### 6.1 Editor(s)

* Matthew Weier O'Phinney

### 6.2 Sponsors

* Paul M. Jones
* Beau Simensen (coordinator)

### 6.3 Contributors

* Michael Dowling
* Larry Garfield
* Evert Pot
* Tobias Schultze
* Bernhard Schussek
* Anton Serdyuk
* Phil Sturgeon
* Chris Wilkinson
