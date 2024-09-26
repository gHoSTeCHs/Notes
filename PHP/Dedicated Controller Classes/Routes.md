It's always a good idea to have dedicated controller classes for your routes. This will enable you to keep your routes file as clean as possible as your project grows. This will allow you to group your routes into sections, and based on what each of them do. Its also good practice to keep to how Laravel specifies naming this routes, and views, as it makes your work easier.
```PHP
index -> entry to the route group
show -> for displaying a resourse in particular
create -> for creating a new resource
store -> 
edit -> for editing the resource
update -> for updating the resource
destroy -> for deleting or destroying the resource
```

You can create a dedicated controller by running
```bash
php artisan make:controller
```
This will ask you a bunch of questions, and you can create the controller based on your needs. This is how your controller is going at the base level
```php
class JobsController extends Controller  
{  
    public function index(){  
        dd('Hello');  
    }  
    public function create(){}  
    public function store(){}  
    public function show(){}  
    public function edit(){}  
    public function update(){}  
    public function destroy(){}  
}
```
This will enable our routes file to go from lookin like this -> 
```php
<?php

use Illuminate\Support\Facades\Route;
use App\Models\Jobs;


Route::get('/', function () {
    return view('home');
});

Route::get('/jobs', function () {
    $jobs = Jobs::with('employer')->latest()->cursorPaginate(10);
    return view('jobs.index', [
        'jobs' => $jobs
    ]);
});

Route::post('/jobs', function () {
    request()->validate([
        'title' => ['required', 'min:3'],
        'salary' => ['required'],
    ]);

    Jobs::create([
        'career' => request('title'),
        'salary' => request('salary'),
        'employer_id' => 1
    ]);

    return redirect('/jobs');
});

Route::get('/create', function () {
    return view('jobs.create');
});

Route::get('/jobs/{id}', function ($id) {
    $job = Jobs::findOrFail($id);
    return view('jobs.show', ['job' => $job]);
});

Route::get('/jobs/{id}/edit', function ($id) {
    $job = Jobs::findOrFail($id);
    return view('jobs.edit', ['job' => $job]);
});

Route::patch('/jobs/{id}', function ($id) {
    // validate fields
    request()->validate([
        'title' => ['required', 'min:3'],
        'salary' => ['required'],
    ]);
    // fetch job from db
    $job = Jobs::findOrFail($id);
    // Update job
    $job->update([
        "career" => request('title'),
        'salary' => request('salary')
    ]);
// Redirect to job
    return redirect('/jobs/'. $job->id);
});

// Delete
Route::delete('/jobs/{id}', function ($id) {
    Jobs::findOrFail($id)->delete();
    return redirect('/jobs');
});

Route::get('/contact', function () {
    return view('contact');
});
```
to looking like this
```php
<?php  
  
use App\Http\Controllers\JobsController;  
use Illuminate\Support\Facades\Route;  
use App\Models\Jobs;  
  
  
Route::get('/', function () {  
    return view('home');  
});  
  
Route::get('/jobs', [JobsController::class, 'index']);  
Route::get('/create', [JobsController::class, 'create']);  
Route::post('/jobs', [JobsController::class, 'store']);  
Route::get('/jobs/{job}', [JobsController::class, 'show'] );  
Route::get('/jobs/{job}/edit', [JobsController::class, 'edit']);  
Route::patch('/jobs/{job}', [JobsController::class, 'update']);  
Route::delete('/jobs/{job}', [JobsController::class, 'destroy']);  
  
Route::get('/contact', function () {  
    return view('contact');  
});
```

^154232

And this is only the first stage.
### Route Groups
If you have a controller with multiple routes, you can place them in route groups to effectively clean things up. You can take your routes file from looking like [[#^154232|this]], to 
```php
Route::controller(JobsController::class)->group(function (){  
    Route::get('/jobs', 'index');  
    Route::get('/create', 'create');  
    Route::post('/jobs',  'store');  
    Route::get('/jobs/{job}', 'show' );  
    Route::get('/jobs/{job}/edit', 'edit');  
    Route::patch('/jobs/{job}', 'update');  
    Route::delete('/jobs/{job}', 'destroy');  
});
```

^d043d9

### Route Resource
This is where the importance of naming conventions are emphasized. We can further shrink down the size of [[#^d043d9| our routes file]] by creating a resource group.
```php
Route::resource('name', controller);
```
In this case, the name of our resource is jobs, and controller is JobsController
```php
Route::resource('jobs', JobsController);
```

^92781b

This will effectively reduce the size of route file, and if we followed the proper naming conventions, all things should work as intended. This can also be extended. In the case you want to generate only specific resources, or not generate some, [[#^92781b|this]] can also be extend to accommodate that. You simply add an array and specify what you want.
```php
Route::resource('jobs', JobsController, [
	'except' => ['edit'] -> this will generate all resources but edit 
]);
```

```php
Route::resource('jobs', JobsController, [
	'only' => ['edit'] -> this will generate only the edit resource
]);
```
