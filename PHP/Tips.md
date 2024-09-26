- Use debugbar to check the amount of queries that are being executed in your app. The lesser the queries, the more performant your app gets to be.
- It's always best practice to prevent lazyloading from the AppServiceProvider. You can do so by adding the following to the boot method. The boot method runs after all the dependency for the application has been loaded.
```php
Model::preventLazyLoading()
```

- Adding `proctected guarded = []` allows you to be able to mass assign values in your table. You can add columns you don't want to mass assign into here, and Laravel will automatically prevent mass assignment.
- If you returning just a view for a route (without any data), Laravel already has a shorthand for that.
```php
Route::view('/', 'home')
```
- If you ever want to list all the routes available in your application, you can run the following command
```bash
php artisan route:list --except-vendor
```
