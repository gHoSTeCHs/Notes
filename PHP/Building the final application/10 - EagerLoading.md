This is where we try to optimize our queries. Initially we are just make queries for every relationship, but that's not very ideal in a production instance. We should try to make it so that the relationships are loaded with the response that the server returns from the database.
```php
$jobs = Job::latest()->with(['employer', 'tags'])->get()->groupBy('featured');
```
Here, we are returning the jobs, and sorting based on the latest one, and fetching them with the appropriate relationships, and grouping them based on the `featured` attribute. This effectively reduces our queries from 47, to 9.