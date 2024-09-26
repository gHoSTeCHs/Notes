This simply refers to using test cases to build out your applications. You write a test case, and then iterate on the errors it provides you with. While this is quite slow, it can help with early error detections.
To create a test in a Laravel application, we can do that from `phpunit.xml`. 
```xml
<php>  
    <env name="APP_ENV" value="testing"/>  
    <env name="APP_MAINTENANCE_DRIVER" value="file"/>  
    <env name="BCRYPT_ROUNDS" value="4"/>  
    <env name="CACHE_STORE" value="array"/>  
     <env name="DB_CONNECTION" value="sqlite"/>   
<env name="DB_DATABASE" value=":memory:"/>   
<env name="MAIL_MAILER" value="array"/>  
    <env name="PULSE_ENABLED" value="false"/>  
    <env name="QUEUE_CONNECTION" value="sync"/>  
    <env name="SESSION_DRIVER" value="array"/>  
    <env name="TELESCOPE_ENABLED" value="false"/>  
</php>
```

You can then run
```shell
php artisan make:test
```
which will then prompt you to choose between two different types of test.
1. Feature Test
2. Unit Test
`Feature Test` can be considered testing a wild spectrum of your application, while `Unit Test` refers to a more narrow and focused test.

To run a test, 
```shell
php artisan test
```

Say we want to check if a job belongs an employer
```php
test('belongs to an employer', function () {  
      
});
```
we can either have the prefix as `test`, or `it` based on the usecase. 
```php
it('belongs to an employer', function () {  
      
});
```

There are triple As when it comes to testing
1. Arrange
2. Act
3. Assert
We begin by arranging the world we want the test to live in. We then act upon the arrangement, after which we say what we expect to happen after that assertion.

```php
test('belongs to an employer', function () {  
    $employer = Employer::factory()->create();  
    $job = Job::factory()->create([  
        'employer_id' => $employer->id  
    ]);  
  
    expect($job->employer->is($employer))->toBeTrue();  
});
```
Here, we are creating a user and employer factory, where we then check if the employer id from the job is the same.

#### Using TDD
We will now try to understand how TDD works by using the tags model. We first write the test of how we want the model to behave.
```php
it('can have tags', function () {
    $job = Job::factory()->create();

    $job->tag('Frontend');

    expect($job->tags)->toHaveCount(1);
});
```
Here we are creating a Job with the factory, and then tag it with `frontend`. We are then expecting the job to have at least one tag. Keep in mind that at this stage of development, we haven't created the tags table, so this is going to fail. We can then use the errors we got to build up.

Running the test, the first error we see is that we are trying to call to a method that doesn't exist on the the Jobs class
```shell
it can have tags                                                                                                                                   0.09s  
  ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────  
   FAILED  Tests\Unit\JobTest > it can have tags                                                                                     BadMethodCallException   
  Call to undefined method App\Models\Job::tag()

  at vendor\laravel\framework\src\Illuminate\Support\Traits\ForwardsCalls.php:67
     63▕      * @throws \BadMethodCallException
     64▕      */
     65▕     protected static function throwBadMethodCallException($method)
     66▕     {
  ➜  67▕         throw new BadMethodCallException(sprintf(
     68▕             'Call to undefined method %s::%s()', static::class, $method
     69▕         ));
     70▕     }
     71▕ }

  1   vendor\laravel\framework\src\Illuminate\Support\Traits\ForwardsCalls.php:67
  2   vendor\laravel\framework\src\Illuminate\Support\Traits\ForwardsCalls.php:36

```

We can then go ahead to create the method in the Job class which will still return an error. We can then go a head and create the Tag model with a factory, and a migration
```shell
php artisan make:model Tag -fm
```

and the migrations will look something like this
```php
public function up(): void  
{  
    Schema::create('tags', function (Blueprint $table) {  
        $table->id();  
        $table->string('name')->unique();  
        $table->timestamps();  
    });  
}
```

and the Tag factory
```php
public function definition(): array  
{  
    return [  
        'name' => fake()->unique()->word,  
    ];  
}
```

We can then set up the relationships, as that's what the current error is telling us. We know that a Job can have many tags, so
```php
public function tags()
{
	return $this->belongsToMany(Tag::class);
}
```
and we will set up a pivot table for this.

```shell
php artisan make:migration create_job_tag_table
```
and set up the constraints
```php
Schema::create('job_tag', function (Blueprint $table) {
            $table->id();
            $table->foreignIdFor(\App\Models\Job::class)->constrained()->cascadeOnDelete();
            $table->foreignIdFor(\App\Models\Tag::class)->constrained()->cascadeOnDelete();
            $table->timestamps();
        });
    }
```

Running the test will still return an error
```shell
 FAILED  Tests\Unit\JobTest > it can have tags
  Failed asserting that actual size 0 matches expected size 1.
```

and then set up the remaining relationships. The Job class should now be looking something like this
```php
class Job extends Model  
{  
    use HasFactory;  
  
    public function tag(string $name): void  
    {  
        $tag = Tag::firstOrCreate(['name' => $name]);  
  
        $this->tags()->attach($tag);  
    }  
  
    public function tags(): BelongsToMany  
    {  
        return $this->belongsToMany(Tag::class);  
    }  
  
    public function employer(): BelongsTo  
    {  
        return $this->belongsTo(Employer::class);  
    }  
}
```
 Which will then enable all the tests to be passed.