This is usually the first step. We make migrations for our models, and we can then define the relationships between them. Since we are building the a Job board, we first create the employers, and the related models
```php
php artisan make:model Employer
```

We then define how the Employer's table is relate to other tables (i.e the relationships it has with other relevant tables).
```php
Schema::create('employers', function (Blueprint $table) {  
    $table->id();  
    $table->foreignIdFor(User::class);  
    $table->string('name');  
    $table->string('logo');  
    $table->timestamps();  
});
```

Then run the necessary migrations to create the the database table.
```shell
php artisan migrate
```

For this project, we will change the default table names that Laravel ships with for queues. This is because they are going to interfere the naming of our job models. We simply just add queued_ to the beginning of the table names.

We can then create the Job model and other related classes 
```shell
php artisan make:model Job --all
```
This creates all the files for Job class, and this is what our Job migration is going to look like
```php
Schema::create('jobs', function (Blueprint $table) {  
    $table->id();  
    $table->foreignIdFor(Employer::class);  
    $table->string('title');  
    $table->string('salary');  
    $table->string('location');  
    $table->string('schedule')->default('Full Time');  
    $table->string('url');  
    $table->boolean('featured')->default(false);  
    $table->timestamps();  
});
```

We can then set up our [[2 - Relationships|relationships]]. 