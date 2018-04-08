# Sessions

# Introduction

You'll find yourself needing to add temporary data to an individual user. Sessions allow you to do this by adding a key value pair and attaching it to the user's IP address. You can reset the session data anytime you want; which is usually done on logout or login actions.

The Session features are adding to the framework though the `SessionProvider` Service Provider. This provider needs to be between the `AppProvider` and the `RouteProvider`. For Masonite 1.5+, this provider is already available for you.

It's important to note that the Session will default to the `memory` driver. This means that 

## Getting Started

There are a two ideas behind sessions. There is **session data** and **flash data**. Session data is any data that is persistent for the duration of the session and flash data is data that is only persisted on the next request. Flash data is useful for showing things like success or warning messages after a redirection.

## Using Sessions

Sessions is loaded into the container with the `Session` key. So you may access the `Session` class in any parts of the code that is resolved by the container. These include controllers, drivers, middleware etc:

```python
def show(self, Session):
    print(Session) # Session class
```

## Setting Data

Data can easily be persisted to the current user by using the `set` method.

```python
def show(self, Session):
    Session.set('key', 'value')
```

This will update a dictionary that is linked to the current user.

## Getting Data

Data can be pulled from the session:

```python
def show(self, Session):
    Session.get('key') # Returns 'value'
```

## Checking Data

Very often, you will want to see if a value exists in the session:

```python
def show(self, Session):
    Session.has('key') # Returns True
```

## Getting all Data

You can get all data for the current user:

```python
def show(self, Session):
    Session.all() # Returns {'key': 'value'}
```

## Flashing Data

Data can be inserted only for the next request. This is useful when using redirection and displaying a success message.

```python
def show(self, Session):
    Session.flash('success', 'Your action is successful')
```

## Request Class

The `SessionProvider` attaches a `session` attribute to the `Request` class. This attribute is the `Session` class itself.

```python
def show(self):
    request().session.flash('success', 'Your action is successful')
```

## Templates

The `SessionProvider` comes with a helper method that is automatically injected into all templates. You can use the session helper just like you would use the `Session` class.

```python
{% if session().has('success') %}
    <div class="alert alert-success">
        {{ session().get('success') }}
    </div>
{% endif %}
```

You could use this to create a simple Jinja include template that can show success, warning or error messages. Your template could be located inside a `resources/templates/helpers/messages.html`:

```python
{% if session().has('success') %}

    <div class="alert alert-success">
        {{ session().get('success') }}
    </div>
    
{% elif session().has('warning') %}

    <div class="alert alert-warning">
        {{ session().get('warning') }}
    </div>
    
{% if session().has('danger') %}

    <div class="alert alert-danger">
        {{ session().get('danger') }}
    </div>
    
{% endif %}
```

Then inside your working template you could add:

```html
{% include 'helpers/messages.html' %}
```

Then inside your controller you could do something like:

```python
def show(self):
    return request().redirect('/dashboard') \
        .session.flash('success', 'Action Successful!')
```

or as separate statements

```python
def show(self, Session):
    Session.flash('success', 'Action Successful!')

    return request().redirect('/dashboard')
```

Which will show the correct message and message type. 


## Resetting Data

You can reset both the flash data and the session data through the `reset` method.

To reset just the session data:

```python
def show(self, Session):
    Session.reset()
```

Or to reset only the flash data:

```python
def show(self, Session):
    Session.reset(flash_only=True)
```

Remember that Masonite will reset flashed data at the end of a successful `200 OK` requests anyway so you will most likely not use the `flash_only=True` keyword parameter.