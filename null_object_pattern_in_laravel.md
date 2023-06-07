## Null Object Pattern

### Introduction

Null Object Pattern in Programming is a design pattern that is used to avoid
null references by providing a default object. 

### Null Object Pattern in Laravel

Let's explore this pattern in Laravel. We will use a simple example to explain
the concept.

Assume we have a `User` model and a `Post` model having a one-to-many
relationship setup like this:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function posts()
    {
        return $this->hasMany(Post::class);
    }
}
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

In our controller after fetching a post, we can load the user relationship like
this and send it to the view:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class PostController extends Controller
{
    public function show(Post $post)
    {
        $post = $post->load('user');
        
        return view('post.show', ['post' => $post]);
    }
}
```

In our blade file, we can access the user's name like this:

```php
// post.show.blade.php
// ...
{{ $post->user->name }}
// ...
```

The problem with this approach is that if the user happens to be null, we will
get an error like this:

```bash
Trying to get property 'name' of non-object
```

> This is the most common error in Laravel and programming in general. The
> traditional way to solve this problem is to use conditional statements to
> check
> for null safety before accessing the property.

So, in our blade file, we can do something like this to avoid the error:

```php
// post.show.blade.php
// ...
@if($post->user)
    {{ $post->user->name }}
@endif
// ...
```

This approach works, but it is not the best way to solve this problem.

### The Null Object Pattern in Laravel

Let's apply the Null Object Pattern to solve this problem.

We'll modify our `Post` model to use `withDefault` method on the `user`
relationship:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class)
        ->withDefault([
            'name' => 'Unknown user',
        ]);
    }
}
```

Now, if the user is null, we will get the default user object with the name set
to `Unknown user`; therefore we don't have to worry about null safety in our
blade file.

```php
// post.show.blade.php
{{ $post->user->name }}
```

We can also supply a fallback model instance to the `withDefault` method like
this:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class)
        ->withDefault(NullUser::make()->toArray());
    }
}
```

And the `NullUser` class will look something like this:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class NullUser extends User
{
    protected $attributes = [
        'name' => 'Unknown user',
    ];
}
```

Now, if the user is null, we will get the `NullUser` instance with the name set
to `Unknown user`.

### Conclusion

In this article, we learned how to implement the Null Object Pattern in Laravel
and how it can help us avoid null references. For more information, you can check out
the [official documentation](https://laravel.com/docs/10.x/eloquent-relationships#default-models).

**Medium Link**: https://brainlet.medium.com/null-safety-in-laravel-null-object-pattern-cb78682ac630 
