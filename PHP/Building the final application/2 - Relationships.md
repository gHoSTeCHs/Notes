We know that an employer belongs to a user, so in the employer's table,
```php
public function user(): BelongsTo  
{  
    return $this->belongsTo(User::class);  
}
```

For the inverse relationship, we know that a user should equate to one employer, so the [[Eloquent Relatioships|relationship]] is a `hasone`
```php
public function employer(): HasOne
{
	return $this->hasone(Employer::class);
}
```

We also know that a Job belongs to an employer, so in the Job class
```php
public function employer(): BelongsTo  
{  
    return $this->belongsTo(Employer::class);  
}
```