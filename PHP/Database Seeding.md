Seeding simply refers to the process of populating your database tables with random data during development. This is basically like calling factories from `tinker`, but this process is faster. Refreshing your your database and seeding it (if this has already been configured) can be as easy as
```php
php artisan migrate:fresh --seed
```
If you just need to seed the database, you can do that as so
```php
php artisan db:seed
```
To create a new seeder, you can run 
```php
php artisan make:seeder
```
