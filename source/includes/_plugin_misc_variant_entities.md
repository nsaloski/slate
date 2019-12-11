### Implementing Variant (A/B Test) Support to Entities

Caymland has some helper methods with adding support for creating variants of a given entity. This becomes particularly useful for A/B testing.

#### \Caymland\CoreBundle\Entity\VariantInterface

This Entity interface ensures that everything is needed in order for Caymland to handle the variants correctly for an entity.

#### \Caymland\CoreBundle\Entity\VariantEntityTrait

This trait provides properties needed to define an Entity's relationship to other items. In the Entity's `loadMetadata()` method, be sure to call `$this->addVariantMetadata()`.

#### \Caymland\CoreBundle\VariantModelTrait

This trait provides the methods `preVariantSaveEntity()`, `postVariantSaveEntity()` and `convertVariant()`. `preVariantSaveEntity()` should be executed prior to `saveEntity` then `postVariantSaveEntity()`. See example.

```php
<?php
// plugins/HelloWorldBundle/Model/WorldModel.php

// Reset a/b test if applicable
$variantStartDate = new \DateTime();
// setVariantHits is the stat tracker properties for this variant
$resetVariants    = $this->preVariantSaveEntity($entity, ['setVariantHits'], $variantStartDate);

parent::saveEntity($entity, $unlock);

$this->postVariantSaveEntity($entity, $resetVariants, $entity->getRelatedEntityIds(), $variantStartDate);
````
<div class="clear-right"></div>

#### \Caymland\CoreBundle\Doctrine\VariantMigrationTrait

To ease the generation of schema to match the Entity, use this trait then execute `$this->addVariantSchema()`.

#### Translated Entity Form

Add `variantParent` field's like the code example. In the example, the `variantParent` value is set in the controller due to a `Add A/B Test` button is clicked. The specific use for the plugin may require a select list rather than a hidden field. Change this to meet the code's needs.

```php
<?php 
// plugins/HelloWorldPlugin/Form/Type/WorldType.php

$transformer = new \Caymland\CoreBundle\Form\Transformer\IdToEntityModelTransformer($this->em, 'HelloWorldBundle:World');
$builder->add(
   $builder->create(
       'variantParent',
       'hidden'
   )->addModelTransformer($transformer)
);
```
