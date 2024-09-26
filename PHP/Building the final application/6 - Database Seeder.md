For our Job seeder
```php
public function run(): void  
{  
    //  
    $tags = Tag::factory(3)->create();  
  
    Job::factory(20)->hasAttached($tags)->create();  
}
```
which we can then call from the database seeder
```php
$this->call(JobSeeder::class);
```
