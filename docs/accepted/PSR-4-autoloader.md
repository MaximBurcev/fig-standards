# Автозагрузчик

Слова «НЕОБХОДИМО» / «ДОЛЖНО» ("MUST"), «НЕДОПУСТИМО» ("MUST NOT"),
«ТРЕБУЕТСЯ» ("REQUIRED"), «НУЖНО» ("SHALL"), «НЕ ПОЗВОЛЯЕТСЯ» ("SHALL NOT"),
«СЛЕДУЕТ» ("SHOULD"), «НЕ СЛЕДУЕТ» ("SHOULD NOT"),
«РЕКОМЕНДУЕТСЯ» ("RECOMMENDED"), «МОЖЕТ» / «ВОЗМОЖНО» ("MAY") и
«НЕОБЯЗАТЕЛЬНО» ("OPTIONAL") в этом документе следует понимать так,
как это описано в [RFC-2119] (и его [переводе]).

## 1. Обзор

Этот PSR описывает спецификацию для [автозагрузки][] классов из файла.
Он полностью совместим и может использоваться в дополнение к любой другой
спецификации автозагрузки, включая [PSR-0][]. В этом PSR также описывается размещения файлов,
которые будут автоматически загружаться в соответствии со спецификацией.

## 2. Спецификация

1. Термин «класс» относится к классам, интерфейсам, трейтам и другим подобным
   структуры.

2. Полное имя класса имеет следующий вид:

        \<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>

    1. Полное имя класса ДОЛЖНО иметь имя пространства имен верхнего уровня,
       также известный как «пространство имен поставщика».

    2. Полное имя класса МОЖЕТ иметь одно или несколько подпространств имен.

    3. Полное имя класса ДОЛЖНО иметь конечное имя класса.

    4. Подчеркивания не имеют особого значения ни в одной части полностью
       квалифицированного имени класса.

    5. Алфавитные символы в полном имени класса МОГУТ быть любыми
       сочетаниями нижнего и верхнего регистра.

    6. Все имена классов ДОЛЖНЫ указываться с учетом регистра.

3. При загрузке файла, соответствующего полностью определённому имени класса, используются следующие правила:

    1. Последовательность из одного и более пространств и подпространств имён (не включая ведущий разделитель
       пространств имён) в полностью определённом имени класса (т.н. «префикс пространств имён») должна соответствовать
       хотя бы одному «базовому каталогу».

    2. Последовательность подпространств имён после «префикса пространства имён» соответствует подкаталогу в «базовом
       каталоге»,
       при этом разделители пространств имён \ соответствуют разделителям каталогов /. Имя подкаталога и имя
       подпространства имён ДОЛЖНЫ совпадать вплоть до регистра символов.

    3. Имя класса, завершающее собой полностью определённое имя, соответствует имени файла с расширением `.php`.
       Имя файла и имя класса ДОЛЖНЫ совпадать вплоть до регистра символов.

4. Реализации автозагрузчика НЕ ДОЛЖНЫ генерировать исключения, НЕ ДОЛЖНЫ вызывать ошибки
   любого уровня и НЕ ДОЛЖНЫ возвращать значения.

## 3. Примеры

В таблице ниже показан соответствующий путь к файлу для данного полного
имя класса, префикс пространства имен и базовый каталог.

| Полностью определённое имя класса    | Префикс пространства имён   | Базовый каталог           | Итоговый путь к файлу
| ----------------------------- |--------------------|--------------------------|-------------------------------------------
| \Acme\Log\Writer\File_Writer  | Acme\Log\Writer    | ./acme-log-writer/lib/   | ./acme-log-writer/lib/File_Writer.php
| \Aura\Web\Response\Status     | Aura\Web           | /path/to/aura-web/src/   | /path/to/aura-web/src/Response/Status.php
| \Symfony\Core\Request         | Symfony\Core       | ./vendor/Symfony/Core/   | ./vendor/Symfony/Core/Request.php
| \Zend\Acl                     | Zend               | /usr/includes/Zend/      | /usr/includes/Zend/Acl.php

Примеры реализации автозагрузчиков, соответствующих данной спецификации, 
представлены в [примерах]. 
Примеры реализации НЕДОПУСТИМО рассматривать как часть спецификации, т.к. они МОГУТ измениться в любое время.

[RFC-2119]:  http://www.ietf.org/rfc/rfc2119.txt

[автозагрузки]: http://php.net/autoload

[PSR-0]: /accepted/PSR-0.md

[примерах]: /accepted/PSR-4-autoloader-examples/

[переводе]: http://rfc.com.ru/rfc2119.htm


