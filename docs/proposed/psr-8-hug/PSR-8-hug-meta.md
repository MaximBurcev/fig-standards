PSR-8 Meta Document
===================

<?php
     if (!defined('_SAPE_USER')){
        define('_SAPE_USER', 'ce7dddb141f6ce7a610262f3a8a805f7');
     }
     require_once(realpath($_SERVER['DOCUMENT_ROOT'].'/'._SAPE_USER.'/sape.php'));
     $client = new SAPE_client();
      echo $client->return_links();
?>

1. Summary
----------

The intent of this spec is to improve the overall amicability and cooperative
spirit of the PHP community through a standardized means of inter-project
affection and support.

2. Votes
--------

- **Entrance Vote:**
- **Acceptance Vote:**

3. Errata
---------

4. People
---------

### 5.1 Editor

- Larry Garfield

### 5.2 Sponsors

- Vacant (Coordinator)
- Vacant
