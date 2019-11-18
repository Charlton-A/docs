# Middleware

## Introduction

Middleware is an extremely important aspect of web applications as it allows you to run important code either before or after every request or even before or after certain routes. In this documentation we'll talk about how middleware works, how to consume middleware and how to create your own middleware. Middleware is only ran when the route is found and a status code of 200 will be returned.

## Getting Started

Middleware classes are placed inside the `app/http/middleware` by convention but can be placed anywhere you like. All middleware are just classes that contain a `before` method and/or an `after` method.

There are four types of middleware in total:

* Middleware ran before every request
* Middleware ran after every request
* Middleware ran before certain routes
* Middleware ran after certain routes

There are what Masonite calls HTTP Middleware which are middleware designed to run on every request and Route Middleware which is middleware designed to only be called on certain requests, such as checking if a user is logged in.

## Configuration

We have one of two configuration constants we need to work with. These constants both reside in our `config/middleware.py` file and are `HTTP_MIDDLEWARE` and `ROUTE_MIDDLEWARE`.

`HTTP_MIDDLEWARE` is a simple list which should contain an aggregation of your middleware classes. This constant is a list because all middleware will simply run in succession one after another, similar to Django middleware

Middleware is a string to the module location of your middleware class. If your class is located in `app/http/middleware/DashboardMiddleware.py` then the value we place in our middleware configuration will be a string: `app.http.middleware.DashboardMiddleware.DashboardMiddleware`. Masonite will locate the class and execute either the `before` method or the `after` method.

In our `config/middleware.py` file this type of middleware may look something like:

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.DashboardMiddleware.DashboardMiddleware'
]
```

## Route Middleware

`ROUTE_MIDDLEWARE` is also simple but instead of a list, it is a dictionary with a custom name as the key and the middleware class as the value. This is so we can specify the middleware based on the key in our routes file.

In our `config/middleware.py` file this might look something like:

```python
from app.http.middleware.RouteMiddleware import RouteMiddleware

ROUTE_MIDDLEWARE = {
    'auth': 'app.http.middleware.RouteMiddleware.RouteMiddleware'
}
```

### Middleware Stacks

Route middleware have the unique option of also being stacks of middleware \(or lists, really\). So we can specify a middleware to have a list of middleware instead of one string based middleware:

```python
from app.http.middleware.RouteMiddleware import RouteMiddleware

ROUTE_MIDDLEWARE = {
    'admin': 'app.http.middleware.AdminMiddleware.AdminMiddleware',
    'auth': [
        'app.http.middleware.AuthMiddleware.AuthMiddleware',
        'app.http.middleware.VerifyMiddleware.VerifyMiddleware',
    ]
}
```

Notice that we can use both lists and strings for middleware. As a list, all the middleware in the list will run when we call the `auth` middleware stack.

## Default Middleware

There are 4 default middleware that comes with Masonite. These middleware can be changed or removed to fit your application better.

### Authentication Middleware

This middleware is design to redirect users to the login page if they are not logged in. This is loaded as a Route Middleware inside `config/middleware.py` and design to only be ran on specific routes you specify.

You can run this middleware on any route by specifying the key in the middleware method on your route:

{% code title="routes/web.py" %}
```python
from masonite.helpers.routes import get
...

ROUTES = [
    ...
    get('/dashboard', 'DashboardController@show').middleware('auth')
]
```
{% endcode %}

### Verify Email Middleware

This middleware checks to see if the logged in user has verified their email. If they haven't it will redirect the user to a page reminding them to verify their email.

You can run this middleware on any route by specifying the key in the middleware method on your route:

{% code title="routes/web.py" %}
```python
from masonite.helpers.routes import get
...

