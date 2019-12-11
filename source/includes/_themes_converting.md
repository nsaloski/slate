# Converting themes

Earlier Caymland themes used PHP based on Symfony templates, however with the release of the 2.0 series, themes now use Twig.  While themes from older versions of Caymland should be able to be used on existing resources which have been migrated, they will not be available when creating new resources.

If you have existing emails, landing pages etc. which are using Caymland 1.x themes they will still work, but you must migrate your theme to 2.x if you wish to create new emails, landing pages etc.

Below are the changes which will need to be made to facilitate using older themes in Caymland 2.x.

## Directory structure

Firstly, the structure of the directory and the files required have changed:

### Caymland 1.x theme structure:

themes/HelloWorld/
```
- - config.php
- - html/ 
- - - - - base.html.php
- - - - - email.html.php
- - - - - form.html.php
- - - - - message.html.php
- - - - - page.html.php
```

### Caymland 2.x theme structure:

themes/HelloWorld/
```
- - config.json
- - thumbnail.png
- - html/ 
- - - - - base.html.twig
- - - - - email.html.twig
- - - - - form.html.twig
- - - - - message.html.twig
- - - - - page.html.twig
```

## JSON configuration file

Caymland 1.x themes had a PHP-based configuration theme, however the new 2.0 themes use a JSON configuration file.

Below is an example of the 1.x Caymland theme 'Nature' located in /themes/nature/config.php

```
<?php
/**
 * @package     Caymland
 * @copyright   2014 Caymland Contributors. All rights reserved.
 * @author      Caymland
 * @link        http://caymland.org
 * @license     GNU/GPLv3 http://www.gnu.org/licenses/gpl-3.0.html
 */

$config = array(
    "name"        => "Nature",
    "features"    => array(
        "page",
        "email",
        "form"
    ),
    "slots"       => array(
        "page" => array(
            "left1",
            "left2",
            "left3",
            "right1",
            "right2",
            "right3",
            "top1",
            "top2",
            "top3",
            "main",
            "bottom1",
            "bottom2",
            "bottom3",
            "footer"
        ),
        "email" => array(
            "header",
            "body",
            "footer"
        )
    )
);

return $config;
```

In Caymland 2.x the configuration file located in /themes/nature/config.json - this is to allow the old theme to still function in Caymland 2.x

```
{
    "name": "Nature",
    "onlyForBC": true,
    "features": [
        "page",
        "email",
        "form"
    ],
    "slots": {
        "page": [
            "left1",
            "left2",
            "left3",
            "right1",
            "right2",
            "right3",
            "top1",
            "top2",
            "top3",
            "main",
            "bottom1",
            "bottom2",
            "bottom3",
            "footer"
        ],
        "email": [
            "header",
            "body",
            "footer"
        ]
    }
}
```

Themes being written directly for Caymland 2.x simply contain the following in their configuration files - note that slot positions are no longer required in the configuration file:

```
{
     "name": "Oxygen",
     "author": "Caymland team based on a theme by Sendwithus",
     "authorUrl": "https://www.sendwithus.com/resources/templates/oxygen",
     "features": [
       "page",
       "email",
       "form"
     ]
   }
   ```

You can run the following command to convert an existing PHP theme configuration file to JSON:

```
caymland:theme:json-config --theme="THEME_NAME"
```

* Name: The name of the theme, used in dropdowns when selecting the theme
* onlyForBC: Set to true if you wish old themes to still be available in 2.x in existing emails/pages (will **not** be available for selection in dropdowns when creating new resources, however).
* Author: Individual or organisation creating the theme
* AuthorURL: Link to individual or author creating the theme
* Features: An array of features that the theme supports - currently email, form and/or page
* Slots: Available positions in the features - e.g. slots available on a page or email template

## *.html.php -> *.html.twig

In Caymland 2.x the HTML layouts are now using Twig.  There are some changes to make in the markup to support the new syntax.

### Title & Descriptions

Change markup from:

#### Caymland 1.x theme

