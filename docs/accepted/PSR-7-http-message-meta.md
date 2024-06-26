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

// Не нужно перезаписывать заголовки или тело запроса!
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

`MessageInterface` использует значение тела, которое должно реализовывать `StreamInterface`. Это проектное решение было принято для того, чтобы разработчики могли отправлять и получать (и/или получать и отправлять) HTTP-сообщения, которые содержат больше данных, чем практически можно хранить в памяти, при этом обеспечивая удобство взаимодействия с телами сообщений в виде строки. Хотя PHP обеспечивает абстракцию потока посредством оболочек потока, работать с потоковыми ресурсами может быть неудобно: потоковые ресурсы можно преобразовать в строку только с помощью `stream_get_contents()` или вручную прочитать оставшуюся часть строки. Добавление пользовательского поведения к потоку по мере его потребления или заполнения требует регистрации фильтра потока; однако фильтры потока можно добавлять в поток только после регистрации фильтра в PHP (т. е. механизм автозагрузки фильтров потока отсутствует).

Использование четко определенного интерфейса потока позволяет использовать гибкие декораторы потока, которые можно добавлять к запросу или ответу перед отправкой, чтобы обеспечить такие функции, как шифрование и сжатие, гарантируя, что количество загруженных байтов отражает количество переданных байтов. в «Content-Length» ответа и т. д. Декорирование потоков — это устоявшаяся практика в [Java](http://docs.oracle.com/javase/7/docs/api/java/io/package-tree.html)
и [Node](http://nodejs.org/api/stream.html#stream_class_stream_transform_1), что позволяет создавать очень гибкие потоки.

Большая часть API StreamInterface основана на
[Модуль io Python](http://docs.python.org/3.1/library/io.html), который предоставляет практичный и удобный API. Вместо реализации возможностей потока с использованием чего-то вроде `WritableStreamInterface` и `ReadableStreamInterface`, возможности потока предоставляются такими методами, как `isReadable()`, `isWritable()` и т. д. Этот подход используется Python,
[C#, C++](http://msdn.microsoft.com/en-us/library/system.io.stream.aspx), [Ruby](http://www.ruby-doc.org/core-2.0 .0/IO.html), [Node](http://nodejs.org/api/stream.html) и другими языками.

#### Что делать, если я просто хочу вернуть файл?

В некоторых случаях вам может потребоваться вернуть файл из файловой системы. Типичный способ сделать это в PHP — один из следующих:

~~~php
readfile($filename);

stream_copy_to_stream(fopen($filename, 'r'), fopen('php://output', 'w'));
~~~

Обратите внимание, что вышеизложенное не отправляет соответствующие заголовки Content-Type и Content-Length; разработчику необходимо будет создать их перед вызовом приведенного выше кода.

Эквивалентом использования HTTP-сообщений было бы использование реализации StreamInterface, которая принимает имя файла и/или ресурс потока и передает это экземпляру ответа. Полный пример, включая установку соответствующих заголовков:

~~~php
// where Stream is a concrete StreamInterface:
$stream   = new Stream($filename);
$finfo    = new finfo(FILEINFO_MIME);
$response = $response
    ->withHeader('Content-Type', $finfo->file($filename))
    ->withHeader('Content-Length', (string) filesize($filename))
    ->withBody($stream);
~~~

При отправке этого ответа файл будет отправлен клиенту.

#### Что, если я хочу напрямую выдавать выходные данные?

Непосредственная передача вывода (например, через `echo`, `printf` или запись в поток `php://output`) обычно рекомендуется только в целях оптимизации производительности или при отправке больших наборов данных. Если это необходимо сделать, и вы все еще хотите работать в парадигме HTTP-сообщений, одним из подходов может быть использование реализации StreamInterface на основе обратного вызова, согласно [этому примеру] (https://github.com/phly/psr7examples#direct-output). Оберните любой код, выдающий выходные данные, непосредственно в обратный вызов, передайте его соответствующей реализации StreamInterface и предоставьте его в тело сообщения:

~~~php
$output = new CallbackStream(function () use ($request) {
    printf("The requested URI was: %s<br>\n", $request->getUri());
    return '';
});
return (new Response())
    ->withHeader('Content-Type', 'text/html')
    ->withBody($output);
~~~

#### Что делать, если я хочу использовать итератор для контента?

Реализация Ruby Rack использует подход на основе итератора для тела ответного сообщения на стороне сервера. Это можно эмулировать с использованием парадигмы HTTP-сообщений с помощью подхода StreamInterface на основе итератора, как [подробно описано в репозитории psr7examples](https://github.com/phly/psr7examples#iterators-and-generators).

### Почему потоки изменяемы?

API `StreamInterface` включает в себя такие методы, как `write()`, которые могут изменять содержимое сообщения, что прямо противоречит неизменяемым сообщениям.

Возникающая проблема связана с тем, что интерфейс предназначен для переноса потока PHP или чего-то подобного. Таким образом, операция записи будет проксироваться для записи в поток. Даже если мы сделаем `StreamInterface` неизменяемым, после обновления потока любой экземпляр, который обертывает этот поток, также будет обновлен, что делает невозможным принудительное соблюдение неизменяемости.

Наша рекомендация заключается в том, чтобы реализации использовали потоки только для чтения для запросов на стороне сервера и ответов на стороне клиента.

### Обоснование интерфейса ServerRequestInterface

RequestInterface и ResponseInterface имеют по существу корреляцию 1:1 с сообщениями запроса и ответа, описанными в [RFC 7230](http://www.ietf.org/rfc/rfc7230.txt). Они предоставляют интерфейсы для реализации объектов значений, соответствующих конкретным типам HTTP-сообщений, которые они моделируют.

Для серверных приложений существуют и другие аспекты входящих запросов:

- Доступ к параметрам сервера (потенциально получаемый из запроса, но также потенциально являющийся результатом конфигурации сервера и обычно представленный через суперглобальный объект `$_SERVER`; они являются частью PHP Server API (SAPI)).
- Доступ к аргументам строки запроса (обычно инкапсулируемым в PHP через суперглобальный объект $_GET).
- Доступ к проанализированному телу (т. е. данным, десериализованным из тела входящего запроса; в PHP это обычно результат POST-запросов с использованием
  типы контента `application/x-www-form-urlencoded` и инкапсулированы в суперглобальный объект `$_POST`, но для данных, отличных от POST, и данных, не закодированных в форме, это может быть массив или объект).
- Доступ к загруженным файлам (инкапсулированный в PHP через суперглобальный объект $_FILES).
- Доступ к значениям файлов cookie (инкапсулированным в PHP через суперглобальный объект $_COOKIE).
- Доступ к атрибутам, полученным из запроса (обычно, помимо прочего, к атрибутам, сопоставленным с URL-путем).

Унифицированный доступ к этим параметрам повышает жизнеспособность взаимодействия между платформами и библиотеками, поскольку теперь они могут предполагать, что если запрос реализует ServerRequestInterface, они могут получить эти значения. Он также решает проблемы внутри самого языка PHP:

- До версии 5.6.0 `php://input` читался один раз; Таким образом, создание нескольких экземпляров запроса из нескольких платформ/библиотек может привести к несогласованному состоянию, поскольку первый, кто получит доступ к `php://input`, будет единственным, кто получит данные.
- Модульное тестирование суперглобальных переменных (например, `$_GET`, `$_FILES` и т. д.) сложно и обычно нестабильно. Инкапсуляция их внутри реализации ServerRequestInterface упрощает тестирование.

### Почему «разобранное тело» в ServerRequestInterface?

Были выдвинуты аргументы для использования терминологии «BodyParams» и требования, чтобы значение было массивом, со следующим обоснованием:

- Согласованность с доступом к другим параметрам на стороне сервера.
- `$_POST` — это массив, и вариант использования 80% будет нацелен на этот суперглобальный массив.
- Один тип обеспечивает надежный контракт, упрощая использование.

Основной аргумент заключается в том, что если параметры тела представляют собой массив, разработчики имеют предсказуемый доступ к значениям:

~~~php
$foo = isset($request->getBodyParams()['foo'])
    ? $request->getBodyParams()['foo']
    : null;
~~~

Аргумент в пользу использования «разобранного тела» был выдвинут при исследовании домена. Тело сообщения может содержать буквально что угодно. В то время как традиционные веб-приложения используют формы и отправляют данные с помощью POST, этот вариант использования быстро бросает вызов современным тенденциям веб-разработки, которые часто ориентированы на API и, следовательно, используют альтернативные методы запроса (в частности, PUT и PATCH), а также как контент, не закодированный в форме (обычно JSON или XML), который во многих случаях _может_ быть преобразован в массивы, но во многих случаях также _не_ может_ или _не должен_.

Если заставить свойство, представляющее разобранное тело, быть только массивом, разработчикам потребуется общее соглашение о том, куда поместить результаты синтаксического анализа тела. Они могут включать в себя:

- Специальный ключ под параметрами тела, например `__parsed__`.
- Атрибут со специальным именем, например `__body__`.

Конечным результатом является то, что разработчику теперь приходится искать в нескольких местах:

~~~php
$data = $request->getBodyParams();
if (isset($data['__parsed__']) && is_object($data['__parsed__'])) {
    $data = $data['__parsed__'];
}

// или:
$data = $request->getBodyParams();
if ($request->hasAttribute('__body__')) {
    $data = $request->getAttribute('__body__');
}
~~~

Представленное решение заключается в использовании терминологии «ParsedBody», которая подразумевает, что значения являются результатами анализа тела сообщения. Это также означает, что возвращаемое значение _будет_ неоднозначным; однако, поскольку это атрибут домена, это также ожидается. Таким образом, использование станет:

~~~php
$data = $request->getParsedBody();
if (! $data instanceof \stdClass) {
    // Бросаем исключение!
}
// в остальном мы имеем то, что ожидали
~~~

Этот подход устраняет ограничения принудительного использования массива за счет неоднозначности возвращаемого значения. Учитывая, что другие предлагаемые решения — помещение разобранных данных в специальный ключ параметра тела или в атрибут — также страдают неоднозначностью, предлагаемое решение проще, поскольку не требует дополнений к спецификации интерфейса. В конечном счете, неоднозначность обеспечивает гибкость, необходимую при представлении результатов анализа тела.

### Почему не включена функция получения «базового пути»?

Многие фреймворки предоставляют возможность получить «базовый путь», который обычно считается путем до фронт-контроллера включительно. Например, если приложение обслуживается по адресу http://example.com/b2b/index.php, а текущий URI, используемый для его запроса, — http://example.com/b2b/index.php/customer/register, функция получения базового пути вернет `/b2b/index.php`. Затем это значение может использоваться маршрутизаторами для удаления этого сегмента пути перед попыткой сопоставления.

Это значение часто затем используется для генерации URI внутри приложений; параметры будут переданы маршрутизатору, который сгенерирует путь и укажет ему префикс базового пути, чтобы вернуть полный URI. Другие инструменты — обычно помощники представлений, фильтры шаблонов или функции шаблонов — используются для разрешения пути относительно базового пути, чтобы сгенерировать URI для ссылки на ресурсы, такие как статические ресурсы.

При рассмотрении нескольких различных реализаций мы заметили следующее:

- Логика определения базового пути сильно различается в зависимости от реализации. В качестве примера сравните [логику в ZF2](https://github.com/zendframework/zf2/blob/release-2.3.7/library/Zend/Http/PhpEnvironment/Request.php#L477-L575)
  к [логике в Symfony 2](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/HttpFoundation/Request.php#L1858-L1877).
- Большинство реализаций, по-видимому, допускают ручное введение базового пути к маршрутизатору и/или любым средствам, используемым для генерации URI.
- Основные варианты использования — маршрутизация и генерация URI — обычно являются единственными потребителями функциональности; разработчикам обычно не нужно знать концепцию базового пути, поскольку другие объекты позаботятся об этой детали за них.
  В качестве примеров:
  - Маршрутизатор удалит вам базовый путь во время маршрутизации; вам не нужно передавать измененный путь маршрутизатору.
  - Помощники представлений, фильтры шаблонов и т. д. обычно вводятся с базовым путем перед вызовом. Иногда это делается вручную, но чаще всего это результат работы фреймворка.
- Все источники, необходимые для расчета базового пути, *уже находятся в экземпляре RequestInterface* через параметры сервера и экземпляр URI.

Наша позиция заключается в том, что обнаружение базового пути зависит от платформы и/или приложения, и результаты обнаружения могут быть легко внедрены в объекты, которые в этом нуждаются, и/или рассчитаны по мере необходимости с использованием служебных функций и/или классов из самого экземпляра RequestInterface. .

### Почему getUploadedFiles() возвращает объекты, а не массивы?

`getUploadedFiles()` возвращает дерево экземпляров `Psr\Http\Message\UploadedFileInterface`. Это сделано в первую очередь для упрощения спецификации: вместо того, чтобы требовать параграфы спецификации реализации для массива, мы указываем интерфейс.

Кроме того, данные в UploadedFileInterface нормализуются для работы как в средах SAPI, так и в средах, отличных от SAPI. Это позволяет создавать процессы для анализа тела сообщения вручную и назначения содержимого потокам без предварительной записи в файловую систему, при этом обеспечивая правильную обработку загрузки файлов в средах SAPI.

### А как насчет «специальных» значений заголовков?

Ряд значений заголовков содержат уникальные требования к представлению, которые могут создавать проблемы как для потребления, так и для генерации; в частности, файлы cookie и заголовок Accept.

Это предложение не предусматривает какой-либо специальной обработки каких-либо типов заголовков. Базовый интерфейс MessageInterface предоставляет методы для получения и установки заголовков, и все значения заголовков в конечном итоге являются строковыми значениями.

Разработчикам рекомендуется писать стандартные библиотеки для взаимодействия с этими значениями заголовков либо для целей анализа, либо для генерации. Затем пользователи могут использовать эти библиотеки, когда им необходимо взаимодействовать с этими значениями. Примеры такой практики уже существуют в таких библиотеках, как [willdurand/Negotiation](https://github.com/willdurand/Negotiation) и
[Aura.Accept](https://github.com/auraphp/Aura.Accept). Пока объект имеет функцию преобразования значения в строку, эти объекты можно использовать для заполнения заголовков HTTP-сообщения.

## 6. Люди

### 6.1 Редактор(ы)

* Matthew Weier O'Phinney

### 6.2 Спонсоры

* Paul M. Jones
* Beau Simensen (координатор)

### 6.3 Авторы

* Michael Dowling
* Larry Garfield
* Evert Pot
* Tobias Schultze
* Bernhard Schussek
* Anton Serdyuk
* Phil Sturgeon
* Chris Wilkinson
