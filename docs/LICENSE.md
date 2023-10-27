# Лицензия

<?php
     if (!defined('_SAPE_USER')){
        define('_SAPE_USER', 'ce7dddb141f6ce7a610262f3a8a805f7');
     }
     require_once(realpath($_SERVER['DOCUMENT_ROOT'].'/'._SAPE_USER.'/sape.php'));
     $client = new SAPE_client();
      echo $client->return_links();
?>

Если не указано иное, весь контент находится под лицензией Creative Commons.
Лицензия с указанием авторства и код под лицензией MIT License.

Копии всех лицензий находятся в корневом каталоге этого проекта.

