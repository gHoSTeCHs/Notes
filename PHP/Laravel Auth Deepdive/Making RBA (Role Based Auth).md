We first have to to update our users table migrations to include a role column which we would user to ascertain the role assigned to a user, after which we then migrate the database.
```php
$table->string('role')->default('user');
```

We then update the User model to include helper functions that would help us check for the assigned role
```php
class User extends Authenticatable
{
    public function isAdmin()
    {
        return $this->role === 'admin';
    }

    public function isUser()
    {
        return $this->role === 'user';
    }
}
```

After which we then modify the SessionController for role based auth.
```php
public function store(Request $request): Application|Redirector|RedirectResponse
{
    $userAttributes = $request->validate([
        'email' => ['required', 'email'],
        'password' => ['required']
    ]);

    if (!Auth::attempt($userAttributes)) {
        throw ValidationException::withMessages([
            'email' => 'Sorry, those credentials do not match'
        ]);
    }

    $request->session()->regenerate();

    $user = Auth::user();

    // Redirect based on role
    if ($user->role === 'admin') {
        return redirect('/admin/dashboard');
    } elseif ($user->role === 'editor') {
        return redirect('/editor/home');
    }

    return redirect('/user/profile');
}
```

We can further increase the security by making a role middleware to restrict access to certain routes based on the roles.
```sh
php artisan make:middleware RoleMiddleware
```
```php
public function handle($request, Closure $next, $role)
{
    if (!Auth::check() || Auth::user()->role !== $role) {
        return redirect('/');
    }

    return $next($request);
}
```

Register the middleware in `app/Http/Kernel.php`
```php
protected $routeMiddleware = [
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

Then, apply the middleware in your routes:
```php
Route::middleware(['role:admin'])->group(function () {
    Route::get('/admin/dashboard', [AdminController::class, 'index']);
});
```