ROUTES = [
    ...
    get('/dashboard', 'DashboardController@show').middleware('verified')
]
```
{% endcode %}

### CSRF Middleware

This middleware is an HTTP Middleware and runs on every request. This middleware checks for the CSRF token on `POST` requests. For `GET` requests, Masonite generates a new token. The default behavior is good for most applications but you may change this behavior to suit your application better.

### Load User Middleware

This middleware checks if the user is logged in and if so, loads the user into the request object. This enables you to use something like:

{% code title="app/http/controllers/YourController.py" %}
```python
def show(self):
    return request().user() # Returns the user or None
```
{% endcode %}

## Creating Middleware

Again, middleware should live inside the `app/http/middleware` folder and should look something like:

```python
class AuthenticationMiddleware:
    ''' Middleware class '''

    def __init__(self, Request):
        self.request = Request

    def before(self):
        pass

    def after(self):
        pass
```

Middleware constructors are resolved by the container so simply pass in whatever you like in the parameter list and it will be injected for you.

{% hint style="success" %}
Read more about this in the [Service Container](../architectural-concepts/service-container.md) documentation.
{% endhint %}

If Masonite is running a “before” middleware, that is middleware that should be ran before the request, Masonite will check all middleware and look for a `before` method and execute that. The same for “after” middleware.

You may exclude either method if you do not wish for that middleware to run before or after.

This is a boilerplate for middleware. It's simply a class with a before and/or after method. Creating a middleware is that simple. Let's create a middleware that checks if the user is authenticated and redirect to the login page if they are not. Because we have access to the request object from the Service Container, we can do something like:

```python
class AuthenticationMiddleware:
    ''' Middleware class which loads the current user into the request '''

    def __init__(self, Request):
        self.request = Request

    def before(self):
        if not self.request.user():
            self.request.redirect_to('login')

    def after(self):
        pass
```

That's it! Now we just have to make sure our route picks this up. If we wanted this to execute after a request, we could use the exact same logic in the `after` method instead.

Since we are not utilizing the `after` method, we may exclude it all together. Masonite will check if the method exists before executing it.

## Configuration

We have one of two configuration constants we need to work with. These constants both reside in our `config/middleware.py` file and are `HTTP_MIDDLEWARE` and `ROUTE_MIDDLEWARE`.

`HTTP_MIDDLEWARE` is a simple list which should contain an aggregation of your middleware classes. This constant is a list because all middleware will simply run in succession one after another, similar to Django middleware

Middleware is a string to the module location of your middleware class. If your class is located in `app/http/middleware/DashboardMiddleware.py` then the value we place in our middleware configuration will be a string: `app.http.middleware.DashboardMiddleware.DashboardMiddleware`. Masonite will locate the class and execute either the `before` method or the `after` method.

In our `config/middleware.py` file this type of middleware may look something like:

```python
HTTP_MIDDLEWARE = [
    'app.http.middleware.DashboardMiddleware.DashboardMiddleware'
]
```

`ROUTE_MIDDLEWARE` is also simple but instead of a list, it is a dictionary with a custom name as the key and the middleware class as the value. This is so we can specify the middleware based on the key in our routes file.

In our `config/middleware.py` file this might look something like:

```python
from app.http.middleware.RouteMiddleware import RouteMiddleware

ROUTE_MIDDLEWARE = {
    'auth': 'app.http.middleware.RouteMiddleware.RouteMiddleware'
}
```

## Consuming Middleware

Using middleware is also simple. If we put our middleware in the `HTTP_MIDDLEWARE` constant then we don't have to worry about it anymore. It will run on every successful request, that is when a route match is found from our `web.py` file.

If we are using a route middleware, we'll need to specify which route should execute the middleware. To specify which route we can just append a `.middleware()` method onto our routes. This will look something like:

```python
Get().route('/dashboard', 'DashboardController@show').name('dashboard').middleware('auth')
Get().route('/login', 'LoginController@show').name('login')
```

This will execute the auth middleware only when the user visits the `/dashboard` url and as per our middleware will be redirected to the named route of `login`

Awesome! You’re now an expert at how middleware works with Masonite.

