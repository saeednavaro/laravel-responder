<p align="center"><img src="http://designiack.no/package-logo.png" width="396" height="111"></p>

<p align="center">
    <a href="https://github.com/flugger/laravel-responder"><img src="https://poser.pugx.org/flugger/laravel-responder/v/stable?format=flat-square" alt="Latest Stable Version"></a>
    <a href="https://packagist.org/packages/flugger/laravel-responder"><img src="https://img.shields.io/packagist/dt/flugger/laravel-responder.svg?style=flat-square" alt="Packagist Downloads"></a>
    <a href="license.md"><img src="https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square" alt="Software License"></a>
    <a href="https://travis-ci.org/flugger/laravel-responder"><img src="https://img.shields.io/travis/flugger/laravel-responder/master.svg?style=flat-square" alt="Build Status"></a>
    <a href="https://scrutinizer-ci.com/g/flugger/laravel-responder/?branch=master"><img src="https://img.shields.io/scrutinizer/g/flugger/laravel-responder.svg?style=flat-square" alt="Code Quality"></a>
    <a href="https://scrutinizer-ci.com/g/flugger/laravel-responder/code-structure/master"><img src="https://img.shields.io/scrutinizer/coverage/g/flugger/laravel-responder.svg?style=flat-square" alt="Test Coverage"></a>
    <a href="https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=PRMC9WLJY8E46&lc=NO&item_name=Laravel%20Responder&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHosted"><img src="https://img.shields.io/badge/donate-PayPal-yellow.svg?style=flat-square" alt="Donate"></a>
</p>

