We first setup our endpoints
```php
Route::get('/register', [RegisteredUserController::class, 'create']);  
Route::post('/register', [RegisteredUserController::class, 'store']);  
  
Route::get('/login', [SessionController::class, 'create']);  
Route::post('/login', [SessionController::class, 'store']);  
Route::delete('/logout', [SessionController::class, 'destroy']);
```
and then make the relevant controllers with resources.
```shell
php artisan make:controller SessionController --resourse
```

#### 1 - Registration
We first on the RegisterUserController. Validating the input is the first thing to do here.
%% User Section %%
```php
$request->validate([
            'name' => ['required'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'confirmed', Password::min(6)],
        ]);
```
%% Employer Section %%
```php
 $request->validate([
            'employer' => ['required'],
            'logo' => ['required', File::types(['png', 'jpg', 'webp'])],
        ]);
```

We are separating them in other to allow for storing into different tables, as the employers table, and user table are different.
```php
$userAttributes = $request->validate([
	'name' => ['required'],
	'email' => ['required', 'email', 'unique:users,email'],
	'password' => ['required', 'confirmed', Password::min(6)],
]);

$employerAttributes = $request->validate([
	'employer' => ['required'],
	'logo' => ['required', File::types(['png', 'jpg', 'webp'])],
]);
```

This will then allow us to pass in the values into the different classes responsible for the operations
```php
$user = User::create($userAttributes);

$logoPath = $request->logo->store('logos');

$user->employer()->create([
	'name' => $employerAttributes['employer'],
	'logo' => $logoPath,
]);
```

>[!note]
>You can see how we are trying to create the user. This is because of how the Employer relates to a user.

>[!note]
>You can also see how we are trying to store the logo. We are storing it in the public directory of the application which Laravel provides support for out of the box. It provides multiple storage options like local, stp, public, and s3. You can find all this in `filesystems.php` under `app/config`, which is where you configure how you want to handle image uploads.

Overall, the registration controller is going to look something like this
```php
class RegisteredUserController extends Controller  
{  
  
    public function create(): Factory|View|Application  
    {  
        return view('auth.register');  
    }  
  
    /**  
     * Store a newly created resource in storage.     */  
    public function store(Request $request): Application|Redirector|RedirectResponse  
    {  
        $userAttributes = $request->validate([  
            'name' => ['required'],  
            'email' => ['required', 'email', 'unique:users,email'],  
            'password' => ['required', 'confirmed', Password::min(6)],  
        ]);  
  
        $employerAttributes = $request->validate([  
            'employer' => ['required'],  
            'logo' => ['required', File::types(['png', 'jpg', 'webp'])],  
        ]);  
  
        $user = User::create($userAttributes);  
  
        $logoPath = $request->logo->store('logos');  
  
        $user->employer()->create([  
            'name' => $employerAttributes['employer'],  
            'logo' => $logoPath,  
        ]);  
  
        Auth::login($user);  
  
        return redirect('/');  
    }  
}
```

#### 2 - Login
Logging a user is quite easy. We simply validate the provided credentials, check if they exist in the database, regenerate the session (for [[Best Practices|best practices]]), and then route the user to where we want them.
```php
class SessionController extends Controller  
{  
    /**  
     * Show the form for creating a new resource.     */    public function create(): Factory|View|Application  
    {  
        return view('auth.login');  
    }  
  
    /**  
     * Store a newly created resource in storage.     * @throws ValidationException  
     */    public function store(Request $request): Application|Redirector|RedirectResponse  
    {  
        $attributes = request()->validate([  
            'email' => ['required', 'email'],  
            'password' => ['required'],  
        ]);  
  
        if (!Auth::attempt($attributes)) {  
            throw ValidationException::withMessages([  
                'email' => 'Sorry, those credentials do not match.',  
            ]);  
        }  
  
        request()->session()->regenerate();  
  
        return redirect('/');  
    }  
  
    /**  
     * Remove the specified resource from storage.     */    public function destroy(string $id): Application|Redirector|RedirectResponse  
    {  
        Auth::logout();  
  
        return redirect('/');  
    }  
}
```