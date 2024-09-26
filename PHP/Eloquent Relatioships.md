```php
$this->belongsTo(Employer::class)
```
This is used to a one is to one relationship between classes. In this case, we are saying that one job belongs to an employer.

```Php
return $this->hasMany(Jobs::class);
```
This is used to say that a resource has many other resource attributed to it. In this case, we are saying that a Employer can have many Jobs in the system.

```php
return $this->belongsToMany(Tag::class, 'job_tag', foreignPivotKey: 'job_listing_id')
```
This is used to say that a resource can belong to multiple other elements. Here we are saying that a Job can have many tags associated to it.