# To use object keys as variables we need to create a variable from string. 

```php
<?php 
  // create an object with some key value data
  $myObject = [
    'name' => 'my name',
    'email' => 'my email'
  ];
  // loop object data
  foreach($myObject as $key => $value){
    // create a variable from key name, then assign to that variable the key value
    $$key = $value;
  }
  
  echo $name; // my name
?>
```
