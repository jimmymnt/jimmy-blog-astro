---
title: "Request in LaraWP"
publishDate: "2 September 2023"
description: "Another example post for Astro Cactus, this time written in a plain markdown file"
tags: ["larawp", "request"]
---
# HTTP Requests

- [Introduction](#introduction)
- [Interacting With The Request](#interacting-with-the-request)
  - [Accessing The Request](#accessing-the-request)
- [Form Request Validation](#form-request-validation)
  - [Creating Form Requests](#creating-form-requests)
  - [Displaying The Validation Errors](#displaying-the-validation-errors)
  - [Stopping On First Validation Failure](#stopping-on-first-validation-failure)
  - [Stopping On First Validation Failure Attribute](#stopping-on-first-validation-failure-attribute)

## Introduction

LaraWP provides several approaches to validate your application's incoming data.
It is most common to use the validate method available on all incoming HTTP requests.
However, we will discuss other approaches to validation as well.

LaraWP includes a wide variety of convenient validation rules that you may apply to data, even providing the ability to
validate if values are unique in a given database table. We'll cover each of these validation rules in detail so that
you are familiar with all of LaraWP's validation features.

LaraWP `LaraWP\Http\Request` class provides an object-oriented way to interact with the current HTTP request being
handled by your application as well as retrieve the input, cookies, and files that were submitted with the request.

```html
<a name="larawp-interacting-with-the-request"></a>
```

## Interacting With The Request

```html
<a name="larawp-accessing-the-request"></a>
```

### Accessing The Request

To obtain an instance of the current HTTP request via dependency injection, you should type-hint
the `LaraWP\Http\Request` class on your route closure or controller method. The incoming request instance will
automatically be injected by the LaraWP Service Container.

```php
<?php

namespace Riak\App\Http\Controllers;

use App\Admin\Controllers\Controller;
use LaraWP\Http\RedirectResponse;
use LaraWP\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
    */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return lp_redirect('/users');
    }
}
```

As mentioned, you may also type-hint the `LaraWP\Http\Request` class on a route closure. The service container will
automatically inject the incoming request into the closure when it is executed:

```php
use LaraWP\Http\Request;
use use LaraWP\Support\Facades\Route;

Route::get('/', function (Request $request) {
    // Logic goes here
});
```

## Form Request Validation

### Creating Form Requests

For more complex validation scenarios, you may wish to create a "form request". Form requests are custom request classes
that encapsulate their own validation and authorization logic. To create a form request class, you may use the example
class below as a template:

```php
<?php

namespace Riak\App\Http\Requests;

use LaraWP\Foundation\Http\FormRequest;

class EmailVariableConfigurationSettingRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * @return array[]
     */
    public function rules(): array
    {
        return [
            // Fields go here
        ];
    }

    /**
     * @return string[]
     */
    public function messages(): array
    {
        return [
            // Messages go here
        ];
    }
}
```

### Displaying The Validation Errors

So, what if the incoming request fields do not pass the given validation rules? As mentioned previously, Laravel will
automatically redirect the user back to their previous location. In addition, all the validation errors and request
input will automatically be flashed to the session.

An `$errors` variable is shared with all of your application's views by the
`LaraWP\View\Middleware\ShareErrorsFromSession` middleware, which is provided by the web middleware group. When this
middleware is applied an `$errors` variable will always be available in your views, allowing you to conveniently assume
the `$errors` variable is always defined and can be safely used.

It's allowing us to display the error messages in the view:

```php
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

### Stopping On First Validation Failure

Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so,
assign the `bail` rule to the attribute:

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

In this example, if the `unique` rule on the title attribute fails, the `max` rule will not be checked. Rules will be
validated in the order they are assigned.

### Stopping On First Validation Failure Attribute

By adding a `stopOnFirstFailure` property to your request class, you may inform the validator that it should stop
validating all attributes once a single validation failure has occurred:

```php
/**
 * Indicates if the validator should stop on the first rule failure.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

### Customizing The Redirect Location

A redirect response will be generated to send the user back to their previous location when form request validation
fails. However, you are free to customize this behavior. To do so, define a `$redirect` property on your form request:

```php
/**
 * The URI that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

Or, if you would like to redirect users to a named route, you may define a `$redirectRoute` property instead:

```php
/**
 * The route that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```
