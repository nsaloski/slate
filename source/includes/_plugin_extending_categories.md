### Extending Categories

Caymland has a CategoryBundle that can be leveraged to incorporate categories into a plugin.

#### Adding categories
As of Caymland 1.2.1, register categories through the plugin's config.php file by adding the following as a key to the returned config array:

<pre class="inline">
    'categories' => array(
        'plugin:helloWorld' => 'caymland.helloworld.world.categories'
    ),
</pre>

The category keys need be prefixed with `plugin:` as it is used in determining permissions to manage categories. The `helloWorld` should match the permission class name.

#### Configuring Categories for Menu

It is now recommended to not show the category in the main Menu.

To add a category menu item for the plugin, simply add the following to `menu` [config](#menu) for whichever menu the item should appear (`main` or `admin`):

<pre class="inline">
    'caymland.category.menu.index' => array(
        'bundle' => 'plugin:helloWorld'
    )
</pre>

The `bundle` value needs be prefixed with `plugin:` as it is used in determining permissions to manage categories. The `helloWorld` should be the bundle name of the plugin.

#### Configuring Categories for Routes

There is no need to add custom routes for categories. However, when [generating a URL](#router) to the plugin's category list, use

<pre class="inline">
$categoryUrl = $router->generateUrl('caymland_category_index', array('bundle' => 'plugin:helloWorld'));
</pre>

#### Including Category in Forms

To add a category select list to a [form](#forms), use `category` as the form type and pass `bundle` as an option:
  
<pre class="inline">
    //add category
    $builder->add('category', 'category', array(
        'bundle' => 'plugin:helloWorld'
    ));
</pre>

#### Restricting Category Management

To restrict access to catgories, use the following in the plugin's [Permission class](#roles-and-permissions).

In `__construct()` add `$this->addStandardPermissions('categories');` then in `buildForm()`, add `$this->addStandardFormFields('helloWorld', 'categories', $builder, $data);`.

See a code example in [Roles and Permissions](#roles-and-permissions).

The two standard helper methods will add the permissions of `view`, `edit`, `create`, `delete`, `publish`, and `full` for categories.


