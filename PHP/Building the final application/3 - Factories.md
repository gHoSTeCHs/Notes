We write this to determine how our databases should be populated should we want to make use of [[Database Seeding|seeders]], of [[Tinker Commands|tinker]]. 

We first make the employer factory
```php
public function definition(): array
    {
        return [
            'name' => fake()->company(),
            'logo' => fake()->imageUrl(),
            'user_id' => User::factory(),
        ];
    }
```

then the job factory
```php
public function definition(): array
    {
        return [
            'employer_id' => Employer::factory(),
            'title' => fake()->jobTitle,
            'salary' => fake()->randomElement(['$50,000 USD', '$90,000 USD', '$150,000 USD']),
            'location' => 'Remote',
            'schedule' => 'Full Time',
            'url' => fake()->url,
            'featured' => false,
        ];
    }
```