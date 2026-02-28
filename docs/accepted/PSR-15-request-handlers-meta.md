---
description: "Цель данного PSR — определить формальные интерфейсы для обработчиков HTTP-запросов на стороне сервера и middleware, совместимых с PSR-7."
---

Мета-документ: Обработчики HTTP-запросов на стороне сервера
============================================================


## 1. Краткое описание

Цель данного PSR — определить формальные интерфейсы для обработчиков HTTP-запросов на стороне сервера («обработчики запросов») и промежуточного программного обеспечения HTTP-сервера («middleware»), совместимых с HTTP-сообщениями, определёнными в [PSR-7][psr7] или последующих заменяющих PSR.

_Примечание: все ссылки на «обработчики запросов» и «middleware» относятся исключительно к обработке **серверных запросов**._

[psr7]: https://www.php-fig.org/psr/psr-7/

## 2. Зачем это нужно?

Спецификация HTTP-сообщений не содержит никаких ссылок на обработчики запросов или middleware.

Обработчики запросов являются фундаментальной частью любого веб-приложения. Обработчик — это компонент, получающий запрос и формирующий ответ. Практически весь код, работающий с HTTP-сообщениями, содержит какой-либо обработчик запросов.

[Middleware][middleware] существует в экосистеме PHP уже много лет. Общая концепция переиспользуемого middleware была популяризована [StackPHP][stackphp]. После появления HTTP-сообщений в виде PSR многие фреймворки внедрили middleware, использующее интерфейсы HTTP-сообщений.

Принятие формальных интерфейсов для обработчиков запросов и middleware устраняет ряд проблем и даёт следующие преимущества:

* Предоставляет разработчикам формальный стандарт.
* Позволяет любому компоненту middleware работать в любом совместимом фреймворке.
* Устраняет дублирование схожих интерфейсов, определённых в различных фреймворках.
* Исключает незначительные расхождения в сигнатурах методов.

[middleware]: https://en.wikipedia.org/wiki/Middleware
[stackphp]: http://stackphp.com/

## 3. Область применения

### 3.1 Цели

* Создать интерфейс обработчика запросов, использующий HTTP-сообщения.
* Создать интерфейс middleware, использующий HTTP-сообщения.
* Реализовать сигнатуры обработчика запросов и middleware, основанные на лучших практиках.
* Обеспечить совместимость обработчиков запросов и middleware с любой реализацией HTTP-сообщений.

### 3.2 Не является целью

* Определение механизма создания HTTP-ответов.
* Определение интерфейсов для клиентского/асинхронного middleware.
* Определение способа диспетчеризации middleware.

## 4. Подходы к обработчикам запросов

Существует множество подходов к обработчикам запросов, использующим HTTP-сообщения. Однако общий процесс одинаков во всех из них:

На основе HTTP-запроса сформировать HTTP-ответ для данного запроса.

Внутренние требования этого процесса будут варьироваться от фреймворка к фреймворку и от приложения к приложению. Данное предложение не стремится определить, каким должен быть этот процесс.

## 5. Подходы к middleware

В настоящее время существуют два распространённых подхода к middleware, использующему HTTP-сообщения.

### 5.1 Double Pass (двойная передача)

Сигнатура, используемая большинством реализаций middleware, была в основном одинаковой и основана на [Express middleware][express], которое определяется следующим образом:

```
fn(request, response, next): response
```

[express]: http://expressjs.com/en/guide/writing-middleware.html

На основе реализаций middleware, уже используемых фреймворками, принявшими данную сигнатуру, можно выделить следующие общие черты:

* Middleware определяется как [callable][php-callable].
* При вызове middleware передаётся 3 аргумента:
  1. Реализация `ServerRequestInterface`.
  2. Реализация `ResponseInterface`.
  3. `callable`, принимающий запрос и ответ для делегирования следующему middleware.

[php-callable]: http://php.net/manual/language.types.callable.php

