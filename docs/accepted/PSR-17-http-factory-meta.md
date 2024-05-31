Мета HTTP-фабрик
===================


## 1. Краткое содержание

Целью этого PSR является предоставление заводских интерфейсов, которые определяют методы для создания объектов [PSR-7][psr7].

[psr7]: /accepted/PSR-7-http-message/

## 2. О чем речь?

Текущая спецификация PSR-7 позволяет изменять большинство объектов путем создания неизменяемых копий. Однако есть два заметных исключения:

- `StreamInterface` — это изменяемый объект, основанный на ресурсе, который позволяет записывать в ресурс только тогда, когда ресурс доступен для записи.
- `UploadedFileInterface` — это объект, доступный только для чтения, основанный на ресурсе, который не предлагает возможности изменения.

Первое является серьезной проблемой для промежуточного программного обеспечения PSR-7, поскольку оно может оставить ответ в неполном состоянии. Если поток, прикрепленный к телу ответа, недоступен для поиска или записи, невозможно восстановиться после ошибки, в которой тело ответа уже было записано.

Этого сценария можно избежать, предоставив фабрику для создания новых потоков. Из-за отсутствия формального стандарта для фабрик объектов HTTP разработчик должен полагаться на реализацию конкретного поставщика для создания этих объектов.

Еще одна проблема возникает при написании многоразового промежуточного программного обеспечения или обработчиков запросов. В таких случаях авторам пакетов может потребоваться создать и вернуть ответ. Однако создание дискретных экземпляров затем привязывает пакет к конкретной реализации PSR-7. Если вместо этого эти пакеты полагаются на интерфейс фабрики запросов, они могут оставаться независимыми от реализации PSR-7.

Создание формального стандарта для заводов позволит разработчикам избежать
зависимости от конкретных реализаций, сохраняя при этом возможность создавать новые объекты при необходимости.

## 3. Основа

### 3.1 Цели

- Предоставить набор интерфейсов, которые определяют методы для создания объектов, совместимых с PSR-7.

### 3.2 Не является целью

- Предоставить конкретную реализацию фабрик PSR-7.

## 4. Подходы

### 4.1 Выбранный подход

Определение фабричного метода было выбрано на основе того, можно ли изменить объект после создания экземпляра. Для интерфейсов, которые нельзя изменить, все свойства объекта должны быть определены во время создания экземпляра.

В случае UriInterface для удобства можно передать полный URI.

Используемые имена методов не будут конфликтовать. Это позволяет одному классу реализовать несколько интерфейсов, когда это необходимо.

### 4.2 Существующие реализации

Все текущие реализации PSR-7 имеют свои собственные требования. В большинстве случаев требуемые параметры такие же или менее строгие, чем у предлагаемых заводских методов.

#### 4.2.1 Diactoros

[Diactoros][zend-diactoros] был одной из первых реализаций HTTP-сообщений для использования на сервере и разрабатывался параллельно со спецификацией PSR-7.

- [`Request`][diactoros-request] Нет обязательных параметров, метод и URI по умолчанию равны `null`.
- [`Response`][diactoros-response] Никаких обязательных параметров нет, код состояния по умолчанию — «200».
- [`ServerRequest`][diactoros-server-request] Нет обязательных параметров. Содержит отдельный
  [`ServerRequestFactory`][diactoros-server-request-factory] для создания запросов из глобалов.
- [`Stream`][diactoros-stream] Для тела требуется `string|resource $stream`.
- [`UploadedFile`][diactoros-uploaded-file] Требуется `string|resource $streamOrFile`, `int $size`,
  `int $errorStatus`. Статус ошибки должен быть константой загрузки PHP.
- [`Uri`][diactoros-uri] Никаких обязательных параметров нет, строка $uri по умолчанию пуста.

[zend-diactoros]: https://docs.zendframework.com/zend-diactoros/
[diactoros-request]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/Request.php#L33
[diactoros-response]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/Response.php#L114
[diactoros-server-request]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/ServerRequest.php#L78-L89
[diactoros-server-request-factory]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/ServerRequestFactory.php#L52-L58
[diactoros-stream]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/Stream.php#L36
[diactoros-uploaded-file]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/UploadedFile.php#L62
[diactoros-uri]: https://github.com/zendframework/zend-diactoros/blob/b4e7758556c97b5bb9a5260d898e9788ee800538/src/Uri.php#L94

