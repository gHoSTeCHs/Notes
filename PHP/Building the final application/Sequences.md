This is basically having a set values that you eloquent model can iterate over. You can define multiple sequences, and then have your factory assign those values in a sequential order.
This is what the Jobseeder looks like at the moment
```php
public function run(): void  
{  
    $tags = Tag::factory(3)->create();  
  
    Job::factory(20)->hasAttached($tags)->create();  
}
```

To add a sequence, we call a new instance of a sequence
```php

```