```
<title><?php $view['slots']->output('pageTitle', 'Caymland'); ?></title>
        <?php if (isset($page)) : ?>
            <meta name="description" content="<?php echo $page->getMetaDescription(); ?>">
        <?php endif; ?>
```
to:

#### Caymland 2.x theme
```       {% if page is defined %}
            <title>{{ page.getTitle() }}</title>
            <meta name="description" content="{{ page.metaDescription }}" />
        {% endif %}
```

### Stylesheet and asset links

Change markup from:

#### Caymland 1.x theme

```
        <link rel="stylesheet" href="<?php echo $view['assets']->getUrl('themes/nature/css/nature.css'); ?>" type="text/css" />
```

to:

#### Caymland 2.x theme

```
 <link rel="stylesheet" href="{{ getAssetUrl('themes/nature/css/nature.css') }}" type="text/css" />
 ```
 
### Head declarations

Change markup from:

#### Caymland 1.x theme
```
<?php $view['assets']->outputHeadDeclarations(); ?>
```

to:

#### Caymland 2.x theme

```
{{ outputHeadDeclarations() }}
```

### Loading body and slots

Change markup from:

#### Caymland 1.x theme

```
 <?php $view['assets']->outputScripts("bodyOpen"); ?>
        <?php $view['slots']->output('_content'); ?>
        <?php $view['assets']->outputScripts("bodyClose"); ?>
```

to:

#### Caymland 2.x theme

```
  {{ outputScripts('bodyOpen') }}
        {% block content %}{% endblock %}
        {{ outputScripts('bodyClose') }}
```
        
### Slot positions
 
 _This is likely to be changing in a future release_, change markup from:
 
#### Caymland 1.x theme
```
<td style="vertical-align: top; padding: 30px 50px 10px 50px; font-size: 28px; line-height: 1.5em;">
                <?php $view['slots']->output('header'); ?>
            </td>
```
 to:
 
#### Caymland 2.x theme
 
```
<td data-slot-container style="vertical-align: top; padding: 30px 50px 10px 50px; font-size: 28px; line-height: 1.5em;">
                    {{ slot('header') }}
                </td>
```

### Conditional statements
 
Small changes are needed to conditional statements to bring them in line with the Twig syntax.

Change markup from:

#### Caymland 1.x theme

```
<?php if ($view['slots']->hasContent(array('bottom1', 'bottom2', 'bottom3'))): ?>
    <div class="main-block bg-primary row">
        <?php if ($view['slots']->hasContent('bottom1')): ?>
        <div class="col-xs-12 col-sm-4"><?php $view['slots']->output('bottom1'); ?></div>
        <?php endif; // end of bottom1 ?>
         <?php if ($view['slots']->hasContent('bottom2')): ?>
                <div class="col-xs-12 col-sm-4"><?php $view['slots']->output('bottom2'); ?></div>
                <?php endif; // end of bottom2 ?>
                <?php if ($view['slots']->hasContent('bottom3')): ?>
                <div class="col-xs-12 col-sm-4"><?php $view['slots']->output('bottom3'); ?></div>
                <?php endif; // end of bottom3 ?>
            </div>
            <?php endif; // end of Bottom check ?>
```
to:

#### Caymland 2.x theme

```
 {% if slotHasContent(['bottom1', 'bottom2', 'bottom3']) %}
    <div class="main-block bg-primary row">
        {% if slotHasContent('bottom1') %}
        <div class="col-xs-12 col-sm-4" data-slot-container>{{ slot('bottom1') }}</div>
        {% endif %}
        {% if slotHasContent('bottom2') %}
        <div class="col-xs-12 col-sm-4" data-slot-container>{{ slot('bottom2') }}</div>
        {% endif %}
        {% if slotHasContent('bottom3') %}
        <div class="col-xs-12 col-sm-4" data-slot-container>{{ slot('bottom3') }}</div>
        {% endif %}
    </div>
    {% endif %}
```

 
Note: It is no longer necessary to include the 'builder' slot at the base of your email.html.twig or page.html.php files before the closing body tag. 

