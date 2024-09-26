We first take of the routes responsible for this action. To get to the job creation page, you have to hit the `jobs/create` route, which will require the auth middleware
```php
Route::get('/jobs/create', [JobController::class, 'create'])->middleware('auth');
```

and to add the job to the database, you will have to send a post request to `/jobs`
```php
Route::post('/jobs', [JobController::class, 'store'])->middleware('auth');
```

We can then define the class responsible for various actions on the job. To store a job, we first validate the inputs provided
```php
$attributes = $request->validate([  
    'title' => ['required'],  
    'salary' => ['required'],  
    'location' => ['required'],  
    'schedule' => ['required', Rule::in(['Part Time', 'Full Time'])],  
    'url' => ['required', 'active_url'],  
    'tags' => ['nullable'],  
]);
```
This is not the full list of the fields accepted by the form, as some of them require some extra magic to work.

For the `featured`, we will try to see if the featured is true, or false
```php
$attributes['featured'] = $request->has('featured');
```
After which will then work on the tags provided (if any was given).
```php
if ($attributes['tags'] ?? false) {
            foreach (explode(',', $attributes['tags']) as $tag) {
                $job->tag($tag);
            }
        }
```
We are checking to to see if the attributes tag is false, and append where neccesary.