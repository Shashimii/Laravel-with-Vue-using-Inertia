# Laravel + Inertia + Vue
Guide how to Setup Laravel with Vue using Inertia on Laravel 11.x

## Laravel 
#### Create a new Laravel App normally
```
    laravel new
```

## Inertia
#### Install Dependencies
```
    composer require inertiajs/inertia-laravel
```
It uses the composer package manager to install Inertia dependencies.

#### Root Template
- Setup the root template that will be loaded on the first page visit to your application. 
- This will be used to load your site assets `(CSS and JavaScript)` 
- Contains a root `<div>` in which to boot your JavaScript application.
```
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
        @vite('resources/js/app.js')
        @inertiaHead
    </head>
    <body>
        @inertia
    </body>
    </html>
```
- On `views` folder replace the `welcome.blade.php` with `app.blade.php` then paste the template.

#### Middleware
- setup the Inertia middleware 
- Publish the HandleInertiaRequests middleware to your application 
- using the following Artisan command.

```
    php artisan inertia:middleware
```
- On application root directory it should have the `app\Middleware\HandleInertiaRequests.php`
- Laravel 11.x does not have the default Middleware folder anymore. 
- this should be created when you already Published the Middleware using the following Artisan command.

#### Append the Middleware
- Append the HandleInertiaRequests middleware to the web middleware group in your application's `bootstrap/app.php` file.

```
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use App\Http\Middleware\HandleInertiaRequests;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->web(append: [
            HandleInertiaRequests::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();

```
- `app.php` should look like this after Appending.

#### Make the Pages folder
- On `resources/js` create a `Pages` folder.
- This is where you create your `.vue` pages like `welcome.vue`

## Vue
#### Install Dependencies
- Install the Inertia client-side adapter corresponding to your framework of choice which is `Vue.js`.

```
    npm install @inertiajs/vue3
```
#### Initialize the Inertia app
- Remove the `js/bootstrap.js`
- On `js/app.js` replace the contents with this.

```
    import { createApp, h } from 'vue'
    import { createInertiaApp } from '@inertiajs/vue3'

    createInertiaApp({
    resolve: name => {
        const pages = import.meta.glob('./Pages/**/*.vue', { eager: true })
        return pages[`./Pages/${name}.vue`]
    },
    setup({ el, App, props, plugin }) {
        createApp({ render: () => h(App, props) })
        .use(plugin)
        .mount(el)
    },
    })
```
- The setup callback receives everything necessary to initialize the client-side framework.
- Including the root Inertia App component.

#### Install Vue 3 and the Vite plugin for Vue:
```
npm install vue@3 @vitejs/plugin-vue
```

#### Initialize Vite
- On `vite.config.js` replace the contents with this.
```
    import { defineConfig } from 'vite';
    import laravel from 'laravel-vite-plugin';
    import vue from '@vitejs/plugin-vue';

    export default defineConfig({
        plugins: [
            laravel({
                input: ['resources/css/app.css', 'resources/js/app.js'],
                refresh: true,
            }),
            vue(3)
        ],
    });

```
- Vite has replaced Laravel Mix in new Laravel installations.

## Create Samples
- Sample Routes with Data being Passed as props
`web.php`

```
    <?php

    use Illuminate\Support\Facades\Route;
    use Inertia\Inertia;

    Route::get('/', function () {
        return Inertia::render('welcome', [
                'name' => 'Shashimii',
                'frameworks' => [
                    'Laravel', 'Inertia', 'Vue'
                ]
            ]
        );
    });
```
- Sample Vue Page
`resources/js/Pages/welcome.vue`

```
<template>
    <div>
      <h1>Welcome to Inertia.js with Vue.js!</h1>
      <h3>This is a Single String Data</h3>
      <h3>{{ name }}</h3>
      <h3>This is Array with a Length of 2</h3>
      <ul>
        <li v-for="framework in frameworks" :key="framework">
          {{ framework }}
        </li>
      </ul>
      <h3>Passed by Inertia from Laravel Routes to Vue Props</h3>
    </div>
</template>
  
<script>
    export default {
        props: {
          name: String,
          frameworks: Array,
        }
    };
</script>

<style scoped>
    h1 {
        font-size: 2em;
        color: #333;
    }
</style>
```
- It uses the Data Passed as props

## Run the Application
- Laravel
```
    php artisan serve
```
- Vue

```
    npm run dev
```
- Visit http://localhost:8000 in your browser, and you should see your Vue component rendered by Inertia.js.

## Version
| Framework | Version  |
| :-------- | :------- |
| `Laravel` | `11.x`   | 
| `Inertia` | `0.11.0` |
| `Vue`     | `3`      | 