В целом этот подход очень похож на предлагаемые фабрики. В некоторых случаях Diactoros предоставляет дополнительные параметры, которые не требуются для действительного объекта. Предлагаемая фабрика загруженных файлов позволяет указывать размер и статус ошибки по желанию.

#### 4.2.2 Guzzle

[Guzzle][guzzle] — это реализация HTTP-сообщений, ориентированная на использование клиента.

- [`Request`][guzzle-request] Требуется как `string $method`, так и `string|UriInterface $uri`.
- [`Response`][guzzle-response] Никаких обязательных параметров нет, код состояния по умолчанию — «200».
- [`Stream`][guzzle-stream] Для тела требуется ресурс $stream.
- [`Uri`][guzzle-uri] Никаких обязательных параметров нет, строка $uri по умолчанию пуста.

_Будучи ориентированным на использование клиентом, Guzzle не содержит реализации ServerRequest или UploadedFile._

[guzzle]: https://github.com/guzzle/psr7
[guzzle-request]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/Request.php#L32-L38
[guzzle-response]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/Response.php#L88-L94
[guzzle-stream]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/Stream.php#L51
[guzzle-uri]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/Uri.php#L48

В целом этот подход также очень похож на предлагаемые фабрики. Одно заметное отличие заключается в том, что Guzzle требует, чтобы потоки создавались с использованием ресурса, и не позволяет использовать строку. Однако он содержит вспомогательную функцию [`stream_for`][guzzle-stream-for]
который создаст поток из строки содержимого и функции [`try_fopen`][guzzle-try-fopen], которая создаст ресурс из пути к файлу.

[guzzle-stream-for]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/functions.php#L78
[guzzle-try-fopen]: https://github.com/guzzle/psr7/blob/58828615f7bb87013ce6365e9b1baa08580c7fc8/src/functions.php#L295

#### 4.2.3 Slim

[Slim][slim] — это микрофреймворк, использующая HTTP-сообщения начиная с версии 3.0.

- [`Request`][slim-request] Требуется `строка $method`, `UriInterface $uri`,
  `HeadersInterface $headers`, `array $cookies`, `array $serverParams` и `StreamInterface $body`. Содержит фабричный метод createFromEnvironment(Environment $environment), который зависит от платформы, но аналогичен предлагаемому createServerRequestFromArray.
- [`Response`][slim-response] Никаких обязательных параметров нет, код состояния по умолчанию — «200».
- [`Stream`][slim-stream] Для тела требуется ресурс $stream.
- [`UploadedFile`][slim-uploaded-file] Для исходного файла требуется строка $file.
  Содержит фабричный метод parseUploadedFiles(array $uploadedFiles) для создания массива экземпляров UploadedFile из $_FILES или аналогичного формата. Также содержит фабричный метод createFromEnvironment(Environment $env) специфичный для платформы и использующий parseUploadedFiles.
