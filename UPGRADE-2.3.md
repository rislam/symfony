﻿UPGRADE FROM 2.2 to 2.3
=======================

Form
----

 * Although this was not officially supported nor documented, it was possible to
   set the option "validation_groups" to false, resulting in the group "Default"
   being validated. Now, if you set "validation_groups" to false, the validation
   of a form will be skipped (except for a few integrity checks on the form).

   If you want to validate a form in group "Default", you should either
   explicitly set "validation_groups" to "Default" or alternatively set it to
   null.

   Before:

   ```
   // equivalent notations for validating in group "Default"
   "validation_groups" => null
   "validation_groups" => "Default"
   "validation_groups" => false

   // notation for skipping validation
   "validation_groups" => array()
   ```

   After:

   ```
   // equivalent notations for validating in group "Default"
   "validation_groups" => null
   "validation_groups" => "Default"

   // equivalent notations for skipping validation
   "validation_groups" => false
   "validation_groups" => array()
   ```
 * The array type hint from DataMapperInterface was removed. You should adapt
   implementations of that interface accordingly.

   Before:

   ```
   use Symfony\Component\Form\DataMapperInterface;

   class MyDataMapper
   {
       public function mapFormsToData(array $forms, $data)
       {
           // ...
       }

       public function mapDataToForms($data, array $forms)
       {
           // ...
       }
   }
   ```

   After:

   ```
   use Symfony\Component\Form\DataMapperInterface;

   class MyDataMapper
   {
       public function mapFormsToData($forms, $data)
       {
           // ...
       }

       public function mapDataToForms($data, $forms)
       {
           // ...
       }
   }
   ```

   Instead of an array, the methods here are now passed a
   RecursiveIteratorIterator containing an InheritDataAwareIterator by default,
   so you don't need to handle forms inheriting their parent data (former
   "virtual forms") in the data mapper anymore.

   Before:

   ```
   use Symfony\Component\Form\Util\VirtualFormAwareIterator;

   public function mapFormsToData(array $forms, $data)
   {
       $iterator = new \RecursiveIteratorIterator(
           new VirtualFormAwareIterator($forms)
       );

       foreach ($iterator as $form) {
           // ...
       }
   }
   ```

   After:

   ```
   public function mapFormsToData($forms, $data)
   {
       foreach ($forms as $form) {
           // ...
       }
   }
   ```

PropertyAccess
--------------

 * PropertyAccessor was changed to continue its search for a property or method
   even if a non-public match was found. This means that the property "author"
   in the following class will now correctly be found:

   ```
   class Article
   {
       public $author;

       private function getAuthor()
       {
           // ...
       }
   }
   ```

   Although this is uncommon, similar cases exist in practice.

   Instead of the PropertyAccessDeniedException that was thrown here, the more
   generic NoSuchPropertyException is thrown now if no public property nor
   method are found by the PropertyAccessor. PropertyAccessDeniedException was
   removed completely.

   Before:

   ```
   use Symfony\Component\PropertyAccess\Exception\PropertyAccessDeniedException;
   use Symfony\Component\PropertyAccess\Exception\NoSuchPropertyException;

   try {
       $value = $accessor->getValue($article, 'author');
   } catch (PropertyAccessDeniedException $e) {
       // Method/property was found but not public
   } catch (NoSuchPropertyException $e) {
       // Method/property was not found
   }
   ```

   After:

   ```
   use Symfony\Component\PropertyAccess\Exception\NoSuchPropertyException;

   try {
       $value = $accessor->getValue($article, 'author');
   } catch (NoSuchPropertyException $e) {
       // Method/property was not found or not public
   }
   ```

DomCrawler
----------

 * `Crawler::each()` and `Crawler::reduce()` now return Crawler instances
   instead of DomElement instances:

   Before:

   ```
   $data = $crawler->each(function ($node, $i) {
       return $node->nodeValue;
   });
   ```

   After:

   ```
   $data = $crawler->each(function ($crawler, $i) {
       return $crawler->text();
   });
   ```

Console
-------

 * New verbosity levels have been added, therefore if you used to do check
   the output verbosity level directly for VERBOSITY_VERBOSE you probably
   want to update it to a greater than comparison:

   Before:

   ```
   if (OutputInterface::VERBOSITY_VERBOSE === $output->getVerbosity()) { ... }
   ```

   After:

   ```
   if (OutputInterface::VERBOSITY_VERBOSE <= $output->getVerbosity()) { ... }
   ```
