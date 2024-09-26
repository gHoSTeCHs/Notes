The first to step to sending mails is to generate what Laravel will refer to as a mailable.
```shell
php artsian make:mail
```
If you want to send out an email, you would have to fill out the envelope.
```php
public function envelope(): Envelope  
{  
    return new Envelope(  
        subject: 'Job Posted',  
    );  
}
```

You can then configure the views that is responsible for the mail. This view will define the structure and format your mail will be sent with.
```php
public function content(): Content  
{  
    return new Content(  
        view: 'mail.job-posted',  
    );  
}
```

To send a mail, we call the mail facade, and use the send method where we provide it with a new instance of the mail class we would like to send.
```php
Route::get('/test', function () {  
    Mail::to('kingswatter@gmail.com')->send(  
        new JobPosted()  
    );  
    return 'Done';  
});
```

While you can setup the receiving address for the mail from `config/mail.php`, you can also override the default value that is already set in your `.env` file.
```php
public function envelope(): Envelope  
{  
    return new Envelope(  
        from: 'admin@sommy.com',  
        replyTo: 'admin@sommy.com',   
        subject: 'Job Posted'  
    );  
}
```

To send a mail based on action (like a user creating a job), we define it in method responsible for handling the action.
```php
public function store(): Application|Redirector|RedirectResponse  
{  
    request()->validate([  
        'title' => ['required', 'min:3'],  
        'salary' => ['required'],  
    ]);  
  
    $job = Jobs::create([  
        'career' => request('title'),  
        'salary' => request('salary'),  
        'employer_id' => 1  
    ]);  
  
    Mail::to($job->employer->user)->send(  
        new JobPosted($job)  
    );  
  
    return redirect('/jobs');  
}
```
You can then access the data of the the job posted from the email by referencing the Jobs class, and setting the the variable declaration to public in the constructor of the mail class responsible for the job
```php
public function __construct(public Job $job){
	
}
```
This will expose the Jobs class to the view, and you can then use it to customize what your mail is going to look like. You can also protect tag it as protected if there are some properties you don't want exposed to the view. 

> [!tip]  Note !
> It's best practice to add you mail processes to queues, as send a mail takes some time to process. This will allow your app to be as reactive as possible, and faster for users.