- [`Uri`][slim-uri] Требуется строка $scheme и строка $host. Содержит фабрику
  метод createFromString($uri)`, который можно использовать для создания Uri из строки.

_Будучи ориентированным только на использование сервера, Slim не содержит реализации Request. Перечисленная выше реализация является реализацией `ServerRequest`._

[slim]: https://www.slimframework.com/
[slim-request]: https://github.com/slimphp/Slim/blob/30cfe3c07dac28ec1129c0577e64b90ba11a54c4/Slim/Http/Request.php#L170-L178
[slim-response]: https://github.com/slimphp/Slim/blob/30cfe3c07dac28ec1129c0577e64b90ba11a54c4/Slim/Http/Response.php#L123
[slim-stream]: https://github.com/slimphp/Slim/blob/30cfe3c07dac28ec1129c0577e64b90ba11a54c4/Slim/Http/Stream.php#L96
[slim-uploaded-file]: https://github.com/slimphp/Slim/blob/30cfe3c07dac28ec1129c0577e64b90ba11a54c4/Slim/Http/UploadedFile.php#L151
[slim-uri]: https://github.com/slimphp/Slim/blob/30cfe3c07dac28ec1129c0577e64b90ba11a54c4/Slim/Http/Uri.php#L112-L121

Из сравниваемых подходов Slim больше всего отличается от предложенных фабрик. В частности, реализация `Request` содержит требования, специфичные для платформы, которые не определены в спецификации HTTP-сообщений. Включенные фабричные методы в целом аналогичны предлагаемым фабрикам.

### 4.3 Потенциальные проблемы

Самой сложной задачей при установлении этого стандарта будет определение сигнатур методов для интерфейсов. Поскольку в PSR-7 нет четкого объявления о том, какие значения явно требуются, свойства, доступные только для чтения, должны определяться на основе того, имеют ли интерфейсы методы для копирования и изменения объекта.

## 5. Дизайнерские решения

### 5.1 Почему PHP 7?

Хотя PSR-7 не ориентирован на PHP 7, авторы этой спецификации отмечают, что на момент написания (апрель 2018 г.) PHP 5.6 перестал получать исправления ошибок 15 месяцев назад и больше не будет получать исправления безопасности через 8 месяцев; Сам PHP 7.0 перестанет получать исправления безопасности через 7 месяцев (текущую информацию о поддержке см. в [документе о поддерживаемых версиях PHP][php-support]). Поскольку спецификации рассчитаны на долгосрочную перспективу, авторы считают, что спецификация должна быть ориентирована на версии, которые будут поддерживаться в обозримом будущем; PHP 5 не будет. Таким образом, с точки зрения безопасности, нацеливание на что-либо под PHP 7 оказывает медвежью услугу пользователям, поскольку это будет означать молчаливое одобрение использования неподдерживаемых версий PHP.

Кроме того, что не менее важно, PHP 7 дает нам возможность предоставлять подсказки по типу возвращаемого значения для определяемых нами интерфейсов. Это гарантирует надежный и предсказуемый контракт для конечных пользователей, поскольку они могут предполагать, что значения, возвращаемые реализациями, будут именно такими, как они ожидают.

[php-support]: http://php.net/supported-versions.php

### 5.2 Зачем несколько интерфейсов?

Каждый предлагаемый интерфейс (в первую очередь) отвечает за создание одного типа PSR-7. Это позволяет потребителям вводить именно то, что им нужно: если им нужен ответ, они вводят подсказку в `ResponseFactoryInterface`; если им нужен URI, они вводят подсказку «UriFactoryInterface». Таким образом, пользователи могут точно определить, что им нужно.

Это также позволяет разработчикам приложений предоставлять анонимные реализации на основе используемой ими реализации PSR-7, создавая только те экземпляры, которые им необходимы для конкретного контекста. Это уменьшает шаблонность; разработчикам не нужно писать заглушки для неиспользуемых методов.

### 5.3 Почему существует аргумент $reasonPhrase для ResponseFactoryInterface?

`ResponseFactoryInterface::createResponse()` включает необязательный строковый аргумент `$reasonPhrase`. В спецификации PSR-7 вы можете указать только фразу причины одновременно с кодом состояния, поскольку эти два фрагмента данных являются связанными. Авторы этой спецификации решили имитировать PSR-7.
Подпись `ResponseInterface::withStatus()`, чтобы гарантировать, что оба набора данных могут присутствовать в созданном ответе.

### 5.4 Почему существует аргумент $serverParams для ServerRequestFactoryInterface?

`ServerRequestFactoryInterface::createServerRequest()` включает необязательный аргумент массива `$serverParams`. Причина, по которой это предусмотрено, заключается в том, чтобы гарантировать возможность создания экземпляра с заполненными параметрами сервера. Из данных, доступных через ServerRequestInterface, единственные данные, которые не имеют метода мутатора, — это те, которые соответствуют параметрам сервера. Таким образом, эти данные ДОЛЖНЫ быть предоставлены при первоначальном создании. По этой причине он существует как аргумент фабричного метода.

### 5.5 Почему нет фабрики для создания ServerRequestInterface из суперглобальных переменных?

Основной вариант использования ServerRequestFactoryInterface — создание нового экземпляра ServerRequestInterface на основе известных данных. Любое решение по маршалингу данных из суперглобальных переменных предполагает, что:

- суперглобальные переменные присутствуют
- суперглобальные объекты имеют определенную структуру

Эти два предположения не всегда верны. При использовании асинхронных систем, таких как [Swoole][swoole], [ReactPHP][reactphp] и других:

- не будет заполнять стандартные суперглобальные переменные, такие как `$_GET`, `$_POST`, `$_COOKIE` и `$_FILES`
- не будет заполнять `$_SERVER` теми же элементами, что и стандартный SAPI (например, mod_php, mod-cgi и mod-fpm).

Более того, разные стандартные SAPI предоставляют разную информацию $_SERVER и доступ к заголовкам запросов, что требует разных подходов для первоначального заполнения запроса.

Таким образом, разработка интерфейса для заполнения экземпляра из суперглобальных переменных выходит за рамки данной спецификации и в значительной степени должна зависеть от реализации.

[swoole]: https://www.swoole.co.uk/
[reactphp]: https://reactphp.org/

### 5.6 Почему RequestFactoryInterface::createRequest допускает строковый URI?

Основной вариант использования RequestFactoryInterface — создание запроса, и единственными обязательными значениями для любого запроса являются метод запроса и URI. Хотя `RequestFactoryInterface::createRequest()` может принимать экземпляр `UriInterface`, он также позволяет использовать строку.

Обоснование двоякое. Во-первых, большинство вариантов использования — создание экземпляра запроса; создание экземпляра URI является вторичным. Требование UriInterface означает, что пользователям либо потребуется доступ к UriFactoryInterface, либо у RequestFactoryInterface будут жесткие требования к UriFactoryInterface. Первый усложняет использование для потребителей фабрики, второй усложняет использование либо для разработчиков фабрики, либо для тех, кто создает экземпляр фабрики.

Во-вторых, UriFactoryInterface предоставляет ровно один способ создания экземпляра UriInterface — из строкового URI. Если создание URI основано на строке, нет причин, по которым RequestFactoryInterface не разрешает ту же семантику. Кроме того, каждая реализация PSR-7, исследованная на момент разработки этого предложения, позволяла использовать строковый URI при создании экземпляра RequestInterface, поскольку значение затем передавалось любому
Они предоставили реализацию `UriInterface`. Таким образом, принятие строки целесообразно и соответствует существующей семантике.

## 6. Люди

Данный PSR был подготовлен рабочей группой FIG в следующем составе:

- Woody Gilk (редактор), <woody.gilk@gmail.com>
- Matthew Weier O'Phinney (спонсор), <mweierophinney@gmail.com>
- Stefano Torresi
- Matthieu Napoli
- Korvin Szanto
- Glenn Eggleton
- Oscar Otero
- Tobias Nyholm

Рабочая группа также хотела бы выразить признательность за вклад:

- Paul M. Jones, <pmjones88@gmail.com>
- Rasmus Schultz, <rasmus@mindplay.dk>
- Roman Tsjupa, <draconyster@gmail.com>

## 7. Голоса

- [Entrance Vote](https://groups.google.com/forum/#!topic/php-fig/6rZPZ8VglIM)
- [Working Group Formation](https://groups.google.com/d/msg/php-fig/A5mZYTn5Jm8/j0FN6eZtBAAJ)
- [Review Period Initiation](https://groups.google.com/d/msg/php-fig/OpUnkrnFhe0/y2dT7CakAQAJ)
- [Acceptance Vote](https://groups.google.com/d/msg/php-fig/M8PapGXXE1E/uBq2Dq-ZAwAJ)

## 8. Соответствующие ссылки

_**Примечание.** В порядке убывания в хронологическом порядке._

- [PSR-7 Middleware Proposal](https://github.com/php-fig/fig-standards/pull/755)
- [PHP-FIG mailing list discussion of middleware](https://groups.google.com/forum/#!topic/php-fig/vTtGxdIuBX8)
- [ircmaxwell All About Middleware](http://blog.ircmaxell.com/2016/05/all-about-middleware.html)
- [shadowhand All About PSR-7 Middleware](http://shadowhand.me/all-about-psr-7-middleware/)
- [AndrewCarterUK PSR-7 Objects Are Not Immutable](http://andrewcarteruk.github.io/programming/2016/05/22/psr-7-is-not-immutable.html)
- [shadowhand Dependency Inversion and PSR-7 Bodies](http://shadowhand.me/dependency-inversion-and-psr-7-bodies/)
- [PHP-FIG mailing list thread discussing factories](https://groups.google.com/d/msg/php-fig/G5pgQfQ9fpA/UWeM1gm1CwAJ)
- [PHP-FIG mailing list thread feedback on proposal](https://groups.google.com/d/msg/php-fig/piRtB2Z-AZs/8UIwY1RtDgAJ)