Значительное число проектов предоставляет и/или использует один и тот же интерфейс. Данный подход часто называют «double pass» («двойная передача»), поскольку в middleware передаются и запрос, и ответ.

#### 5.1.1 Проекты, использующие Double Pass

* [mindplay/middleman v1](https://github.com/mindplay-dk/middleman/blob/1.0.0/src/MiddlewareInterface.php#L24)
* [relay/relay v1](https://github.com/relayphp/Relay.Relay/blob/1.0.0/src/MiddlewareInterface.php#L24)
* [slim/slim v3](https://github.com/slimphp/Slim/blob/3.4.0/Slim/MiddlewareAwareTrait.php#L66-L75)
* [zendframework/zend-stratigility v1](https://github.com/zendframework/zend-stratigility/blob/1.0.0/src/MiddlewarePipe.php#L69-L79)

#### 5.1.2 Middleware, реализующее Double Pass

* [bitexpert/adroit](https://github.com/bitExpert/adroit)
* [akrabat/rka-ip-address-middleware](https://github.com/akrabat/rka-ip-address-middleware)
* [akrabat/rka-scheme-and-host-detection-middleware](https://github.com/akrabat/rka-scheme-and-host-detection-middleware)
* [bear/middleware](https://github.com/bearsunday/BEAR.Middleware)
* [los/api-problem](https://github.com/Lansoweb/api-problem)
* [los/los-rate-limit](https://github.com/Lansoweb/LosRateLimit)
* [monii/monii-action-handler-psr7-middleware](https://github.com/monii/monii-action-handler-psr7-middleware)
* [monii/monii-nikic-fast-route-psr7-middleware](https://github.com/monii/monii-nikic-fast-route-psr7-middleware)
* [monii/monii-response-assertion-psr7-middleware](https://github.com/monii/monii-response-assertion-psr7-middleware)
* [mtymek/blast-base-url](https://github.com/mtymek/blast-base-url)
* [ocramius/psr7-session](https://github.com/Ocramius/PSR7Session)
* [oscarotero/psr7-middlewares](https://github.com/oscarotero/psr7-middlewares)
* [php-middleware/block-robots](https://github.com/php-middleware/block-robots)
* [php-middleware/http-authentication](https://github.com/php-middleware/http-authentication)
* [php-middleware/log-http-messages](https://github.com/php-middleware/log-http-messages)
* [php-middleware/maintenance](https://github.com/php-middleware/maintenance)
* [php-middleware/phpdebugbar](https://github.com/php-middleware/phpdebugbar)
* [php-middleware/request-id](https://github.com/php-middleware/request-id)
* [relay/middleware](https://github.com/relayphp/Relay.Middleware)

Основным недостатком данного интерфейса является то, что, хотя сам интерфейс и является callable, в настоящее время нет возможности строго типизировать замыкание.

### 5.2 Single Pass (Lambda)

Другой подход к middleware значительно ближе к стилю [StackPHP][stackphp] и определяется следующим образом:

```
fn(request, next): response
```

Middleware, использующее данный подход, как правило, имеет следующие общие черты:

* Middleware определяется с помощью конкретного интерфейса с методом, принимающим запрос для обработки.
* При вызове middleware передаётся 2 аргумента:
  1. Сообщение HTTP-запроса.
  2. Обработчик запросов, которому middleware может делегировать ответственность за формирование HTTP-ответа.

В данном варианте middleware не имеет доступа к ответу до его формирования обработчиком запросов. После этого middleware МОЖЕТ модифицировать ответ перед его возвратом.

Данный подход часто называют «single pass» («одиночная передача») или «lambda», поскольку в middleware передаётся только запрос.

#### 5.2.1 Проекты, использующие Single Pass

Примеров данного подхода в проектах, использующих HTTP-сообщения, меньше, за одним заметным исключением.

[Guzzle middleware][guzzle-middleware] ориентировано на исходящие (клиентские) запросы и использует следующую сигнатуру:

```php
function (RequestInterface $request, array $options): ResponseInterface
```

#### 5.2.2 Дополнительные проекты, использующие Single Pass

Существуют также значимые проекты, появившиеся до HTTP-сообщений, использующие данный подход.

[StackPHP][stackphp] основан на [Symfony HttpKernel][httpkernel] и поддерживает middleware со следующей сигнатурой:

```php
function handle(Request $request, $type, $catch): Response
```

_Примечание: хотя Stack имеет несколько аргументов, объект ответа среди них не предусмотрен._

[Laravel middleware][laravel-middleware] использует компоненты Symfony и поддерживает middleware со следующей сигнатурой:

```php
function handle(Request $request, callable $next): Response
```

[guzzle-middleware]: http://docs.guzzlephp.org/en/latest/handlers-and-middleware.html
[httpkernel]: https://symfony.com/doc/2.0/components/http_kernel/introduction.html
[laravel-middleware]: https://laravel.com/docs/master/middleware

### 5.3 Сравнение подходов

Подход single pass к middleware давно утвердился в сообществе PHP. Это наглядно демонстрирует большое количество пакетов, основанных на StackPHP.

Подход double pass значительно новее, однако он был почти повсеместно принят ранними разработчиками, использующими HTTP-сообщения (PSR-7).

### 5.4 Выбранный подход

Несмотря на почти повсеместное распространение подхода double pass, он имеет существенные проблемы реализации.

Наиболее серьёзная из них состоит в том, что передача пустого ответа не даёт никаких гарантий, что ответ находится в пригодном для использования состоянии. Ситуацию усугубляет то, что middleware МОЖЕТ модифицировать ответ перед передачей его для дальнейшей обработки.

Дополнительную сложность создаёт невозможность гарантировать, что в тело ответа ничего не было записано, что может привести к неполному выводу или к тому, что ответы об ошибках будут отправлены с прикреплёнными заголовками кэша. Также возможно [повреждение содержимого тела][rob-allen-filtering] при записи поверх существующего содержимого, если новое содержимое короче исходного. Наиболее эффективный способ решения этих проблем — всегда предоставлять новый поток при изменении тела сообщения.

[rob-allen-filtering]: https://akrabat.com/filtering-the-psr-7-body-in-middleware/

Ряд специалистов утверждал, что передача ответа способствует инверсии зависимостей. Хотя это действительно помогает избежать зависимости от конкретной реализации HTTP-сообщений, данную проблему также можно решить путём внедрения фабрик в middleware для создания объектов HTTP-сообщений или путём внедрения пустых экземпляров сообщений. С появлением HTTP-фабрик в [PSR-17][psr17] стал возможен стандартный подход к решению задачи инверсии зависимостей.

[psr17]: https://github.com/php-fig/fig-standards/blob/master/proposed/http-factory/http-factory-meta.md

Более субъективная, но также важная проблема состоит в том, что существующее middleware с подходом double pass, как правило, использует подсказку типа `callable` для ссылки на middleware. Это делает строгую типизацию невозможной, поскольку нет гарантии, что передаваемый `callable` реализует сигнатуру middleware, что снижает безопасность во время выполнения.

**В связи с этими существенными проблемами для данного предложения был выбран подход lambda.**

## 6. Проектные решения

### 6.1 Проектирование обработчика запросов

`RequestHandlerInterface` определяет единственный метод, принимающий запрос и ОБЯЗАННЫЙ возвращать ответ. Обработчик запросов МОЖЕТ делегировать обработку другому обработчику.

#### Почему требуется серверный запрос?

Чтобы явно указать, что обработчик запросов может использоваться только в серверном контексте. В клиентском контексте вместо ответа, скорее всего, возвращался бы [promise][promises].

[promises]: https://promisesaplus.com/

#### Почему используется термин «handler» (обработчик)?

Термин «handler» означает нечто, предназначенное для управления или контроля. Применительно к обработке запросов обработчик запросов — это точка, в которой запрос должен быть обработан для формирования ответа.

В отличие от термина «delegate», который использовался в предыдущей версии данной спецификации, внутреннее поведение данного интерфейса не определяется. До тех пор пока обработчик запросов в конечном счёте формирует ответ, он является корректным.

#### Почему обработчик запросов не использует `__invoke`?

Использование `__invoke` менее прозрачно, чем применение именованного метода. Кроме того, именованный метод упрощает вызов обработчика запросов, когда он присвоен переменной класса, без использования `call_user_func` или другого менее распространённого синтаксиса.

_Дополнительную информацию см. в [обсуждении PHP-FIG по FrameInterface][]._

### 6.2 Проектирование middleware

`MiddlewareInterface` определяет единственный метод, принимающий HTTP-запрос и обработчик запросов и обязанный возвращать ответ. Middleware МОЖЕТ:

- Преобразовывать запрос перед передачей его обработчику запросов.
- Преобразовывать ответ, полученный от обработчика запросов, перед его возвратом.
- Создавать и возвращать ответ без передачи запроса обработчику запросов, тем самым самостоятельно обрабатывая запрос.

При делегировании от одного middleware к другому в цепочке один из подходов для систем диспетчеризации — использование промежуточного обработчика запросов, объединяющего цепочку middleware как способ связывания компонентов middleware. Последний или самый внутренний middleware будет выступать в роли шлюза к коду приложения и формировать ответ на основе его результатов; в качестве альтернативы middleware МОЖЕТ делегировать эту ответственность специализированному обработчику запросов.

#### Почему middleware не использует `__invoke`?

Это создало бы конфликт с существующим middleware, реализующим подход double pass и желающим реализовать интерфейс middleware в целях прямой совместимости с данной спецификацией.

#### Почему метод называется `process()`?

Мы рассмотрели ряд существующих монолитных фреймворков и фреймворков на основе middleware, чтобы определить, какой метод (или методы) каждый из них определяет для обработки входящих запросов. Мы обнаружили, что наиболее распространены следующие варианты:

- `__invoke` (в системах middleware, таких как Slim, Expressive, Relay и др.)
- `handle` (в частности, в программном обеспечении, производном от Symfony [HttpKernel][HttpKernel])
- `dispatch` (Zend Framework [DispatchableInterface][DispatchableInterface])

[HttpKernel]: https://symfony.com/doc/current/components/http_kernel.html
[DispatchableInterface]: https://github.com/zendframework/zend-stdlib/blob/980ce463c29c1a66c33e0eb67961bba895d0e19e/src/DispatchableInterface.php

Мы решили обеспечить прямую совместимость для таких классов, чтобы они могли перепрофилировать себя в middleware (или middleware, совместимое с данной спецификацией), и поэтому нам нужно было выбрать имя, не находящееся в общем употреблении. В результате мы выбрали `process`, указывая тем самым на _обработку_ запроса.

#### Почему требуется серверный запрос?

Чтобы явно указать, что middleware может использоваться только в синхронном, серверном контексте.

Хотя не всем middleware потребуются дополнительные методы, определённые интерфейсом серверного запроса, исходящие запросы, как правило, обрабатываются асинхронно и обычно возвращают [promise][promises] ответа. (Это в первую очередь обусловлено тем, что несколько запросов могут выполняться параллельно и обрабатываться по мере поступления.) Решение задач асинхронного жизненного цикла запрос/ответ выходит за рамки данного предложения.

Попытка определить клиентское middleware была бы на данном этапе преждевременной. Любое будущее предложение, ориентированное на обработку клиентских запросов, должно иметь возможность определить стандарт, специфичный для природы асинхронного middleware.

_Дополнительную информацию см. в [обсуждении PHP-FIG о middleware на стороне клиента и сервера][]._

#### Какова роль обработчика запросов?

Middleware выполняет следующие роли:

- Самостоятельное формирование ответа. Если выполнены определённые условия запроса, middleware МОЖЕТ сформировать и вернуть ответ.

- Возврат результата обработчика запросов. В случаях, когда middleware не МОЖЕТ самостоятельно сформировать ответ, оно МОЖЕТ делегировать эту задачу обработчику запросов; иногда это МОЖЕТ предполагать предоставление преобразованного запроса (например, для внедрения атрибута запроса или результатов разбора тела запроса).

- Манипуляция ответом, возвращённым обработчиком запросов, и его возврат. В некоторых случаях middleware МОЖЕТ быть заинтересовано в изменении ответа, возвращённого обработчиком запросов (например, для сжатия тела ответа в gzip, добавления заголовков CORS и т. д.). В таких случаях middleware перехватывает ответ, возвращённый обработчиком запросов, и возвращает преобразованный ответ по завершении.

В двух последних случаях middleware МОЖЕТ содержать код, подобный следующему:

```php
// Straight delegation:
return $handler->handle($request);

// Capturing the response to manipulate:
$response = $handler->handle($request);
```

Поведение обработчика полностью определяется разработчиком — главное, чтобы в итоге формировался ответ.

В одном из типичных сценариев обработчик реализует _очередь_ или _стек_ экземпляров middleware внутри себя. В таких случаях вызов `$handler->handle($request)` продвигает внутренний указатель, извлекает middleware, связанное с этим указателем, и вызывает его с помощью `$middleware->process($request, $this)`. Если middleware больше не осталось, обработчик, как правило, либо выбрасывает исключение, либо возвращает заранее заготовленный ответ.

Другая возможность — _routing middleware_ («маршрутизирующее middleware»), которое сопоставляет входящий серверный запрос с конкретным обработчиком и возвращает ответ, сформированный при его выполнении. Если сопоставить запрос с обработчиком не удаётся, вместо этого выполняется обработчик, переданный в middleware. (Данный механизм МОЖЕТ применяться даже совместно с очередями и стеками middleware.)

### 6.3 Примеры взаимодействия интерфейсов

Два интерфейса — `RequestHandlerInterface` и `MiddlewareInterface` — были спроектированы для совместной работы. Middleware приобретает гибкость, будучи отделённым от какого-либо объемлющего прикладного уровня и опираясь исключительно на предоставленный обработчик запросов для формирования ответа.

Ниже продемонстрированы два подхода к системам диспетчеризации middleware, которые рабочая группа наблюдала и/или реализовывала. Кроме того, приведены примеры переиспользуемого middleware, показывающие, как писать слабо связанный middleware.

Обратите внимание: данные подходы не претендуют на роль окончательных или исключительных способов определения систем диспетчеризации middleware.

#### Обработчик запросов на основе очереди

В данном подходе обработчик запросов поддерживает очередь middleware и резервный ответ на случай, если очередь исчерпана без формирования ответа. При выполнении первого middleware очередь передаёт себя в качестве обработчика запросов данному middleware.

```php
class QueueRequestHandler implements RequestHandlerInterface
{
    private $middleware = [];
    private $fallbackHandler;

    public function __construct(RequestHandlerInterface $fallbackHandler)
    {
        $this->fallbackHandler = $fallbackHandler;
    }

    public function add(MiddlewareInterface $middleware)
    {
        $this->middleware[] = $middleware;
    }

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        // Last middleware in the queue has called on the request handler.
        if (0 === count($this->middleware)) {
            return $this->fallbackHandler->handle($request);
        }

        $middleware = array_shift($this->middleware);
        return $middleware->process($request, $this);
    }
}
```

Загрузка приложения в таком случае может выглядеть следующим образом:

```php
// Fallback handler:
$fallbackHandler = new NotFoundHandler();

// Create request handler instance:
$app = new QueueRequestHandler($fallbackHandler);

// Add one or more middleware:
$app->add(new AuthorizationMiddleware());
$app->add(new RoutingMiddleware());

// execute it:
$response = $app->handle(ServerRequestFactory::fromGlobals());
```

В данной системе присутствуют два обработчика запросов: один, формирующий ответ, если последнее middleware делегирует обработку обработчику запросов, и один — для диспетчеризации слоёв middleware. (В данном примере `RoutingMiddleware`, скорее всего, будет выполнять скомпонованные обработчики при успешном совпадении маршрута; подробнее об этом ниже.)

Данный подход имеет следующие преимущества:

- Middleware не обязано знать ни о каком другом middleware или о том, как оно скомпоновано в приложении.
- `QueueRequestHandler` не зависит от конкретной реализации PSR-7.
- Middleware выполняется в том порядке, в котором оно добавляется в приложение, что делает код явным.
- Формирование «резервного» ответа делегируется разработчику приложения. Это позволяет разработчику самостоятельно определить, должно ли это быть условие «404 Not Found», страница по умолчанию или что-то иное.

#### Обработчик запросов на основе декорирования

В данном подходе реализация обработчика запросов декорирует как экземпляр middleware, так и резервный обработчик запросов, передаваемый ему. Приложение строится снаружи внутрь, при этом каждый «слой» обработчика запросов передаётся следующему внешнему слою.

```php
class DecoratingRequestHandler implements RequestHandlerInterface
{
    private $middleware;
    private $nextHandler;

    public function __construct(MiddlewareInterface $middleware, RequestHandlerInterface $nextHandler)
    {
        $this->middleware = $middleware;
        $this->nextHandler = $nextHandler;
    }

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        return $this->middleware->process($request, $this->nextHandler);
    }
}

// Create a response prototype to return if no middleware can produce a response
// on its own. This could be a 404, 500, or default page.
$responsePrototype = (new Response())->withStatus(404);
$innerHandler = new class ($responsePrototype) implements RequestHandlerInterface {
    private $responsePrototype;

    public function __construct(ResponseInterface $responsePrototype)
    {
        $this->responsePrototype = $responsePrototype;
    }

    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        return $this->responsePrototype;
    }
};

$layer1 = new DecoratingRequestHandler(new RoutingMiddleware(), $innerHandler);
$layer2 = new DecoratingRequestHandler(new AuthorizationMiddleware(), $layer1);

$response = $layer2->handle(ServerRequestFactory::fromGlobals());
```

Аналогично middleware на основе очереди, обработчики запросов в данной системе выполняют две функции:

- Формирование резервного ответа, если ни один другой слой этого не делает.
- Диспетчеризация middleware.

#### Примеры переиспользуемого middleware

В приведённых выше примерах в каждом из них скомпоновано по два middleware. Чтобы они работали в обоих сценариях, необходимо написать их таким образом, чтобы они взаимодействовали корректно.

Разработчикам middleware, стремящимся к максимальной совместимости, рекомендуется руководствоваться следующими принципами:

- Проверять запрос на соответствие обязательному условию. Если условие не выполнено, использовать скомпонованный прототип ответа или скомпонованную фабрику ответов для формирования и возврата ответа.

- Если предварительные условия выполнены, делегировать создание ответа предоставленному обработчику запросов, при необходимости передавая «новый» запрос путём преобразования предоставленного (например, `$handler->handle($request->withAttribute('foo', 'bar')`).

- Либо передавать ответ, возвращённый обработчиком запросов, без изменений, либо предоставлять новый ответ путём манипуляции возвращённым (например, `return $response->withHeader('X-Foo-Bar', 'baz')`).

`AuthorizationMiddleware` — это пример, задействующий все три принципа:

- Если авторизация требуется, но запрос не авторизован, используется скомпонованный прототип ответа для формирования ответа «unauthorized» («не авторизован»).
- Если авторизация не требуется, запрос делегируется обработчику без изменений.
- Если авторизация требуется и запрос авторизован, запрос делегируется обработчику, а возвращённый ответ подписывается на основе запроса.

```php
class AuthorizationMiddleware implements MiddlewareInterface
{
    private $authorizationMap;

    public function __construct(AuthorizationMap $authorizationMap)
    {
        $this->authorizationMap = $authorizationMap;
    }

    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        if (! $this->authorizationMap->needsAuthorization($request)) {
            return $handler->handle($request);
        }

        if (! $this->authorizationMap->isAuthorized($request)) {
            return $this->authorizationMap->prepareUnauthorizedResponse();
        }

        $response = $handler->handle($request);
        return $this->authorizationMap->signResponse($response, $request);
    }
}
```

Обратите внимание: middleware не интересует реализация обработчика запросов — оно лишь использует его для формирования ответа при выполнении предварительных условий.

Реализация `RoutingMiddleware`, описанная ниже, следует аналогичному принципу: она анализирует запрос на предмет соответствия известным маршрутам. В данной реализации маршруты сопоставляются с обработчиками запросов, и middleware по существу делегирует им задачу формирования ответа. Однако в случае, когда ни один маршрут не совпадает, выполняется обработчик, переданный в middleware.

```php
class RoutingMiddleware implements MiddlewareInterface
{
    private $router;

    public function __construct(Router $router)
    {
        $this->router = $router;
    }

    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        $result = $this->router->match($request);

        if ($result->isSuccess()) {
            return $result->getHandler()->handle($request);
        }

        return $handler->handle($request);
    }
}
```

## 7. Участники

Данный PSR был подготовлен рабочей группой FIG в следующем составе:

* Matthew Weier O'Phinney (спонсор), <mweierophinney@gmail.com>
* Woody Gilk (редактор), <woody.gilk@gmail.com>
* Glenn Eggleton
* Matthieu Napoli
* Oscar Otero
* Korvin Szanto
* Stefano Torresi

Рабочая группа также выражает признательность за вклад следующим участникам:

* Jason Coward, <jason@opengeek.com>
* Paul M. Jones, <pmjones88@gmail.com>
* Rasmus Schultz, <rasmus@mindplay.dk>

## 8. Голосования

* [Формирование рабочей группы](https://groups.google.com/d/msg/php-fig/rPFRTa0NODU/tIU9BZciAgAJ)
* [Начало периода рецензирования](https://groups.google.com/d/msg/php-fig/mfTrFinTvEM/PiYvU2S6BAAJ)
* [Принятие](https://groups.google.com/d/msg/php-fig/bhQmHt39hJE/ZCYrK_O2AQAJ)

## 9. Полезные ссылки

_**Примечание:** порядок — от новых к старым._

* [Тема в списке рассылки PHP-FIG][PHP-FIG mailing list thread]
* [Предложение по middleware от The PHP League][The PHP League middleware proposal]
* [Обсуждение PHP-FIG по FrameInterface][PHP-FIG discussion of FrameInterface]
* [Обсуждение PHP-FIG о middleware на стороне клиента и сервера][PHP-FIG discussion about client vs server side middleware]

## 10. Замечания и исправления

...

[PHP-FIG mailing list thread]: https://groups.google.com/d/msg/php-fig/vTtGxdIuBX8/NXKieN9vDQAJ
[The PHP League middleware proposal]: https://groups.google.com/d/msg/thephpleague/jyztj-Nz_rw/I4lHVFigAAAJ
[PHP-FIG discussion of FrameInterface]: https://groups.google.com/d/msg/php-fig/V12AAcT_SxE/aRXmNnIVCwAJ
[PHP-FIG discussion about client vs server side middleware]: https://groups.google.com/d/topic/php-fig/vBk0BRgDe2s/discussion
