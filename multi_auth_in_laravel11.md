# Multi auth in Laravel 11 (with Inertia.js and React | session driver)

## Introduction

Let's familiarize ourselves with what is multi-auth in Laravel. At this point you already know how the authentication
system works in Laravel. When you use one of the starter kits for Laravel Breeze (I chose Inertia and React); out of the
box you will get a session based authentication system. The gives you the ability to use authentication on `users`
table (the `User` model).

Now let's say you want to have another type of user, say `Admin`. You don't want to use the same `users` table
for `Admin` because they have different fields and different roles. This is where multi-auth comes in. It lets you have
multiple
authentication systems in your Laravel application. Usually this refers to as having multiple guards.

## Steps

### Step 1: Create a new guard

First thing you need to do is create a new guard. You can do this by editing the `config/auth.php` file. Add a new guard

```php
'guards' => [
    // ..

    // our new guard
    'admin' => [ 
        'driver' => 'session',
        'provider' => 'admins',
    ],
    
    // ..
    
    'provider' => [
        // ..
        
        // our new provider (the model on which the guard will be based)
        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Models\Admin::class,
        ],
        
        // ..
    ],
        
    // ..
    
     'passwords' => [
        // ..

        // our new password reset token table (will use the password_resets table for password reset tokens)
        'admins' => [
            'provider' => 'admins',
            'table' => 'password_reset_tokens',
            'expire' => 60,
            'throttle' => 60,
        ],
    ],
],
```

### Step 2: Create a new model and migration

Create a new model and migration for the `Admin` user. You can do this by running the following command

```bash
php artisan make:model Admin -m
```

The migration can look something like this

```php
// .. 
Schema::create('admins', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamp('email_verified_at')->nullable();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});
// ..
```

Use Authenticatable trait in the `Admin` model, so that we can hook into Laravel's authentication system.

```php
use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
    // ..
}
```

### Step 3: Create new routes for the admin

Create new routes for the admin. You can add this into `web.php` in the `routes` directory.

```php
use Illuminate\Support\Facades\Route;

// Guest routes for the admin 
Route::middleware('guest')
->prefix('admin')
->group(function () {
    // /admin/login [GET]
    Route::get('/login', function () {
        return Inertia::render('Admin/Login'); // shows the login page (duplicate the login page from the user)
    })->name('admin.login');
    
    // /admin/login [POST]
    Route::post('/login', [AdminLoginController::class, 'login']); // handles the login and redirects to the dashboard
    
    // /admin/register [GET]
    Route::get('/register', function () {
        return Inertia::render('Admin/Register'); // shows the register page (duplicate the register page from the user)
    })->name('admin.register');
    
    // /admin/register [POST]
    Route::post('/register', [AdminRegisterController::class, 'register']); // handles the registration and redirects to the dashboard
    
    // /admin/logout [POST]
    Route::post('/logout', [AdminLogoutController::class, 'logout']); // handles the logout and redirects to the login page [See redirection below to set it up correctly]
});

// Protected routes for the admin (/admin/dashboard) [GET]
Route::middleware('auth:admin')
->prefix('admin')
->group(function () {
    Route::get('/dashboard', function () {
        return Inertia::render('Admin/Dashboard');
    })->name('admin.dashboard');
});
```

When you did set it up correctly, using `/admin/register` POST request should create a new `Admin` user in the database
in the `admins` table.

### Step 4: Set up redirection

When you want to enter `/admin` you want to be redirected to `/admin/login` if you are not logged in. This simple route
can be added to the `web.php` file.

```php
// ..

Route::get('/admin', fn () => redirect(route('admin.login')));

// ..
```

When you are logged out you should be redirected to `/admin/login`, for this you need to do a little setup
in `bootstrap/app.php` file.

```php
// ..

    ->withMiddleware(function (Middleware $middleware) {

        // ..
        
        $middleware->redirectGuestsTo(function () {
            if (str_starts_with(request()->path(), 'admin')) {
                return route('login.admin');
            }

            return route('login');
        });
        
        // ..
    })
    
// ..
```

In Laravel 11 the file structure is a bit different. You can find the `app.php` file in the `bootstrap` directory. More
on this in the [Laravel 11 release notes](https://laravel.com/docs/11.x/releases#laravel-11).

So when we log out from as an admin we are unauthenticated which Laravel can detect and we can execute our redirection
logic inside the `redirectGuestsTo` method.

On the flip side, if we are logged in and are on `/admin/dashboard` page; and then we enter `/admin/login` we should be redirected to
`/admin/dashboard`. This can be done by adding a guest middleware for the admin and use it for the login and register.

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class RedirectAdminIfAuthenticated
{
    public function handle(Request $request, Closure $next)
    {
        $adminUser = auth('admin')->user() ?? null; // check if the admin user is logged in

        if ($adminUser) {
            return redirect()->route('dashboard.admin');
        }

        return $next($request);
    }
}
```

One little bonus elegance you do is to set an alias for your middleware in the `bootstrap/app.php` file.

```php

// ..

    ->withMiddlewareAliases([
        'guest.admin' => \App\Http\Middleware\RedirectAdminIfAuthenticated::class,
    ])

// ..

```

Now you can use the `guest.admin` middleware in the `web.php` file.

```php

// ..

Route::middleware('guest.admin') // use the guest.admin middleware
->prefix('admin')
->group(function () {
    // ..
});
```

And that's it! You have a multi-auth system in Laravel 11 with Inertia.js and React. You can now have multiple types of users in your application.

If you have any questions or feedback, feel free to reach out to me on [Twitter](https://twitter.com/brainlet_ali).
