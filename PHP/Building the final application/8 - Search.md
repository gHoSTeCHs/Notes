We will try to implement a simple search system for searching for jobs. Just something that will search the database for a job title, based on the input provided from the search bar.
%% Form for the search input %%
```html
<x-forms.form action="/search" class="mt-6">  
    <x-forms.input :label="false" name="q" placeholder="Web Developer..." />  
</x-forms.form>
```

If the action, of function of a controller is particulatly complex, you can reach for invokable controllers. This are controllers that typically perform a single action, and our search functionality in this case is a good example
```php
Route::get('/search',[SearchController::class])
```
```shell
php artisan make:controller SearchController
```

To search for a job where the the title is like the search query
```php
public function __invoke(): Factory|View|Application  
{  
    $jobs = Job::where('title', 'LIKE', '%' . request('q') . '%')->get();  
  
    return view('results', ['jobs' => $jobs]);  
}
```

To also make a search of this nature on the tags,
```php
public function __invoke(Tag $tag): Factory|View|Application  
{  
    // TODO: Implement __invoke() method.  
    return view('results', ['jobs' => $tag->jobs]);  
}
```