Laravel Responder is a package for your JSON API, integrating [Fractal](https://github.com/thephpleague/fractal) into Laravel and Lumen. It can transform your data using transformers, create and serialize success- and error responses, handle exceptions and giving you tools to test your responses.

# Table of Contents

- [Introduction](#philosophy)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
    - [Creating Responses](#creating-responses)
    - [Creating Success Responses](#creating-success-responses)
    - [Creating Error Responses](#creating-error-responses)
    - [Creating Transformers](#creating-transformers)
    - [Transforming Data](#creating-transformers)
    - [Handling Exceptions](#handling-exceptions)
    - [Testing Responses](#testing-responses)
- [Configuration](#configuration)
- [Contributing](#contributing)
- [License](#license)

# Introduction

Laravel lets you return models directly from a controller method to convert it to JSON. This is a quick way to build APIs, but leaves your database columns exposed. [Fractal](fractal.thephpleague.com), a popular PHP package from [The PHP League](https://thephpleague.com/), solves this by introducing transformers. However, it can be a bit cumbersome to integrate into the framework as seen below:

```php
 public function index()
 {
    $resource = new Collection(User::all(), new UserTransformer());

    return response()->json((new Manager)->createData($resource)->toArray());
 }
```

Not _that_ bad, but we all get a little spoiled by Laravel's magic. Wouldn't it be better if we could refactor it to:

```php
public function index()
{
    return responder()->success(User::all())->respond();
}
```

The package will allow you to do this and much more. The goal has been to create a high-quality package that feels like native Laravel. A package that lets you embrace the power of Fractal, while hiding it behind beautiful abstractions. There has also been put a lot of focus and thought to the documentation. Happy exploration!

# Requirements

This package requires:
- PHP __7.0__+
- Laravel __5.1__+ or Lumen __5.1__+

# Installation

To get started, install the package through Composer:

```shell
composer require flugger/laravel-responder
```

## Laravel

The package supports auto-discovery, so if you use Laravel 5.5 or later you may skip registering the service provider and facades and instead run `php artisan package:discover`.

#### Register Service Provider

Append the following line to the `providers` key in `config/app.php` to register the package:

```php
Flugg\Responder\ResponderServiceProvider::class,
```

#### Register Facades _(optional)_

If you like facades, you may also append the `Responder` and `Transformer` facades to the `aliases` key:

```php
'Responder' => Flugg\Responder\Facades\Responder::class,
'Transformer' => Flugg\Responder\Facades\Transformer::class,
```

#### Publish Package Assets _(optional)_

You may additionally publish the package configuration and language file using the `vendor:publish` Artisan command:

```shell
php artisan vendor:publish --provider="Flugg\Responder\ResponderServiceProvider"
```

This will publish a `responder.php` configuration file in your `config` folder. It will also publish an `errors.php` file inside your `lang/en` folder which can be used for storing error messages.

## Lumen

#### Register Service Provider

Add the following line to `app/bootstrap.php` to register the package:

```php
$app->register(Flugg\Responder\ResponderServiceProvider::class);
```

#### Register Facades _(optional)_

You may also add the following lines to `app/bootstrap.php` to register the facades:

```php
class_alias(Flugg\Responder\Facades\Responder::class, 'Responder');
class_alias(Flugg\Responder\Facades\Transformer::class, 'Transformer');
```

#### Publish Package Assets _(optional)_

Seeing there is no `vendor:publish` command in Lumen, you will have to create your own `config/responder.php` file if you want to configure the package.

# Usage

This documentation assumes some knowledge of how [Fractal](https://github.com/thephpleague/fractal) works.

## Creating Responses

The package has a `Responder` service class, which has a `success` and `error` method to build success- and error responses respectively. To use the service and begin creating responses, pick one of the options below:

#### Option 1: Inject `Responder` Service

You may inject the `Flugg\Responder\Responder` service class directly into your controller methods:

```php
public function index(Responder $responder)
{
    return $responder->success();
}
```

You can also use the `error` method to create error responses:

```php
return $responder->error();
```

***
_Alternatively, you can type hint the `Flugg\Responder\Contract\Responder` interface for the same result._
***

#### Option 2: Use `responder` Helper

If you're a fan of Laravel's `response` helper function, you may like the `responder` helper function:

```php
return responder()->success();
```
```php
return responder()->error();
```

#### Option 3: Use `Responder` Facade

Optionally, you may use the `Responder` facade to create responses:

```php
return Responder::success();
```
```php
return Responder->error();
```

#### Option 4: Use `MakesApiResponses` Trait

Lastly, the package provides a `Flugg\Responder\Http\MakesResponses` trait you can use in your controllers:

```php
return $this->success();
```

```php
return $this->error();
```

***
_Which option you pick is up to you, they are all equivalent, the important thing is to stay consistent. The helper function (option 2) will be used for the remaining of the documentation._
***

### Building Responses

The `success` and `error` methods return a `SuccessResponseBuilder` and `ErrorResponseBuilder` respectively, which both extend an abstract `ResponseBuilder`, giving them common behaviors. They will be converted to JSON when returned from a controller, but you can explicitly create an instance of `Illuminate\Http\JsonResponse` with the `respond` method:

```php
return responder()->success()->respond();
```

```php
return responder()->error()->respond();
```

The status code is set to `200` by default, but can be changed by setting the first parameter. You can also pass a list of headers as the second argument:

```php
return responder()->success()->respond(201, ['x-foo' => true]);
```

```php
return responder()->error()->respond(404, ['x-foo' => false]);
```

***
_Consider always using the `respond` method for consistency's sake._
***

### Casting Response Data

Instead of converting the response to a `JsonResponse` using the `respond` method, you can cast the response data to a few other types, like an array:

```php
return responder()->success()->toArray();
```

```php
return responder()->error()->toArray();
```

You also have a `toCollection` and `toJson` method at your disposal.

### Decorating Response

A response decorator allows for last minute changes to the response before it's returned. The package comes with two response decorators out of the box adding a `status` and `success` field to the response output. The `decorators` key in the configuration file defines a list of all enabled response decorators:

```php
'decorators' => [
    \Flugg\Responder\Http\Responses\Decorators\StatusCodeDecorator::class,
    \Flugg\Responder\Http\Responses\Decorators\SuccessFlagDecorator::class,
],
```

You may disable a decorator by removing it from the list, or add your own decorator extending the abstract class `Flugg\Responder\Http\Responses\Decorators\ResponseDecorator`. You can also add additional decorators per response:

```php
return responder()->success()->decorator(ExampleDecorator::class)->respond();
```
```php
return responder()->error()->decorator(ExampleDecorator::class)->respond();
```

## Creating Success Responses

As briefly demonstrated above, success responses are created using the `success` method:

```php
return responder()->success()->respond();
```

Assuming no changes have been made to the configuration, the above code would output the following JSON:

```json
{
    "status": 200,
    "success": true,
    "data": null
}
```

### Setting Response Data

The `success` method takes the response data as the first argument:

```php
return responder()->success(Product::all())->respond();
```

It accepts the same data types as you would normally return from your controllers, however, it also supports query builder and relationship instances:

```php
return responder()->success(Product::where('id', 1))->respond();
```

```php
return responder()->success(Product::first()->shipments())->respond();
```

***
_The package will run the queries and convert them to collections behind the scenes._
***

### Transforming Response Data

The response data will be transformed with Fractal if you've attached a transformer to the response. There are two ways to attach a transformer; either _explicitly_ by setting it on the response, or _implicitly_ by binding it to a model. Let's look at both ways in greater details.

#### Setting Transformer On Response

You can attach a transformer to the response by sending a second argument to the `success` method. For instance, below we're attaching a simple closure transformer, transforming a list of products to only output their names:

```php
return responder()->success(Product::all(), function ($product) {
    return ['name' => $product->name];
})->respond();
```

You may also transform using a dedicated transformer class:

```php
return responder()->success(Product::all(), ProductTransformer::class)->respond();
```

```php
return responder()->success(Product::all(), new ProductTransformer)->respond();
```

***
_You can read more about creating dedicated transformer classes in the [Creating Transformers](#creating-transformers) chapter._
***

#### Binding Transformer To Model

If no transformer is set, the package will search the response data for an element implementing the `Flugg\Responder\Contracts\Transformable` interface to resolve a transformer from. You can take use of this by implementing the `Transformable` interface in your models:

```php
class Product extends Model implements Transformable {}
```

You can satisfy the contract by adding a `transformer` method that returns the corresponding transformer:

```php
/**
 * Get a transformer for the class.
 *
 * @return \Flugg\Responder\Transformers\Transformer|string|callable
 */
public function transformer()
{
    return ProductTransformer::class;
}
```

***
_You're not limited to returning a class name string, you can return a transformer instance or closure transformer, just like the second parameter of the `success` method._
***

Instead of implementing the `Transformable` contract for all models, an alternative approach is to bind the transformers using the `bind` method on the `TransformerResolver` class. You can place the code below within `AppServiceProvider` or an entirely new `TransformerServiceProvider`:

```php
use Flugg\Responder\Transformers\TransformerResolver;

public function boot()
{
    $this->app->make(TransformerResolver)->bind([
        \App\Post::class => \App\Transformers\PostTransformer::class,
        \App\User::class => \App\Transformers\UserTransformer::class,
    ]);
}
```

After you've bound a transformer to a model you can skip the second parameter and still transform the data:

```php
return responder()->success(Post::all())->respond();
```

***
_As you might have noticed, unlike Fractal, you don't need to worry about creating resource objects like `Item` and `Collection`. The package will make one for you based on the data type, however, you may wrap your data in a resource to override this._
***

### Paginating Response Data

Sending a paginator to the `success` method will set pagination meta data and transform the data automatically, as well as append any query string parameters to the paginator links.

```php
return responder()->success(Product::paginate())->respond();
```

Assuming there are no products and the default configuration is used, the JSON output would look like:

```json
{
    "success": true,
    "status": 200,
    "data": null,
    "pagination": {
        "total": 0,
        "count": 0,
        "perPage": 15,
        "currentPage": 1,
        "totalPages": 1
    }
}
```

#### Setting Paginator On Response

Instead of sending a paginator as data, you may set the data and paginator seperately, like you traditionally would with Fractal. You can manually set a paginator using the `paginator` method, which expects an instance of `League\Fractal\Pagination\IlluminatePaginatorAdapter`:

```php
$paginator = Product::paginate();
$adapter = new IlluminatePaginatorAdapter($paginator);

return responder()->success($paginator->getCollection())->paginator($adapter)->respond();
```

#### Setting Cursor On Response

You can also set cursors using the `cursor` method, which expects an instance of `League\Fractal\Pagination\Cursor`:

```php
$cursor = new Cursor(request()->cursor, request()->previous, request()->next, Product::count());

return responder()->success(Product::all())->cursor($cursor)->respond();
```

### Including Relationships

If a transformer class is attached to the response, you can include relationships using the `with` method:

```php
return responder()->success(Post::all())->with('user')->respond();
```

You can send multiple arguments and nested relations using dot notation:

```php
return responder()->success(Post::all())->with('user', 'comments.user')->respond();
```

All relationships will be automatically eager loaded, and just like you would when using `with` or `load` to eager load with Eloquent, you may use a callback to specify additional query constraints. Like in the example below, where we're only including related comments posted by the authenticated user:

```php
return responder()->success(Post::all())->with(['comments' => function ($query) {
    $query->where('user_id', Auth::id());
}])->respond();
```

#### Excluding Default Relations

In your transformer classes, you may specify relations to automatically load. You may disable any of these relations using the `without` method:

```php
return responder()->success(Post::all())->without('comments')->respond();
```

#### Including From Query String

Relationships are loaded from a query string parameter if the `load_relations_parameter` configuration key is set to a string. By default, it's set to `with`, allowing you to automatically include relations from the query string:

```
GET /users?with=user,comments.user
```

### Filtering Transform Data

The technique of filtering the transformed data to only return what we need is called sparse fieldsets and can be specified using the `only` method:

```php
return responder()->success(Product::all())->only('id', 'name')->respond();
```

### Adding Meta Data

Sometimes you may want to attach additional data to your response. You can do this using the `meta` method:

```php
return responder()->success(Product::all())->meta('count', Product::count())->respond();
```

When using the default serializer, the meta data will simply be appended to the response array:

```json
{
    "success": true,
    "status": 200,
    "data": [],
    "count": 0
}
```

### Serializing Response Data

After the data has been transformed, it will be serialized using the set success serializer in the configuration file, which defaults to the package's own `Flugg\Responder\Serializers\SuccessSerializer`. You can overwrite this on your responses using the `serializer` method:

```php
return responder()->success()->serializer(JsonApiSerializer::class)->respond();
```

```php
return responder()->success()->serializer(new JsonApiSerializer())->respond();
```

Above we're using Fractal's `JsonApiSerializer` class. Fractal also ships with an `ArraySerializer` and `DataArraySerializer` class. If none of these suit your taste, feel free to create your own serializer by extending `League\Fractal\Serializer\SerializerAbstract`. You can read more about it in [Fractal's documentation](http://fractal.thephpleague.com/serializers/).

## Creating Transformers

A dedicated transformer class gives you a convenient location to transform data and allows you to use the same transformer multiple times. It also allows you to include and transform relationships. You can create a transformer using the `make:transformer` Artisan command:

```shell
php artisan make:transformer UserTransformer
```

The command will generate a new `UserTransformer.php` file in the `app/Transformers` folder:

```php
<?php

namespace App\Transformers;

use App\User;
use Flugg\Responder\Transformers\Transformer;

class UserTransformer extends Transformer
{
    /**
     * List of available relations.
     *
     * @var string[]
     */
    protected $relations = ['*'];

    /**
     * A list of autoloaded default relations.
     *
     * @var array
     */
    protected $load = [];

    /**
     * Transform the model.
     *
     * @param  \App\User $user
     * @return array
     */
    public function transform(User $user): array
    {
        return [
            'id' => (int) $user->id,
        ];
    }
}
```

It will automatically resolve a model name from the name provided. For instance, in the example above, the package will extract `User` from `UserTransformer` and assume the models live directly in the `app` folder (as per Laravel's convention). If you store them somewhere else, you can use the `--model` (or `-m`) option to override it: 

```shell
php artisan make:transformer UserTransformer --model="App\Models\User"
```

#### Creating Plain Transformers

The transformer file generated above is a model transformer expecting an `App\User` model for the `transform` method. However, we can create a plain transformer by applying the `--plain` (or `-p`) modifier:

```shell
php artisan make:transformer UserTransformer --plain
```

This will remove the typehint from the `transform` method and add less boilerplate code.

### Including Relationships

All transformers generated through the `make:transformer` command will include a `$relations` and `$with` property, which are the equivalent to Fractal's `$availableIncludes` and `$defaultIncludes`. Fractal also requires you to to create methods in your transformer for all included relation. While this package also allows you to create such methods, it doesn't require it if you're transforming models. For instance, if you're including a `user` relation in a `PostTransformer`, the package will assume you have a `user` relationship method in your `Post` model and automatically fetch the relation. You can overwrite this by creating an `includeUser` method in `PostTransformer`, just like you would with Fractal:

```php
/**
 * Include related user.
 *
 * @param  \App\Post                     $post
 * @param  \League\Fractal\ParamBag|null $parameters
 * @return \League\Fractal\ItemResource
 */
public function includeUser(Post $post, ParamBag $parameters = null)
{
    return $this->resource($post->user);
}
```

The `resource` method used above replaces Fractal's `item` and `collection` methods in the Transformer for creating a resource. It will automatically figure out wether it should be an item or collection resource based on the data. It will also resolve a transformer from the `User` model, if a transformer binding is set, just like the `success` method. In fact, it accepts the exact same arguments as the `success` method:

```php
return $this->resource($post->user, new UserTransformer);
```

***
_You should be careful with executing any new database calls inside the include methods as you might end up with an unexpected amount of hits to the database._
***

#### Setting Available Relationships

The `$relations` property specifies a list of relations available to be included. When you generate a transformer, the `$relations` property will be equal to a wildcard, allowing all relations on the transformer:

```php
protected $relations = ['*'];
```

If you only want to whitelist certain relations, you can instead set a list of relations you want to make available:

```php
protected $relations = ['user', 'comments'];
```

***
_**Security warning:** Since the transformer doesn't know what relations exists on a model unless you specify it in `$relations`, you're technically allowing calls to any method on your model when using a wildcard. You should therefore consider always specifying a whitelist._
***

#### Setting Default Relationships

The `$with` property specifies a list of relations to be autoloaded every time you transform data with the transformer. By mapping a transformer to the relation the package will also be able to automatically eager load all default relations, including nested ones:

```php
protected $with = [
    'user' => UserTransformer::class,
    'comments' => CommentTransformer::class,
];
```

If you're transforming non-model data or don't care about the eager loading, you can skip the transformer mapping and just specify a list of relations:

```php
protected $with = ['user', 'comments'];
```

***
_You don't have to add relations to both `$relations` and `$with`, all relations in `$with` will be available by nature._
***

### Filtering Relationships

After a relation has been included, you can make any last second changes to it using a filter method. For instance, below we're filtering the comments collection to only include comments containing the word "Laravel".

```php
/**
 * Filter included comments.
 *
 * @param  \Illuminate\Database\Eloquent\Collection $comments
 * @return \Illuminate\Database\Eloquent\Collection
 */
public function filterComments($comments)
{
    return $comments->filter(function ($comment) {
        return str_contains($comment->body, 'Laravel');
    });
}
```

## Transforming Data

We've already looked at how to transform data when creating success responses, however, you may want to transform data in other places than your controllers. An example of when you would want to transform data is in your broadcasted events. You're exposing data using websockets instead of HTTP, but you still want to receive the same transformed data in your frontend. 

#### Option 1: The `transform` Helper

You can use the `transform` helper function to transform data without creating a response:

```php
transform(Post::all());
```

Unlike the `success` method, this wont serialize the data. However, it will resolve a transformer from the model if a binding is set, and you can overwrite the transformer by setting a second parameter. You can also specify a list of included relations as a third argument:

```php
transform(Post::all(), new PostTransformer, ['comments']);
```

In addition, if you want to blacklist any of the default loaded relations, you can fill the fourth parameter:

```php
transform(Post::all(), new PostTransformer, ['comments'], ['user']);
```

#### Option 2: The `Transformer` Facade

Instead of using the `transform` helper function, you can use the `Transformer` facade to do the same thing:

```php
Transformer::transform(Post::all(), new PostTransformer, ['comments'], ['user']);
```

#### Option 3: The `Transformer` Service

Both the helper method and facade uses the `Flugg\Responder\Transformer` service class to apply the transformation. You can use the service yourself by injecting the service:

```php
public function __construct(Transformer $transformer)
{
    $transformer->transform(Post::all(), new PostTransformer, ['comments'], ['user']);
}
```

### Transforming To Camel Case

Model attributes are traditionally specified in snake case, however, you might prefer to use camel case in your response data. A transformer makes for a perfect location to convert the attributes, like the `userId` field in the example below:

```php
return responder()->transform($post, function ($post) {
    return [
        'id' => (int) $post->id,
        'userId' => (int) $post->user_id,
    ];    
})->respond();
```

#### Transforming Requests To Snake Case

After responding with camel case, you probably want to let people send in request data using camel case parameters as well. The package provides a `Flugg\Responder\Http\Middleware\ConvertToSnakeCase` middleware you may append to the `$middleware` array in `app/Http/Kernel.php` to convert all request parameters to snake case automatically:

```php
protected $middleware = [
    // ...
    \Flugg\Responder\Http\Middleware\ConvertToSnakeCase::class,
];
```

***
_The middleware will run before request validation, so you should specify your validation rules in snake case as well._
***

## Creating Error Responses

Whenever a consumer of your API does something unexpected, you can return an error response describing the problem. As briefly shown in a previous chapter, an error response can be created using the `error` method:

```php
return responder()->error()->respond();
```

The error response has knowledge about an error code, a corresponding error message, and optionally some error data. If using the default configuration, the above code would output the following JSON:

```json
{
    "success": false,
    "status": 500,
    "error": {
        "code": null,
        "message": null
    }
}
```

### Setting Error Code & Message

You can fill the first parameter of the `error` method to set an error code:

```php
return responder()->error('sold_out_error')->respond();
```

***
_You can also use integers as error codes._
***

Additionally, you may set the second parameter to an error message describing the error:

```php
return responder()->error('sold_out_error', 'The requested product is sold out.')->respond();
```

#### Set Messages In Language Files

Alternatively, you can set the error messages in a language file, allowing for returning messages in different languages for different consumers. The configuration file has an `error_message_files` key defining a list of language files with error messages. By default, it is set to `['errors']`, meaning it will look for an `errors.php` file inside `resources/lang/en`. You can use these files to map error codes to corresponding error messages:

```php
return [
    'sold_out_error' => 'The requested product is sold out.',
];
```

#### Register Messages On `ErrorMessageResolver`

Instead of implementing the `Transformable` contract for all models, an alternative approach is to bind the transformers using the `bind` method on the `TransformerManager` class. You can place the code below within `AppServiceProvider` or an entirely new `TransformerServiceProvider`:

```php
use Flugg\Responder\ErrorMessageResolver;

public function boot()
{
    $this->app->make(ErrorMessageResolver::class)->register([
        'sold_out_error' => 'The requested product is sold out.',
    ]);
}
```

### Adding Error Data

You may want to set additional data on the error response. Like in the example below, we're returning a list of shipments with the `sold_out` error response, giving the consumer information about when a new shipment for the product might arrive.

```php
return responder()->error('sold_out')->data(['shipments' => Shipment::all()])->respond();
```

The error data will be appended to the response data. Assuming we're using the default serializer and there are no shipments in the database, the code above would look like:

```json
{
    "success": false,
    "status": 500,
    "error": {
        "code": "sold_out",
        "message": "The requested product is sold out.",
        "shipments": []
    }
}
```

### Serializing Response Data

## Handling Exceptions

No matter how much we try to avoid them, exceptions do happen. Responding to the exceptions in an elegant manner will improve the user experience of your API. The package can enhance your exception handler to automatically turn exceptions in to error responses. If you want to take use of this, you can either use the package's exception handler or include a trait as described in further details below.

#### Option 1: Replace `Handler` Class

To use the package's exception handler you need to replace the default import in `app/Exceptions/Handler.php`:

```php
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
```

With the package's handler class:

```php
use Flugg\Responder\Exceptions\Handler as ExceptionHandler;
```

***
This will not work with Lumen as its exception handler is incompatible with Laravel's. Look instead at the second option below.
***

#### Option 2: Use `ConvertsExceptions` Trait

The package's exception handler uses the `Flugg\Responder\Exceptions\ConvertsExceptions` trait to load of most of its work. Instead of replacing the exception handler, you can use the trait in your own handler class. To replicate the behavior of the exception handler, you would also have to add the following code to the `render` method:

```php
public function render($request, Exception $exception)
{
    $this->convertDefaultException($exception);

    if ($exception instanceof HttpException) {
        return $this->renderResponse($exception);
    }

    return parent::render($request, $exception);
}
```

### Converting Exceptions

Once you've implemented one of the above options, the package will convert some of Laravel's exceptions to an exception extending `Flugg\Responder\Exceptions\Http\ApiException`. It will then convert these to an error response. The table below shows which Laravel exceptions are converted and what they are converted to. All the exceptions on the right is under the `Flugg\Responder\Exceptions\Http` namespace and extends `Flugg\Responder\Exceptions\Http\ApiException`. All exceptions extending the `ApiException` class will be automatically converted to an error response. 

| Caught Exceptions                                               | Converted To                 |
| --------------------------------------------------------------- | ---------------------------- |
| `Illuminate\Auth\AuthenticationException`                       | `UnauthenticatedException`   |
| `Illuminate\Auth\Access\AuthorizationException`                 | `UnauthorizedException`      |
| `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`  | `PageNotFoundException`      |
| `Illuminate\Database\Eloquent\ModelNotFoundException`           | `PageNotFoundException`      |
| `Illuminate\Database\Eloquent\RelationNotFoundException`        | `RelationNotFoundException`  |
| `Illuminate\Validation\ValidationException`                     | `ValidationFailedException`  |

You can disable the conversions of some of the exceptions above using the `$dontConvert` property:

```php
/**
 * A list of default exception types that should not be converted.
 *
 * @var array
 */
protected $dontConvert = [
    ModelNotFoundException::class,
];
```

***
If you're using the trait option, you can disable all the default conversions by removing the call to `convertDefaultException` in the `render` method.
***

#### Convert Custom Exceptions

In addition to letting the package convert Laravel exceptions, you can also convert your own exceptions using the `convert` method in the `render` method:

```php
$this->convert($exception, [
    InvalidValueException => PageNotFoundException,
]);
```

You can optionally give it a closure that throws the new exception, if you want to give it constructor parameters: 

```php
$this->convert($exception, [
    MaintenanceModeException => function ($exception) {
        throw new ServerDownException($exception->retryAfter);
    },
]);
```

### Creating API Exceptions

An exception class is a convenient place to store information about an error. The package provides an abstract exception class `Flugg\Responder\Exceptions\Http\ApiException`, which has knowledge about status code, an error code and an error message. Continuing on our product example from above, we could create our own `ApiException` class:

```php
<?php

namespace App\Exceptions;

use Flugg\Responder\Exceptions\Http\ApiException;

class SoldOutException extends ApiException
{
    /**
     * An HTTP status code.
     *
     * @var int
     */
    protected $status = 400;

    /**
     * An error code.
     *
     * @var string|null
     */
    protected $code = 'sold_out_error';

    /**
     * An error message.
     *
     * @var string|null
     */
    protected $message = 'The requested product is sold out.';
}
```

You can also add a `data` method returning additional error data:

```php
/**
 * Retrieve additional error data.
 *
 * @return array|null
 */
public function data()
{
    return [
        'shipments' => Shipment::all()
    ];
}
```

If you're letting the package handle exceptions, you can now throw the exception anywhere in your application and it will automatically be rendered to an error response.

```php
throw new SoldOutException();
```

# Configuration

If you've published vendor assets as described in the [installation guide](#installation), you will have access to a `responder.php` file in you `config` folder. You may update this file to change how the package should operate. We'll go through each configuration key.

#### Serializer Class Path

This key represents the full class path to the serializer class you would like the package to use when generating successful JSON responses. You may leave it with the default `Flugg\Responder\Serializers\ApiSerializer`, change it to one of [Fractal's serializers](http://fractal.thephpleague.com/serializers/), or create a [custom one yourself](#custom-serializers).

#### Include Status Code

The package will include a status code for both success- and error responses. You can disable this by setting this key to `false`.

# Contributing

Contributions are more than welcome and you're free to create a pull request on Github. You can run tests with the following command:

```shell
vendor/bin/phpunit
```

If you find bugs or have suggestions for improvements, feel free to submit an issue on Github. However, if it's a security related issue, please send an email to flugged@gmail.com instead.

# License

Laravel Responder is free software distributed under the terms of the MIT license. See [license.md](license.md) for more details.

# Donating