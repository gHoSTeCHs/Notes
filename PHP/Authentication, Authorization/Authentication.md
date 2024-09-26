This is one of the most risky, and hard things to do when it comes to building applications. This is because of the level of care that is required to ensure that your app is safe from all kinds of attack. Laravel tries to make implementing this as easy as possible. These are the steps to take to make a basic authentication system in Laravel.
#### 1. Create the register route, and controller

^b09a44

   You first create the register route and controller which would be in charge of registering users into the system
```php
php artisan make:controller RegisteredUserController
```
 This is creates the controller, and you can add this to your routes file. You create the route access point, and it should look something like this
 ```php
 Route::get('/register', [RegisteredUserController::class, 'create']);
```

#### 2. Create the route group
Following Laravel conventions, its's best practice to group your auth paths.
```js
auth -> folder
	register -> route
	login -> route
```

#### 3. Create the login controller
You can create the login controller using the instructions from [[#^b09a44|step 1]].  As of this moment, this is what the files relevant to this authentication looks like 

*web.php*
```php
// Register User Route  
Route::get('/register', [RegisteredUserController::class, 'create']);  
Route::post('/register', [RegisteredUserController::class, 'store']);  
// Login Route  
Route::get('/login', [SessionController::class, 'create']);  
Route::post('/login', [SessionController::class, 'store']);
```

*RegisteredUserController*
```php
class RegisteredUserController extends Controller  
{  
    //  
    public function create(): Factory|View|Application  
    {  
        return view('auth.register');  
    }  
      
    public function store()  
    {  
       dd(request()->all());  
    }  
}
```

*SessionController*
```php
class SessionController extends Controller  
{  
    //  
    public function create(): Factory|View|Application  
    {  
        return view('auth.login');  
    }  
    public function store()  
    {  
        dd(request()->all());  
    }  
}
```

#### 4. Updating the Registration Controller
The following are the steps to usually take to create a user in your system
1. Validate the data the user sends
2. Create the user in the database
3. Login the user into the system
4. Redirect
You validate the data from the user with the validate method
```php
$validatedAttributes = request()->validate([  
    'first_name' => ['required'],  
    'last_name' => ['required'],  
    'email' => ['required', 'email', 'max:256'],  
    'password' => ['required', Password::min(6), 'confirmed']  
]);
```
If the structure of the form matches the one in the database, you can create the user directly using
```php
User::create($validatedAttributes)
```
And to login the user, Laravel ships with an Auth class with has method in relation to authentication. To login a user, you call the following method
```php
Auth::login($user)
```
After which you redirect the user to the place you want. The overall structure of the RegisteredController will look like this at the moment
```php
class RegisteredUserController extends Controller  
{  
    //  
    public function create(): Factory|View|Application  
    {  
        return view('auth.register');  
    }  
  
    public function store(): Application|Redirector|RedirectResponse  
    {  
        $validatedAttributes = request()->validate([  
            'first_name' => ['required'],  
            'last_name' => ['required'],  
            'email' => ['required', 'email', 'max:256'],  
            'password' => ['required', Password::min(6), 'confirmed']  
        ]);  
  
        $user = User::create($validatedAttributes);  
        Auth::login($user);  
  
        return redirect('/jobs');  
    }  
}
```

#### 5. Login the user
Logging in the user is also quite straightforward. The following are the steps we take when validating the user
1. Validate the data from the user
2. Check if the user exists in the system
3. Attempt to login the user
4. Redirect the user
You can validate the user with
```php
$attributes = request->validate([
	'email' => ['required', 'email'],
	'password' => ['required],
])
```
You then attempt to login the user with values provided. It's always best practice to handle this as a validation field, or something of that nature. To attempt to login the user, we use the attributes sent from the user form, to login.
```php
Auth::attempt($attributes)
```

^65d822

We then generate a session to enable the user login.
```php
request()->session()->regenerate()
```
We then redirect the user to our screen of choice
```php
return redirect('/jobs)
```

This is what our login method looks like at the moment
```php
public function store(): Application|Redirector|RedirectResponse  
{  
    $attributes = request()->validate([  
        'email' => ['required', 'email'],  
        'password' => ['required']  
    ]);  
  
    Auth::attempt($attributes);  
  
    request()->session()->regenerate();  
  
    return redirect('/jobs');  
}
```

But there is a problem with [[#^65d822|this]]. It will redirect the user to the jobs page even if the the attempt was unsuccessful. We handle this by throwing a validation exception if the credentials are not valid.
```php
if(Auth::attempt($attributes)){
	throw ValidationException::withMessages([
		'email'=> 'Sorry, those credentials do not match'
	])
}
```
This will prevent the code to generate a session and redirect from running.

Adding `:value="old('email')"` to the input box responsible for the email will ensure that form is still populated by the email even after it has been submitted.