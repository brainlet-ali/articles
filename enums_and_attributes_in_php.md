# Using ENUMs and Attributes in PHP 8

PHP 8.1 introduced two powerful features: "Enums" and "Attributes". While Enums allow developers to define a set of named values, Attributes provide a way to add metadata to classes, methods, properties, and more. In this article, we'll explore how to use both Enums and Attributes in PHP 8.1 with practical examples.

### What are ENUMs?

Enums, short for "enumerations", are a way to represent a set of named values. They can be thought of as a special class type that can have a number of predefined constant values.

### Why use ENUMs?

- **Type Safety**: Enums provide a way to ensure that only valid values are used.
- **Code Clarity**: Using named values instead of magic numbers or strings makes your code more readable.
- **Refactoring**: Changing the underlying value of an enum constant is easier and safer.

### Defining an ENUM

```php
enum DaysOfWeek {
    case MONDAY;
    case TUESDAY;
    // ... and so on
}
```

### Using an ENUM
    
```php
function getWeekendDay(DaysOfWeek $day): string {
    if ($day === DaysOfWeek::SATURDAY || $day === DaysOfWeek::SUNDAY) {
        return "It's a weekend!";
    }
    
    return "It's a weekday.";
}
```

### Backed ENUMs

Enums can be backed by any scalar type (int, string etc.). For example, the following enum is backed by an integer:

```php
enum DaysOfWeek: int {
    case MONDAY = 1;
    case TUESDAY = 2;
    // ... and so on
}
```

### What are Attributes?

Attributes, also known as annotations in other languages, allow developers to add metadata to classes, methods, properties, and more. This metadata can then be retrieved at runtime using reflection.

### Using Attributes with ENUMs

Imagine you want to add a description to each day of the week in our `DaysOfWeek` enum. You can define an attribute called `Description` and use it with the enum:

```php
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_ENUM_CASE)]
class Description {
    public function __construct(public string $value) {}
}

enum DaysOfWeek: string {
    #[Description("The first day of the week")]
    case MONDAY = 'Monday';

    #[Description("The second day of the week")]
    case TUESDAY = 'Tuesday';

    // ... and so on
}
```

To retrieve the description of an enum case:

```php
$reflection = new ReflectionEnumCase(DaysOfWeek::class, 'MONDAY');
$attribute = $reflection->getAttributes(Description::class)[0] ?? null;

if ($attribute) {
    $description = $attribute->newInstance()->value;
    echo $description;  // Outputs: The first day of the week
}
```

## Using ENUMs and Attributes in Laravel: A Real-World Example

In a Laravel-based application, routes are essential for directing incoming requests to appropriate controllers. Let's say we're building a content management system (CMS) and want to filter articles based on their publication status. We can use ENUMs to represent different publication statuses and Attributes to add metadata like descriptions.

### 1. Defining the PublicationStatus ENUM

```php
enum PublicationStatus: string {
    case DRAFT = 'draft';
    case PUBLISHED = 'published';
    case ARCHIVED = 'archived';
}
```

### 2. Using Attributes for Metadata

```php
#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_ENUM_CASE)]
class StatusDescription {
    public function __construct(public string $value) {}
}

enum PublicationStatus: string {
    #[StatusDescription("Article is in draft mode")]
    case DRAFT = 'draft';

    #[StatusDescription("Article has been published")]
    case PUBLISHED = 'published';

    #[StatusDescription("Article has been archived")]
    case ARCHIVED = 'archived';
}
```

### 3. Using ENUMs in Routes

```php
use App\Enums\PublicationStatus;

Route::get('/articles/{status}', function (PublicationStatus $status) {
    // Fetch articles based on the provided status
    $articles = Article::where('status', $status->value)->get();
    return view('articles.index', ['articles' => $articles]);
});

```

y type-hinting the `PublicationStatus` ENUM in the route closure, Laravel will automatically resolve the correct ENUM value or throw a 404 error if an invalid status is provided.

### 4.  Using Attributes in Middleware

We can also use Attributes to add middleware that displays a description for the current publication status:

```php
use ReflectionEnumCase;

class DisplayStatusDescription {
    public function handle($request, Closure $next) {
        $statusEnum = PublicationStatus::from($request->route('status'));
        $reflection = new ReflectionEnumCase(PublicationStatus::class, $statusEnum->name);
        $attribute = $reflection->getAttributes(StatusDescription::class)[0] ?? null;

        if ($attribute) {
            $description = $attribute->newInstance()->value;
            view()->share('statusDescription', $description);
        }

        return $next($request);
    }
}
```

By applying this middleware to the route, the statusDescription variable will be available in all views, allowing you to display the description of the current publication status.


## Conclusion

In PHP 8.1, the introduction of ENUMs and Attributes has significantly enhanced code clarity and type safety. ENUMs offer a structured way to define a set of named values, reducing potential errors from using loosely-typed values. Attributes allow for embedding metadata directly within code elements, streamlining configurations and adding context. When applied to frameworks like Laravel, these features ensure more robust route handling, model attribute management, and enriched middleware functionality. Together, ENUMs and Attributes elevate the expressiveness and maintainability of PHP applications.

## References

- [PHP 8.1 Enums RFC](https://wiki.php.net/rfc/enumerations)
- [PHP 8.1 Attributes RFC](https://wiki.php.net/rfc/attributes_v2)
- [Laravel 10.x Routing](https://laravel.com/docs/10.x/routing)
- [Laravel 10.x Middleware](https://laravel.com/docs/10.x/middleware)
