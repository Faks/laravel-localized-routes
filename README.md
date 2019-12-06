# Laravel Localized Routes

[![GitHub release](https://img.shields.io/github/release/codezero-be/laravel-localized-routes.svg)]()
[![License](https://img.shields.io/packagist/l/codezero/laravel-localized-routes.svg)]()
[![Build Status](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/badges/build.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/build-status/master)
[![Code Coverage](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/codezero-be/laravel-localized-routes/?branch=master)
[![Total Downloads](https://img.shields.io/packagist/dt/codezero/laravel-localized-routes.svg)](https://packagist.org/packages/codezero/laravel-localized-routes)

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/R6R3UQ8V)

#### A convenient way to set up, manage and use localized routes in a Laravel app.

- [Automatically register](#-register-routes) a route for each locale you wish to support.
- Use [URL slugs or custom domains](#-supported-locales) (or subdomains).
- Optionally omit the locale slug from the URL for your main locale.
- [Generate localized route URL's](#-generate-route-urls) using the `route()` helper.
- [Redirect to localized routes](#-redirect-to-routes) using the `redirect()->route()` helper.
- [Generate localized signed route URL's](#-generate-signed-route-urls)
- Allow routes to be [cached](#-cache-routes).
- Optionally [translate each segment](#-translate-routes) in your URI's.
- Use the middleware to [automatically set the appropriate app locale](#-use-middleware-to-update-app-locale) (and use [route model binding](#-route-model-binding)).
- **Work with routes without thinking too much about locales.**

## 🔌 Demo App

To test and demonstrate some of the features this package provides, I have created a basic Laravel 6 demo app.
Feel free to open any issues there if you want me to test/demonstrate more stuff.

- https://github.com/ivanvermeyen/test-app-laravel-localized-routes

## ✅ Requirements

- PHP >= 7.1
- Laravel >= 5.6

## 📦 Install

```bash
composer require codezero/laravel-localized-routes
```

> Laravel will automatically register the ServiceProvider.

## ⚙️ Configure

#### ☑️ Publish Configuration File

```php
php artisan vendor:publish --provider="CodeZero\LocalizedRoutes\LocalizedRoutesServiceProvider" --tag="config"
```

You will now find a `localized-routes.php` file in the `config` folder.

#### ☑️ Supported Locales

##### Using Slugs

Add any locales you wish to support to your published `config/localized-routes.php` file:

```php
'supported-locales' => ['en', 'nl', 'fr'],
```

 This will automically prepend a slug to your localized routes. [More on this below](#-register-routes).

##### Using Domains

Alternatively, you can use a different domain or subdomain for each locale by adding them to the `supported-locales` like this:

```php
'supported-locales' => [
  'en' => 'example.com',
  'nl' => 'nl.example.com',
  'fr' => 'fr.example.com',
],
```

#### ☑️ Omit Slug for Main Locale

Specify your main locale if you want to omit its slug from the URL:

```php
'omit_url_prefix_for_locale' => null
```

Setting this option to `'en'` will result, for example, in URL's like this:

- English: `/some-url` instead of the default `/en/some-url`
- Dutch: `/nl/some-url` as usual
- French: `/fr/some-url` as usual

> This option has no effect if you use domains instead of slugs.

#### ☑️ Use Middleware to Update App Locale

By default, the app locale will always be what you configured in `config/app.php`. To automatically update the app locale when a localized route is accessed, you need to use a middleware.

**⚠️ Important note for Laravel 6+**

To make route model binding work in Laravel 6+ you always also need to add the middleware to the `$middlewarePriority` array in `app/Http/Kernel.php` so it runs before `SubstituteBindings`:

```php
protected $middlewarePriority = [
    \Illuminate\Session\Middleware\StartSession::class, // <= after this
    //...
    \CodeZero\LocalizedRoutes\Middleware\LocalizedRouteLocaleHandler::class,
    //...
    \Illuminate\Routing\Middleware\SubstituteBindings::class, // <= before this
];
```

You can then enable the middleware in a few ways:

**For every route, via our config file**

Simply set the option to `true`:

```php
'use_locale_middleware' => true
```

**OR, for every route using the `web` middleware group**

You can manually add the middleware to the `$middlewareGroups` array in `app/Http/Kernel.php`:

```php
protected $middlewareGroups = [
    'web' => [
        \Illuminate\Session\Middleware\StartSession::class, // <= after this
        //...
        \CodeZero\LocalizedRoutes\Middleware\LocalizedRouteLocaleHandler::class,
        //...
        \Illuminate\Routing\Middleware\SubstituteBindings::class, // <= before this
    ],
];
```

**OR, for specific routes**

Alternatively, you can add the middleware to a specific route or route group:

```php
Route::localized(function () {

    Route::get('about', AboutController::class.'@index')
        ->name('about')
        ->middleware(\CodeZero\LocalizedRoutes\Middleware\LocalizedRouteLocaleHandler::class);

    Route::group([
        'as' => 'admin.',
        'middleware' => [\CodeZero\LocalizedRoutes\Middleware\LocalizedRouteLocaleHandler::class],
    ], function () {

        Route::get('admin/reports', ReportsController::class.'@index')
            ->name('reports.index');

    });

});
```

#### ☑️ Set Options for the Current Localized Route Group

To set an option for one localized route group only, you can specify it as the second parameter of the localized route macro. This will override the config file settings.

```php
Route::localized(function () {

    Route::get('about', AboutController::class.'@index')
        ->name('about');

}, [
    'supported-locales' => ['en', 'nl', 'fr'],
    'omit_url_prefix_for_locale' => null,
    'use_locale_middleware' => false,
]);
```

## 🚗 Register Routes

Example:

```php
// Not localized
Route::get('home', HomeController::class.'@index')
    ->name('home');

// Localized
Route::localized(function () {

    Route::get('about', AboutController::class.'@index')
        ->name('about');

    Route::name('admin.')->group(function () {
        Route::get('admin/reports', ReportsController::class.'@index')
            ->name('reports.index');
    });

});
```

In the above example there are 5 routes being registered. The routes defined in the `Route::localized` closure are automatically registered for each configured locale. This will prepend the locale to the route's URI and name. If you configured custom domains, it will use those instead of the slugs.

| URI               | Name                   |
| ----------------- | ---------------------- |
| /home             | home                   |
| /en/about         | en.about               |
| /nl/about         | nl.about               |
| /en/admin/reports | en.admin.reports.index |
| /nl/admin/reports | nl.admin.reports.index |

If you set `omit_url_prefix_for_locale` to `'en'` in the configuration file, the resulting routes look like this: 

| URI               | Name                   |
| ----------------- | ---------------------- |
| /home             | home                   |
| /about            | en.about               |
| /nl/about         | nl.about               |
| /admin/reports    | en.admin.reports.index |
| /nl/admin/reports | nl.admin.reports.index |

**⚠️ Beware that you don't register the same URL twice when omitting the locale.** You can't have a localized `/about` route and also register a non-localized `/about` route in this case. The same idea applies to the `/` (root) route! Also note that the route names still have the locale prefix.

### 🚕 Generate Route URL's

You can get the URL of your named routes as usual, using the `route()` helper.

##### 👎 The ugly way...

Normally you would have to include the locale whenever you want to generate a URL:

```php
$url = route(app()->getLocale().'.admin.reports.index');
```

##### 👍 A much nicer way...

Because the former is rather ugly, this package overwrites the `route()` function and the underlying `UrlGenerator` class with an additional, optional `$locale` argument and takes care of the locale prefix for you. If you don't specify a locale, either a normal, non-localized route or a route in the current locale is returned.

```php
$url = route('admin.reports.index'); // current locale
$url = route('admin.reports.index', [], true, 'nl'); // dutch URL
```

This is the new route helper signature:

```php
route($name, $parameters = [], $absolute = true, $locale = null)
```

A few examples (given the example routes we registered above):

```php
app()->setLocale('en');
app()->getLocale(); // 'en'

$url = route('home'); // /home (normal routes have priority)
$url = route('about'); // /en/about (current locale)

// Get specific locales...
// This is most useful if you want to generate a URL to switch language.
$url = route('about', [], true, 'en'); // /en/about
$url = route('about', [], true, 'nl'); // /nl/about

// You could also do this, but it kinda defeats the purpose...
$url = route('en.about'); // /en/about
$url = route('en.about', [], true, 'nl'); // /nl/about
```

> **Note:** in a most practical scenario you would register a route either localized **or** non-localized, but not both. If you do, you will always need to specify a locale to get the URL, because non-localized routes always have priority when using the `route()` function.

### 🚌 Redirect to Routes

Laravel's `Redirector` uses the same `UrlGenerator` as the `route()` function behind the scenes. Because we are overriding this class, you can easily redirect to your routes.

```php
return redirect()->route('home'); // non-localized route, redirects to /home
return redirect()->route('about'); // localized route, redirects to /en/about (current locale)
```

You can't redirect to URL's in a specific locale this way, but if you need to, you can of course just use the `route()` function.

```php
return redirect(route('about', [], true, 'nl')); // localized route, redirects to /nl/about
```

### ✍🏻 Generate Signed Route URL's

Generating a [signed route URL](https://laravel.com/docs/5.8/urls#signed-urls) is just as easy.

Pass it the route name, the nescessary parameters and you will get the URL for the current locale.

```php
$signedUrl = URL::signedRoute('reset.password', ['user' => $id], now()->addMinutes(30));
```

You can also generate a signed URL for a specific locale:

```php
$signedUrl = URL::signedRoute($name, $parameters, $expiration, true, 'nl');
```

Check out the [Laravel docs](https://laravel.com/docs/5.8/urls#signed-urls) for more info on signed routes.

### 🌎 Translate Routes

If you want to translate the segments of your URI's, create a `routes.php` language file for each locale you [configured](#configure-supported-locales):

```
resources
 └── lang
      ├── en
      │    └── routes.php
      └── nl
           └── routes.php
```

In these files, add a translation for each segment.

```php
// lang/nl/routes.php
return [
    'about' => 'over',
    'us' => 'ons',
];
```

Now you can use our `Lang::uri()` macro during route registration:

```php
Route::localized(function () {

    Route::get(Lang::uri('about/us'), AboutController::class.'@index')
        ->name('about.us');

});
```

The above will generate:

- /en/about/us
- /nl/over/ons

> If a translation is not found, the original segment is used.

## 🚏 Route Parameters

Parameter placeholders are not translated via language files. These are values you would provide via the `route()` function. The `Lang::uri()` macro will skip any parameter placeholder segment.

If you have a model that uses a route key that is translated in the current locale, then you can still simply pass the model to the `route()` function to get translated URL's.

An example...

**Given we have a model like this:**

```php
class Post extends \Illuminate\Database\Eloquent\Model
{
    public function getRouteKey()
    {
        $slugs = [
            'en' => 'en-slug',
            'nl' => 'nl-slug',
        ];

        return $slugs[app()->getLocale()];
    }
}
```

> **TIP:** checkout [spatie/laravel-translatable](https://github.com/spatie/laravel-translatable) for translatable models.

**If we have a localized route like this:**

```php
Route::localized(function () {

    Route::get('posts/{post}', PostsController::class.'@show')
        ->name('posts.show');

});
```

**We can now get the URL with the appropriate slug:**

```php
app()->setLocale('en');
app()->getLocale(); // 'en'

$post = new Post;

$url = route('posts.show', [$post]); // /en/posts/en-slug
$url = route('posts.show', [$post], true, 'nl'); // /nl/posts/nl-slug
```

## 🚴‍ Route Model Binding

If you enable the [middleware](#-use-middleware-to-update-app-locale) included in this package,
you can use [Laravel's route model binding](https://laravel.com/docs/routing#route-model-binding)
to automatically inject models with localized route keys in your controllers.

All you need to do is add a `resolveRouteBinding()` method to your model.
Check [Laravel's documentation](https://laravel.com/docs/routing#route-model-binding)
for alternative ways to enable route model binding.

```php
public function resolveRouteBinding($value)
{
    // Perform the logic to find the given slug in the database...
    return $this->where('slug->'.app()->getLocale(), $value)->firstOrFail();
}
```

> **TIP:** checkout [spatie/laravel-translatable](https://github.com/spatie/laravel-translatable) for translatable models.

## 🗃 Cache Routes

In production you can safely cache your routes per usual.

```bash
php artisan route:cache
```

## 🚧 Testing

```bash
composer test
```

## ☕️ Credits

- [Ivan Vermeyen](https://byterider.io)
- [All contributors](../../contributors)

## 🔓 Security

If you discover any security related issues, please [e-mail me](mailto:ivan@codezero.be) instead of using the issue tracker.

## 📑 Changelog

See a list of important changes in the [changelog](CHANGELOG.md).

## 📜 License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
