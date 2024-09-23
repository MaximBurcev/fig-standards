# Рекомендации по стандартам PHP (PHP PSR)

В соответствии с [регламентом рабочего процесса PSR][workflow] каждый PSR имеет статус в процессе работы. После того, как предложение пройдет вступительное голосование, оно будет указано здесь как «Черновик». Если PSR не отмечен как «Принято», он может быть изменен. Черновик может кардинально измениться, но в Ревью будут внесены лишь незначительные изменения.

Как также описано в [Постановлении о рабочем процессе PSR] [workflow]. Редактор или редакторы предложения являются, по сути, ведущими участниками и авторами PSR, и их поддерживают два члена с правом голоса. Члены с правом голоса являются координатором, который отвечает за управление этапом проверки и голосованием; и второй спонсор.
## Индекс по статусу


### Принято

| Номер | Название                                | Редактор                       | Координатор             | Sponsor                 |
|:-----:|-----------------------------------------|--------------------------------|-------------------------|-------------------------|
|   1   | [Основной стандарт кодирования][psr1]   | Paul M. Jones                  | _N/A_                   | _N/A_                   |
|   3   | [Общий интерфейс для логирования][psr3] | Jordi Boggiano                 | _N/A_                   | _N/A_                   |
|   4   | [Стандарт автозагрузки][psr4]            | Paul M. Jones                  | Phil Sturgeon           | Larry Garfield          |
|   6   | [Интерфейс кэширования][psr6]               | Larry Garfield                 | Paul Dragoonis          | Robert Hafner           |
|   7   | [Интерфейс HTTP-сообщений][psr7]          | Matthew Weier O'Phinney        | Beau Simensen           | Paul M. Jones           |
|  11   | [Интерфейс контейнера][psr11]            | Matthieu Napoli, David Négrier | Matthew Weier O'Phinney | Korvin Szanto           |
|  12   | [Расширенное руководство по стилю кодирования][psr12]    | Korvin Szanto                  | Alexander Makarov       |  Chris Tankersley       | 
|  13   | [Гипермедийные ссылки][psr13]               | Larry Garfield                 | Matthew Weier O'Phinney | Marc Alexander          |
|  14   | [Диспетчер событий][psr14]               | Larry Garfield                 | _N/A_                   | Cees-Jan Kiewiet        |
|  15   | [HTTP-обработчики][psr15]                  | Woody Gilk                     | _N/A_                   | Matthew Weier O'Phinney |
|  16   | [Простой кэш][psr16]                   | Paul Dragoonis                 | Jordi Boggiano          | Fabien Potencier        |
|  17   | [HTTP-фабрики][psr17]                 | Woody Gilk                     | _N/A_                   | Matthew Weier O'Phinney |
|  18   | [HTTP-клиент][psr18]                    | Tobias Nyholm                  | _N/A_                   | Sara Golemon            |

### Черновик

| Номер | Название                | Редактор      |
|:-----:|-------------------------|---------------|
|   5   | [Стандарт PHPDoc][psr5] | Chuck Burgess |
|  19   | [Теги PHPDoc][psr19]    | Chuck Burgess |
|  20   | [Часы][psr20]          | Chris Seufert |

### Не поддерживается

| Номер | Название                                  | Редактор       |
|:-----:|-------------------------------------------|----------------|
|   8   | [Удобный интерфейс][psr8]                 | Larry Garfield |
|   9   | [Советы по безопасности][psr9]            | Michael Hess   |
|  10   | [Процесс сообщений о безопасности][psr10] | Michael Hess   |

### Устарело

| Номер | Название                     | Редактор                |
|:-----:|------------------------------|-------------------------|
|   0   | [Стандарт автозагрузки][psr0] | Matthew Weier O'Phinney |
|   2   | [Руководство по стилю кодирования][psr2]   | Paul M. Jones           |

## Числовой индекс

| Номер | Название                                | Редакторы                      | Статус            |
|:-----:|-----------------------------------------|--------------------------------|-------------------|
|   0   | [Стандарт автозагрузки][psr0]            | Matthew Weier O'Phinney        | Устарело          |
|   1   | [Основной стандарт кодирования][psr1]   | Paul M. Jones                  | Принято           |
|   2   | [Руководство по стилю кодирования][psr2]              | Paul M. Jones                  | Устарело          |
|   3   | [Общий интерфейс для логирования][psr3] | Jordi Boggiano                 | Принято           |
|   4   | [Стандарт автозагрузки][psr4]            | Paul M. Jones                  | Принято           |
|   5   | [PHPDoc Standard][psr5]                 | Chuck Burgess                  | Черновик          |
|   6   | [Интерфейс кэширования][psr6]               | Larry Garfield                 | Принято           |
|   7   | [Интерфейс HTTP-сообщений][psr7]          | Matthew Weier O'Phinney        | Принято           |
|   8   | [Huggable Interface][psr8]              | Larry Garfield                 | Не поддерживается |
|   9   | [Security Advisories][psr9]             | Michael Hess                   | Не поддерживается         |
|  10   | [Security Reporting Process][psr10]     | Michael Hess                   | Не поддерживается         |
|  11   | [Интерфейс контейнера][psr11]            | Matthieu Napoli, David Négrier | Принято           |
|  12   | [Расширенное руководство по стилю кодирования][psr12]    | Korvin Szanto                  | Принято           |
|  13   | [Гипермедийные ссылки][psr13]               | Larry Garfield                 | Принято           |
|  14   | [Диспетчер событий][psr14]               | Larry Garfield                 | Принято           |
|  15   | [HTTP-обработчики][psr15]                  | Woody Gilk                     | Принято           |
|  16   | [Простой кэш][psr16]                   | Paul Dragoonis                 | Принято           |
|  17   | [HTTP-фабрики][psr17]                 | Woody Gilk                     | Принято           |
|  18   | [HTTP-клиент][psr18]                    | Tobias Nyholm                  | Принято           |
|  19   | [Теги PHPDoc][psr19]                    | Chuck Burgess                  | Черновик          |
|  20   | [Часы][psr20]                          | Chris Seufert                  | Черновик          |

[workflow]: bylaws/002-psr-workflow.md
[psr0]: accepted/PSR-0.md
[psr1]: accepted/PSR-1-basic-coding-standard.md
[psr2]: accepted/PSR-2-coding-style-guide.md
[psr3]: accepted/PSR-3-logger-interface.md
[psr4]: accepted/PSR-4-autoloader.md
[psr5]: proposed/phpdoc.md
[psr6]: accepted/PSR-6-cache.md
[psr7]: accepted/PSR-7-http-message.md

[//]: # ([psr8]: proposed/psr-8-hug/)
[//]: # ([psr9]: proposed/security-disclosure-publication.md)
[//]: # ([psr10]: proposed/security-reporting-process.md)

[//]: # ([psr11]: accepted/PSR-11-container.md)
[psr12]: accepted/PSR-12-extended-coding-style-guide.md

[//]: # ([psr13]: accepted/PSR-13-links.md)
[//]: # ([psr14]: accepted/PSR-14-event-dispatcher.md)
[//]: # ([psr15]: accepted/PSR-15-request-handlers.md)
[//]: # ([psr16]: accepted/PSR-16-simple-cache.md)
[psr17]: accepted/PSR-17-http-factory.md
[psr18]: accepted/PSR-18-http-client.md
[psr19]: proposed/phpdoc-tags.md

[//]: # ([psr20]: proposed/clock.md)
