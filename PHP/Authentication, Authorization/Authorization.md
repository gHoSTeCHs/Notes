This simply refers to the process of restricting access to some actions based on the level of clearance a user have. It might be for deleting a resource, or accessing a path. We also have make to sure that the models are properly linked, as that will allow to create different levels of access.
#### Step 1 -> Inline Auth
In the case of this example application, we can protect some pages by simply checking if a user is authenticated. We can do this by checking the auth state of the user before granting them access to the page.
```php
if(Auth::guest){
	return redirect('/login');
}
```
Based on how you define your relationships, you can also limit access to resources by referencing is the relationship between a user, and a resource is valid
```php
if ($job->employer->user->isNot(Auth::user())){  
    abort(403);  
}
```

#### Step 2 -> Gates
Gates are simply closures that determine if a user is authorized to perform a given action. Typically, gates are defined within the `boot` method of the `App\Providers\AppServiceProvider` class using the `Gate` facade. Gates always receive a user instance as their first argument and may optionally receive additional arguments such as a relevant Eloquent model.

To define a Gate, we first call the gate façade, which accepts the the name of the Gate, and a function which takes in the models you want to reference as arguments.
```php
Gate::define('edit-job', function (User $user, Jobs $job){  
    return $job->employer->user->is($user);  
});
```

^623617

And we can then call the Gate by
```php
Gate::authorize('edit-job', $job)
```

It's best practice to call [[#^623617|this]] from `App\Providers\AppServiceProvider`, as it will always ensure that the gates available from the boot of the application. This will then allow us to further extend our authorization, and in this can we would like to remover the edit button if the current user is not the owner of the job. We can do this using blade directives.
```html
@can('edit-job', $job)  
    <x-button href="/jobs/{{$job['id']}}/edit">Edit</x-button>  
@endcan
```

#### Step 3 -> Middleware Authorization
As of now, our edit method currently looks like this
```php
public function edit(Jobs $job): Application|Factory|View|Redirector|RedirectResponse  
{  
    Gate::authorize('edit-job', $job);  
  
    return view('jobs.edit', ['job' => $job]);  
}
```
While there isn't any much problems with this, the issue arises with redundancy. We will have to call `Gate::authorize('edit-job', $job);` for every of the method, and that can get boring overtime. This is where middleware authorization shines.

To add the middle ware for auth, we simple have to add `->middleware(auth)` to the routes we would want to be protected by the middleware.
```php
Route::resource('jobs', JobController::class)->middleware('auth');
```
This will only access to the route if you are authenticated. The problem now lies in the fact that all of the routes are now protected. You can fix this by specifying the routes that you only want to have this middleware and those that are fit for it
```php
Route::resource('jobs', JobController::class)->only['index','show']; //This will generate only the index, show route, on methods on the controller
Route::resource('jobs', JobController::class)->except['index','show']->middleware('auth'); 
//This will generate all methods except the index and show, and then applies the auth middleware to it.
```

But most times, its best practices to reach out for named routes in situations like this.
```php
Route::get('/jobs', [JobsController::class, 'index']);  
Route::get('/jobs/create', [JobsController::class, 'create']);  
Route::post('/jobs', [JobsController::class, 'store'])->middleware('auth');  
Route::get('/jobs/{job}', [JobsController::class, 'show']);  
  
Route::get('/jobs/{job}/edit', [JobsController::class, 'edit'])  
    ->middleware('auth');  
  
Route::patch('/jobs/{job}', [JobsController::class, 'update']);  
Route::delete('/jobs/{job}', [JobsController::class, 'destroy']);
```

You can reference the gates if you so desire by calling the name of gate, and the name of the action.
```php
Route::get('/jobs/{job}/edit', [JobsController::class, 'edit'])  
    ->middleware('auth', 'can:edit-job,job');
```
You can further tweak the above as follows
```php
Route::get('/jobs/{job}/edit', [JobsController::class, 'edit'])  
    ->middleware
    ->can('edit-job', 'job');
```

#### Step 4 -> Polices
Policies are classes that organize authorization logic around a particular model or resource. For example, if your application is a blog, you may have an `App\Models\Post` model and a corresponding `App\Policies\PostPolicy` to authorize user actions such as creating or updating posts. To get started, you generate a policy from the command line
```php
php artisan make:policy
```
This will prompt you to enter the name of the policy, and the model that it's related to. You can then start making changes based on what you want. In this case, we just want check if a user can edit a job.
The initial structure of a newly generated policy is something like this
```php
<?php  
  
namespace App\Policies;  
  
use App\Models\Jobs;  
use App\Models\User;  
use Illuminate\Auth\Access\Response;  
  
class JobPolicy  
{  
    /**  
	     * Determine whether the user can view any models.     */    public function viewAny(User $user): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can view the model.     */    public function view(User $user, Jobs $jobs): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can create models.     */    public function create(User $user): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can update the model.     */    public function update(User $user, Jobs $jobs): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can delete the model.     */    public function delete(User $user, Jobs $jobs): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can restore the model.     */    public function restore(User $user, Jobs $jobs): bool  
    {  
        //  
    }  
  
    /**  
     * Determine whether the user can permanently delete the model.     */    public function forceDelete(User $user, Jobs $jobs): bool  
    {  
        //  
    }  
}
```
But we only need to edit, so we can refactor it to
```php
class JobPolicy  
{  
   public function edit(User $user, Jobs $job)  
   {  
         
   }  
}
```
We can then add the gate logic here.
```php
public function edit(User $user, Jobs $job)  
{  
    return $job->employer->user->is($user);  
}
```
And the make the necessary updates, starting from the routes
```php
Route::get('/jobs/{job}/edit', [JobsController::class, 'edit'])  
    ->middleware('auth')  
    ->can('edit', 'job');
```
And then the views
```php
@can('edit', $job)  
    <x-button href="/jobs/{{$job['id']}}/edit">Edit</x-button>  
@endcan
```
You should also beware of Laravel's naming conventions, as it can impact whether your app works, or not. The name of the Policy should be the relate model name, plus policy. If you have a model named `Jobs`, the policy should be `JobsPolicy`. 
