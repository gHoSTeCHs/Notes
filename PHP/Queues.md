Queues are basically a linear order of instructions which dictate how operations are performed. In Laravel, adding a task to a queue simple means that the task gets to run in the background. Let's say for example a user posts a job, and sending a confirmation email will take like 5 seconds. We would add that to a queue, and show a success message once they post the job. The sending of the mail will now run in the background, and send the mail to the poster without having to lose speed.

Our mail in the jobscontroller currently looks like
```php
Mail::to($job->employer->user)->send(  
    new JobPosted($job)  
);
```

We can simply queue this by making minor adjustments
```php
Mail::to($job->employer->user)->queue(  
    new JobPosted($job)  
);
```

To ensure that this mail gets sent, you have to have active `workers`. These workers are in charge of ensuring that your queues are processed. To call on your queue locally, 
```bash
php artisan queue:work
```
This will start a worker, and if you have any job in the queue, it will trigger and execute the job.

In cases where there are multiple jobs to be performed, you can make use of the dispatch helper function
```php
dispatch(function(){
	
})
%% This is known as a dipatch closure %%
```

Say we want to log something, we can do that as so
```php
Route::get('/test', function (){  
   dispatch(function (){  
       logger('Hello');  
   });  
  
   return 'Done';  
});
```

You can do things like delaying a dispatch closure by adding `->delay()` which takes in the time (in seconds) you want to delay the job for.

#### -> Dedicated Job Classes.
This refers to class that are dedicated to performing a single job. Its more advisable, and scalable to do it this way, as it can be helpful when you are trying to scale. These dedicated classes are usually housed in `app/Jobs`. You can create a new dedicated class by running
```shell
php artisan make:job 
```

Say we want to make a dedicated class to translate a job after it has been posted to other languages and we want to name it `TranslateJob`
```shell
php artisan make:job TranslateJob
```
This creates a TranslateJob file in `app/jobs`, and we can then work from there.

The initial class implementation should look like this
```php
class TranslateJob implements ShouldQueue  
{  
    use Queueable;  
  
    public function __construct()  
    {  
        //  
    }  
  
    public function handle(): void  
    {  
        //  
    }  
}
```

We can handle the process from the handle method. Say we simply want to log something, we can do that as follows
```php
public function handle(): void  
    {  
        logger('Hello from TranslateJob');
    } 
```
And we can then update our routes file
```php
Route::get('/test', function () {  
    TranslateJob::dispatch();  
    return 'Done';  
});
```

Say we want to associate the dispatch with a particular job
```php
Route::get('/test', function () {
	$job = Jobs::first();
    TranslateJob::dispatch($job);  
    return 'Done';  
});
```
Here, we are fetching the first job from the database, and then passing it to the TranslateJob dispatch. This is then accessible from the TranslateJob's class, and we can then get a really good idea of the particular job we are dispatching.
```php
public function __construct(public Jobs $job_listing)  
    {  
        //  
    } 
```

>[!note]
>You should know that the queue:work loads to the memory, and you should restart the queue worker whenever you make an breaking update.

