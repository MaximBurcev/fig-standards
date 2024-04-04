Стандарт автозагрузки
====================

> **Устарело** — по состоянию на 21 октября 2014 г. PSR-0 помечен как устаревший. [PSR-4] теперь рекомендуется
как альтернатива.

[PSR-4]: ../accepted/PSR-4-autoloader.md

Ниже описаны обязательные требования, которые необходимо соблюдать  для совместимости с автозагрузчиком.

Обязательно
---------

* Полное пространство имен и класс должны иметь следующую
  структуру `\<Имя поставщика>\(<Пространство имен>\)*<Имя класса>`
* Каждое пространство имен должно иметь пространство имен верхнего уровня («Имя поставщика»).
* Каждое пространство имен может иметь столько подпространств имен, сколько необходимо.
* Каждый разделитель пространства имен преобразуется в `DIRECTORY_SEPARATOR`, при
  загрузки из файловой системы.
* Каждый символ `_` в имени класса преобразуется в
  `DIRECTORY_SEPARATOR`. Символ `_` не имеет специального значения в
  пространстве имен.
* Полное пространство имен и класс имеют суффикс `.php`, при
  загрузки из файловой системы.
* Алфавитные символы в именах поставщиков, пространствах имен и именах классов могут
  иметь любую комбинацию строчных и прописных букв.

Примеры
--------

* `\Doctrine\Common\IsolatedClassLoader` => `/path/to/project/lib/vendor/Doctrine/Common/IsolatedClassLoader.php`
* `\Symfony\Core\Request` => `/path/to/project/lib/vendor/Symfony/Core/Request.php`
* `\Zend\Acl` => `/path/to/project/lib/vendor/Zend/Acl.php`
* `\Zend\Mail\Message` => `/path/to/project/lib/vendor/Zend/Mail/Message.php`

Знаки подчеркивания в пространствах имен и именах классов
-----------------------------------------

* `\namespace\package\Class_Name` => `/path/to/project/lib/vendor/namespace/package/Class/Name.php`
* `\namespace\package_name\Class_Name` => `/path/to/project/lib/vendor/namespace/package_name/Class/Name.php`

Стандарты, которые мы устанавливаем здесь, должны быть наименьшими доработками для
безболезненной совместимости автозагрузчика. Вы можете проверить, что 
следуя этим стандартам и используя этот образец реализации SplClassLoader
для загрузки классов в PHP 5.3.

Пример реализации
----------------------

Ниже приведен пример функции, чтобы просто продемонстрировать, как
предлагаемые стандарты загружаются автоматически.

~~~php
<?php

function autoload($className)
{
    $className = ltrim($className, '\\');
    $fileName  = '';
    $namespace = '';
    if ($lastNsPos = strrpos($className, '\\')) {
        $namespace = substr($className, 0, $lastNsPos);
        $className = substr($className, $lastNsPos + 1);
        $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
    }
    $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';

    require $fileName;
}
spl_autoload_register('autoload');
~~~

Реализация SplClassLoader
-----------------------------

Ниже приведена примерная реализация SplClassLoader, которая может
загрузить свои классы, если вы следуете рекомендациям автозагрузчика
стандартов, предложенных выше. 
Это текущий рекомендуемый способ загрузки в PHP 5.3 классов, соответствующие этим стандартам.

* [http://gist.github.com/221634](http://gist.github.com/221634)


