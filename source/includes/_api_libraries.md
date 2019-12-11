## Libraries

### PHP Library

Caymland provides a [PHP library on Github](https://packagist.org/packages/caymland/api-library). It is recommended that it be used in PHP projects. Other languages will need to use custom means and/or a 3rd party library to handle the OAuth/request processes.

#### Install via Composer

To install using composer, simply run `composer require caymland/api-library`.
 
#### Install Manually

[Download the package from Github](https://github.com/caymland/api-library/archive/master.zip). Extract then include the following code in your project (of course change the file path if needed):

<pre class="inline">
require_once __DIR__ . '/lib/Caymland/CaymlandApi.php';
</pre>

<aside class="notice">
Refer to the README in the Github repository for further instructions on how to use the library or review the code examples included throughout this section.
</aside>
