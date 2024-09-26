This is the next phase of building the application. We first refine our routes file to work with the Jobs controller to return an index method when we visit the homepage
```php
Route::get('/', [JobController::class, 'index']);
```

We then update the index method to function the way we desire it to
```php
public function index(): Factory|View|Application  
{  
    return view('jobs.index', [  
        'jobs' => Job::all(),  
        'tags' => Tag::all()  
    ]);  
}
```
Here, we are returning all the jobs, and tags, hence the all method.