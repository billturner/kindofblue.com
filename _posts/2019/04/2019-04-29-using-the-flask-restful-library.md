---
layout: post
title: 'Using the Flask-RESTful library'
date: 2019-04-29 15:21:00 -0500
description: 'An update to our previous post, upgrading our API to use the Flask-RESTful library'
tags:
  - flask
  - json
  - programming
  - pytest
  - python
---

In [my last post]({{site.url}}/2019/04/simple-json-api-with-flask/), I wired up a very simple API server in Python with Flask. This time, I'll add in a library meant to make building APIs a bit easier: [Flask-RESTful](https://flask-restful.readthedocs.io/en/latest/).

The goal will be to add in the library and adjust all code necessary to take advantage of this library. If all goes well, then the tests we wrote the last time should still pass. Let's cross our fingers.

The code for this time will be in the same repository, [github.com/billturner/simple_flask_api](https://github.com/billturner/simple_flask_api), but this time the branch used will be `02-flask-restful`.

#### Installing Flask-RESTful

Like you did in the first tutorial, initialize the virtual environment, and then let's install the library:

```bash
pip install flask-restful
```

Many of the library's dependencies will already be installed, so the process should go pretty quickly.

#### Configuration

Since we'll be updating our `app.py` file in chunks, we'll leave the existing library import statements as they are, and then add what we need from `flask-restful`. Thankfully, there's a handy [quickstart guide](https://flask-restful.readthedocs.io/en/latest/quickstart.html) on the library's doc site, which is just about all we'll need to reference for our changes.

Open up our `app.py` file, and add the following below the existing `from flask import Flask` line (again, the `[...]` signifies code that doesn't change):

```python
[...]
from flask_restful import Resource, Api
[...]
```

Then, underneath where we assign the `app` variable, we'll add a new `api` variable that `flask-restful` will use:

```python
[...]
api = Api(app)
[...]
```

#### Updating our first route

Let's start out updating our useless 'hello world' route since it's the simplest one. Delete - or comment out - the existing code for the hello world route, and then add the following:

```python
[...]
class HelloWorld(Resource):
    def get(self):
        return {'message': 'Hello world!'}
api.add_resource(HelloWorld, '/')
[...]
```

Save the file, if we start up the app from the command line, we should be able to try out our new 'hello world' route. As a reminder, you start up the app at the command line like this:

```bash
python app.py
```

And once that starts up, point your web browser to `http://localhost:5000/` and you should still get the familiar JSON 'hello world' message:

```json
{
  "message": "Hello world!"
}
```

Success! Now, just for sanity's sake, let's run the tests and see if they all still run clean. Like before, run the `pytest` command at your prompt to execute the test suite. All the tests should pass just like before.

#### What we changed, and why

The biggest change here was taking the previous `@app.route()` decorator and its function away, and replacing it with a class called `HelloWorld` - which inherits from the `flask-restful` `Resource`. Then we specified a `get()` function in the class to handle our GET request; which in our case, just returns a simple JSON object. Next, we used the `add_resource` method to add our route to the `api` object.

To be honest, at this point I'm not sure I'm seeing a major benefit to using the library. Of course, we just updated a basic GET route, but we didn't really reduce much in the way of lines of code, although we _were_ able to get by without using the `jsonify` method from the main `flask` library.

However, one benefit I do see that could become useful in the future is that each route now has its own `class` for defining what happens. With this, we can easily extract that code into a separate file, one for each route if we wanted to. When you start to build a more complex API service, this could be quite handy.

#### Updating the remaining routes

Similar to how we replaced the 'hello world' root route, let's do the same for the main list listing route:

```python
class GetLists(Resource):
    def get(self):
        return {'lists': lists}
api.add_resource(GetLists, '/lists')
```

Before moving forward, quickly run the tests, and see if that route is working as it should. Then, update the single list route:

```python
class GetList(Resource):
    def get(self, list_id):
        _list = {}
        search = [_list for _list in lists if _list['id'] == list_id]
        if len(search) > 0:
            _list = search[0]
        else:
            abort(404)
        _items = [_item for _item in list_items if _item['list_id'] == list_id]
        if len(_items) > 0:
            _list['list_items'] = _items
        return {'list': _list}
api.add_resource(GetList, '/lists/<int:list_id>')
```

There are a few more changes here. You'll notice now we add the `list_id` parameter to the `get` function. Otherwise, the code is pretty much the same. Let's run the tests now that we have the single list route added.

Uh oh! If you're like me, you got an error when running `pytest`:

```text
>       assert json['error'] == '404 Not Found'
E       KeyError: 'error'

test_app.py:53: KeyError
```

It appears that the `abort(404)` we're using when a list isn't found does not work like it used to. Luckily, all the other tests **did** pass without problems.

#### What happened with `abort()`

If you add in a `print(json)` line in the failing test - before the `assert` line, you'll see that we _do_ get a 'not found' message, but it's just not the one we set up:

```json
{
  "message": "The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again."
}
```

So, where is this error coming from? If you search for that error message, we find it comes from the `werkzeug` library (and `werkzeug.exceptions` specifically). This library is a requirement of the main `flask` library. That's fine, but it'd be nice to know why this is overriding the `abort()` we're specifically calling.

From looking at one of the main `flask_restful` files ([**init**.py](https://github.com/flask-restful/flask-restful/blob/master/flask_restful/__init__.py)), it's overriding the default `flask` `abort()` method with its own version. I don't really want to spend too much time digging into the chain of order, so I think we'll just adjust to using the error handling provided by the `flask_restful` library to make things a bit more simple.

#### Updating our error handling

To keep things _fairly_ simple, we're going to change as little as possible to get the tests working again. First, let's update our imports at the top of `app.py` to remove importing `abort` from `flask` and instead, import it from `flask_restful`:

```python
from flask import Flask, jsonify
from flask_restful import Resource, Api, abort
[...]
```

Next, update the `get` function in the `GetList` class to use the different `abort` syntax:

```python
abort(404, error='404 Not Found')
```

Since the general 404 error handling is working fine, we don't need to update that at all. After saving, and re-running `pytest` at the command line, you should have an error-free output.

#### Wrapping up

That didn't require too many changes, and having the test suite in place definitely made validating the changes made quite easy. I'm curious if the other libraries are just as simple to use.

Next time, I think I'll take a look at the [Flask-RESTPlus library](https://flask-restplus.readthedocs.io/en/stable/), and see how it works with our simple API service.

#### Resources

For this post, the only extra resource needed was the [Flask-RESTful documentation](https://flask-restful.readthedocs.io/en/latest/), which covered our use basic case.
