### Forms

Caymland leverages Symfony's Form component and form classes. Refer to [Symfony's documentation](http://symfony.com/doc/current/book/forms.html#creating-form-classes) for more information.

#### Form Types

As stated in Symfony's documentation referenced above, form type classes are the best way to go.  Caymland makes it easy to register [form type services](http://symfony.com/doc/2.8/cookbook/form/create_custom_field_type.html#defining-the-field-type) through the bundle's config file. Refer to the [Services](#services) section.
 
#### Data Sanitization

```php
<?php
// plugins/HelloWorldBundle/Form/Type/WorldType.php

// ...
use Caymland\CoreBundle\Form\EventListener\CleanFormSubscriber;

// ...
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->addEventSubscriber(
            new CleanFormSubscriber(
                [
                    'content'    => 'html', 
                    'customHtml' => 'html'
                ]
            )
        );
        
       // ...
    }
// ...
```
Form data is not automatically sanitized. Caymland provides a form event subscriber to handle this. 

In your [form type class](http://symfony.com/doc/2.8/cookbook/form/create_custom_field_type.html#defining-the-field-type), register the `Caymland\CoreBundle\Form\EventListener\CleanFormSubscriber` event subscriber. 
 
 The array provided to `CleanFormSubscriber` should contain the names of the form fields as keys and the values the masks to use to sanitize the data. Any un-specified form field will use the `clean` mask by default.
 
#### Manipulating Forms

A form event listener must be used if a form needs to be manipulated based on submitted data such as changing defined fields, adjust constraints, or changing select choices based on submitted values. Refer to [Symfony's documentation](http://symfony.com/doc/2.8/cookbook/form/dynamic_form_modification.html) on this.
 
#### Validating Data

Review [Symfony's form validation documentation](http://symfony.com/doc/2.8/book/forms.html#form-validation) for a general overview.

There are two common means of validating form data.

##### Using Entity Static Callback
```php
<?php
// plugins/HelloWorldBundle/Entity/World.php

// ...
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Validator\Mapping\ClassMetadata;
use Symfony\Component\Form\Form;

// ...
  /**
     * @param ClassMetadata $metadata
     */
    public static function loadValidatorMetadata (ClassMetadata $metadata)
    {
        $metadata->addPropertyConstraint(
            'name',
            new NotBlank(
                array(
                    'message' => 'caymland.core.name.required'
                )
            )
        );
        
        $metadata->addPropertyConstraint(
            'population', 
            new NotBlank(
                array(
                    'message' => 'caymland.core.value.required',
                    'groups'  => array('VisitedWorld')
                )
            
            )
        );
    }
    
    /**
     * @param Form $form
     *
     * @return array
     */
    public static function determineValidationGroups (Form $form)
    {
        $data   = $form->getData();
        $groups = array('AllWorlds');

        if (!$data->getId() || ($data->getId() && $data->getVisitCount() > 0)) {
            $groups[] = 'VisitedWorld';
        }

        return $groups;
    }
// ...
```

If the underlying data of a form is an Entity object, a static method `loadValidatorMetadata` can be defined in the Entity class. This will automatically be called when Symfony is processing form data.

A form can also use [`validation_groups`](http://symfony.com/doc/2.8/book/forms.html#validation-groups) to change the order of data to be validated or only validate if certain criteria is true. For example, only validate a password confirmation field if the first password field passes validation. When registering a validation group in the form type class, one can use static callback that can be used to determine what validation group(s) should be used. 

```php
<?php
// plugins/HelloWorldBundle/Form/Type/WorldType.php

//...
    /**
     * {@inheritdoc}
     */
    public function setDefaultOptions (OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'data_class'        => 'CaymlandPlugin\HelloWorld\Entity\World',
            'validation_groups' => array(
                'CaymlandPlugin\HelloWorld\Entity\World',
                'determineValidationGroups',
            )
        ));
    }
// ...
```

<div class="clear-right"></div>

##### Using Constraints

A [form type service](http://symfony.com/doc/2.8/cookbook/form/create_custom_field_type.html#defining-the-field-type) can also register [constraints](http://symfony.com/doc/2.8/reference/constraints.html) when defining the form fields.

```php
<?php
// plugins/HelloWorldBundle/Form/Type/WorldType.php

// ...

use Symfony\Component\Validator\Constraints\NotBlank;

// ...

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
          $builder->add(
              'name',
              'text',
              array(
                  'label'       => 'caymland.core.name',
                  'label_attr'  => array('class' => 'control-label'),
                  'attr'        => array(
                      'class'   => 'form-control'
                  ),
                  'constraints' => array(
                      new NotBlank(
                          array(
                              'message' => 'caymland.core.value.required'
                          )
                      )
                  )
              )
          );
    }
    
// ...
```
