This is refers to the process of taking all your assets, like CSS, JS, images and bundling in such a manner that they are ready for production, and are optimized. Vite is one of the most popular asset bundler out there, and Laravel has support for it out of the box.

We first install the packages from npm 
```bash
npm install
```
This will install all the necessary packages
```json
{  
    "private": true,  
    "type": "module",  
    "scripts": {  
        "dev": "vite",  
        "build": "vite build"  
    },  
    "devDependencies": {  
        "axios": "^1.6.4",  
        "laravel-vite-plugin": "^1.0",  
        "vite": "^5.0"  
    }  
}
```

We can then reference the resources by using the `@vite()` blade directive to reference the file we want to import. Importing the CSS file will look something like this `@vite(['resources/css/app.css'])`.

#### Hot Reloading
This simply means instant reloading, or updates without losing the application state. Running `npm run dev` will start this, and bootstrap your app for reactive changes. You can Vite import your CSS from the config if your a building a traditional server side application, it would be best practice to import it from the JavaScript if you are building a SPA