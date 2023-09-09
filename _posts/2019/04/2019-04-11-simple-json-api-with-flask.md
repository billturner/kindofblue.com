---
layout: post
title: 'Simple JSON API with Python and Flask'
date: 2019-04-11 19:00:00 -0500
description: 'A tutorial covering the bare minimum for creating a simple JSON API using Python and Flask'
tags:
  - flask
  - json
  - programming
  - pytest
  - python
---

This should be the first of a few blog posts explaining how to get a simple API up and running with Python, and using the Flask framework. I'll try and link all these together once I get more up. And if you'd like the source code at any time, you can grab it here: [github.com/billturner/simple_flask_api](https://github.com/billturner/simple_flask_api 'Github repository for this tutorial'). The branch for this part will be `01-bare-bones`.

I'll be using Python 3.x, and I'm going to assume you have it all set up and in your path. If you don't have it set up, you can find a downloadable installer [on the Python web site](https://www.python.org/downloads/ 'The Python downloads page.').

#### Setting up the virtual environment and installing Flask

We want to create a directory for our Python files, so make one of your own choosing; I used `simple_flask_api`. Change into that directory, and from here on out all files should go in here. I'll use the virtual environment ability built into newer versions of Python, and I'll call the environment `venv`:

```bash
python -m venv venv
```

To enter the virtual environment, you'll need to run the appropriate initialization script. Since I'm on Windows, I'll be executing this command in PowerShell, so what I use may be different than you. For most Linux and OS X users, you'll use another command which I'll list below (for other platforms, there are [instructions in the Python docs](https://docs.python.org/3/library/venv.html 'Virtual environment docs on Python web site')):

```bash
# for Windows/PowerShell users, you should run:
.\venv\Scripts\Activate.ps1
# for most Linux and OS X users, you should run:
source ./venv/bin/activate
```

Now, you should have a prompt that has `(venv)` at the beginning, like this (the `...` is just me shortening the path):

```bash
(venv) PS C:\Users\...\simple_flask_api>
```

For the rest of this tutorial, we'll assume you're in this directory and have activated the virtual environmen. Once you have activated the virtual environment, you can install the first of the two libraries we'll need for this part of the tutorial, Flask:

```bash
pip install flask
```

You should see some text scroll by on the screen as the library (and its dependents) are installed. Now we're ready to build an API.

#### Hello world

A tutorial isn't a tutorial without a "hello world" example, so let's write up a quick one (a version of which is in a _lot_ of Flask tutorials). Create a new file named `app.py` and place the following code in it:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/", methods=['GET'])
def hello():
    return 'Hello world!'

if __name__ == '__main__':
    app.run(debug=True)
```

What we're doing here is importing from the Flask library, initializing the app with Flask, and then creating a simple route, and a function to be called when that route is called. For this tutorial, we are only going to be fetching data from our API, so the only HTTP method we'll need is `GET`.

We're adding `debug=True` to `app.run()` so that when you make changes to the source code, it will automatically reload the app. Now, to start the app running, execute this command where we've been building this:

```bash
python app.py
```

You should get some feedback from Flask starting up, and then a URL you can use to access the app in the browser. Should be something like `http://127.0.0.1:5000/`. If all went well, then you should see the string "Hello world!" in your browser.

But wait, we're building an API, and a basic string isn't helpful when building an API, JSON is. So, let's also import the `jsonify` utility from Flask and then return a JSON object from our `hello()` function (I'm only showing the changed lines here, and the `[...]` signifies code that doesn't change):

```python
from flask import Flask, jsonify
[...]
@app.route("/", methods=['GET'])
def hello():
    return jsonify({'message': 'Hello world!'})
[...]
```

With that change, and reloading your browser, you should have a JSON object response, instead of a string:

```json
{
  "message": "hello world!"
}
```

We now have a JSON API! It doesn't do much at the moment, but it's a start.

#### Adding mock data

Create a file named `mock_data.py` and place [the contents of this gist](https://gist.github.com/billturner/a3c73427215697286a5f21d1ad028002) into it. Since the end goal of teaching myself Flask is so that I can [get Lists of Bests up and running again]({{site.url}}/2017/11/how-ill-build-the-new-lists-of-bests 'An earlier post where I talk about Lists of Bests'), we'll use some fake data for possible book and album lists for the site. We're including it in a separate file just to keep `app.py` as simple as possible.

At the top of `app.py`, we'll pull the mock data into our app:

```python
from flask import Flask
from mock_data import lists
```

And now that we have the lists, let's add a new route to handle displaying the lists. Add the following after our `hello()` function:

```python
[...]
@app.route("/lists", methods=['GET'])
def get_lists():
    return jsonify({'lists': lists})
[...]
```

Once the app reloads, if we now try to access `http://localhost:5000/lists` in the browser, we should get a JSON representation of the lists from our mock data file, that looks like this:

```json
{
  "lists": [
    {
      "description": "This is  the description for the first list",
      "id": 1,
      "name": "This is an example list",
      "type": "albums"
    },
    {
      "description": "A description for this list should be different",
      "id": 2,
      "name": "Another list",
      "type": "books"
    }
  ]
}
```

#### Getting a specific list by id

In order to get lists by a passed in `id`, we will need a new route and handling function. Let's add this below our last one for all the lists:

```python
[...]
@app.route("/lists/<int:list_id>", methods=['GET'])
def get_list(list_id):
    _list = [_list for _list in lists if _list['id'] == list_id][0]
    return jsonify({'list': _list})
[...]
```

I'm using `_list` for the variable name above, since `list` is a reserved word and can cause problems. And since we're only getting a single list, I'm grabbing the first of the array with `[0]`. This is brittle, but will work for our simple case where we know exactly what the data looks like.

Once saved, you can now access this by passing a valid `id` to the URL as the last part of the URL, like `http://localhost:5000/lists/1`.

We now have a list, but where are the list items that we added to the mock data? Let's make sure we're including the `list_items` in our import statement, and adjust our function just a bit and add the proper list items into the response:

```python
[...]
from mock_data import lists, list_items
[...]
@app.route("/lists/<int:list_id>", methods=['GET'])
def get_list(list_id):
    _list = [_list for _list in lists if _list['id'] == list_id][0]
    _items = [_item for _item in list_items if _item['list_id'] == list_id]
    if len(_items) > 0:
        _list['list_items'] = _items
    return jsonify({'list': _list})
[...]
```

After saving and reloading, the output should look like this:

```json
{
  "list": {
    "description": "This is  the description for the first list",
    "id": 1,
    "list_items": [
      {
        "creator": "Radiohead",
        "id": 1,
        "list_id": 1,
        "title": "OK Computer"
      },
      {
        "creator": "Miles Davis",
        "id": 2,
        "list_id": 1,
        "title": "Kind of Blue"
      },
      {
        "creator": "Thelonius Monk",
        "id": 3,
        "list_id": 1,
        "title": "Monk Alone"
      }
    ],
    "name": "This is an example list",
    "type": "albums"
  }
}
```

Nice! Now we have a list and all its items. We could probably clean things up a bit - like removing the `list_id` attribute from the items - but this works just fine for our basic example.

#### Testing

Since we're good little developers, it would probably be a good idea to write some tests to validate that the API is working as expected. We'll be using pytest as the library, so let's install that:

```bash
pip install pytest
```

The Flask documentation has [some examples of testing using pytest](http://flask.pocoo.org/docs/1.0/testing/), so we'll use those as reference while making our own. To start, create a file named `test_app.py` in the same directory as the other files. First, we'll need to import `pytest` and our app:

```python
import pytest
from app import app
```

In order to reduce a bit of extra typing, let's write a test fixture for the app client used for testing:

```python
[...]
@pytest.fixture
def client():
    return app.test_client()
```

We're not saving a lot of typing here, but later if you have to initialize a database, or set up anything else, this fixture would be a good place to do that. First, let's write a super-simple test - that makes use of our `client` fixture - to make sure we're getting the right kind of HTTP response code, and MIME type back from our JSON API server:

```python
[...]
def test_json_with_proper_mimetype(client):
    response = client.get('/')
    assert response.status_code == 200
    assert response.content_type == 'application/json'
```

Now, in order to run this our first test, run the `pytest` command at the console:

```bash
pytest
```

And you should get some output that looks like this (minus a bunch of long lines like `===========`):

```text
platform win32 -- Python 3.7.1, pytest-4.4.0, py-1.8.0, pluggy-0.9.0
rootdir: C:\Users\...\simple_flask_api
collected 1 item

test_app.py .                                [100%]

1 passed in 0.18 seconds
```

The little dot next to `test_app.py` should be green, signifying its passing. You'll get a green dot for each passing test, and if you get an error, it will give you a wordy explanation telling you what went wrong.

With our first test passing, let's add one where we test the JSON in the response. Here's one that checks that the JSON from our root URL is correct - and we'll use the `get_json()` helper method to parse the JSON for us:

```python
[...]
def test_hello_world(client):
    response = client.get('/')
    json = response.get_json()
    assert json['message'] == 'Hello world!'
```

If you run `pytest` again, you should now have 2 green dots. To finish this part up, let's test that we're getting the right number lists from our `/lists` API endpoint, and that the endpoint for a single list also returns the list items:

```python
[...]
def test_lists(client):
    response = client.get('/lists')
    json = response.get_json()
    assert len(json['lists']) == 2

def test_single_list(client):
    response = client.get('/lists/1')
    json = response.get_json()
    assert json['list']['name'] == 'This is an example list'

def test_single_list_items(client):
    response = client.get('/lists/1')
    json = response.get_json()
    assert len(json['list']['list_items']) == 3
```

Running `pytest` now should show that we have 5 successful tests.

#### Catching errors

Right now, if you try to access any URL outside of the ones we've specified, you get Flask's generic 404 HTML error page. It would be nice to customize this a bit to return a JSON error since this is an API service and not a regular web site.

First, let's write an `errorhandler` for 404 errors to display them as JSON. In `app.py`, you can place this before the `if __name__ == '__main__'` line:

```python
[...]
@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': '404 Not Found'}), 404
[...]
```

Now, whenever Flask would normally handle a 404 error, instead of returning its built in generic HTML page, it will now return this JSON response. If you try and load a non-existent URL - like `http://localhost:5000/fakeurl` in our app, you should now get our JSON error.

```json
{
  "error": "404 Not Found"
}
```

And now that we have a test file all set up, let's write one to validate what we're seeing in the browser. You can add this test below all the others:

```python
def test_not_found(client):
    response = client.get('/not/real/url')
    json = response.get_json()
    assert response.status_code == 404
    assert json['error'] == '404 Not Found'
```

And that should pass like all the other tests. Finally, there is one more case I'd like to test: what if you try and access a list that doesn't exist. We know we have lists with `id` of `1` and `2`, but what happens if we try for list `3` with `http://localhost:5000/lists/3`? If you do try it out, you'll get a nasty Flask error message that says `IndexError: list index out of range` at the top. We should account for lists not being available.

We can use the built-in `abort()` method to trigger a not found error if we try and access a list that doesn't exist. First, we'll add `abort` to our import statement, and then update our `get_list()` function to trigger this when we can't find the list:

```python
from flask import Flask, jsonify, abort
[...]
def get_list(list_id):
    _list = {}
    search = [_list for _list in lists if _list['id'] == list_id]
    if len(search) > 0:
        _list = search[0]
    else:
        abort(404)
    _items = [_item for _item in list_items if _item['list_id'] == list_id]
    if len(_items) > 0:
        _list['list_items'] = _items
    return jsonify({'list': _list})
[...]
```

We had to add a bit more code to capture the case where we don't find a list based on our data, but not too much. Now, if you try accessing a list that doesn't exist, then you'll get that handy 404 error we created above. And like before, let's add a test to validate the changes. Again, just add to the bottom of the `test_app.py` file:

```python
def test_list_not_found(client):
    response = client.get('/lists/99')
    json = response.get_json()
    assert response.status_code == 404
    assert json['error'] == '404 Not Found'
```

When you run `pytest` now, you'll see a new green dot for a passing test, but also this helps show that the adjustment of our `get_list()` function still works as expected.

#### Wrapping up

Even in building our _very_ basic JSON API, it does take a bit of set up to get things started. And at this point we're not even accessing a database, or allowing updates to the API with POST and PUT requests. However, something like this tutorial was what I initially looked for; I just wanted the absolute basics. And this basic tutorial gives me a good base to build more complex features upon.

And it goes without saying that some of the code above may not be the best, but I'm sure I'll learn to write more idiomatic Python as I continue.

I think that's about it for this part of the tutorial. For the next one, I'll likely pick one of the API libraries - [RESTful](https://flask-restful.readthedocs.io/en/latest/) or [RESTPlus](https://flask-restplus.readthedocs.io/en/stable/) - and see how much of a difference they make.

If you have any feedback or questions about this tutorial, [please let me know]({{site.url}}/about.html 'My About page, which includes contact info').

#### Resources

Some resources that helped out in this first part of the tutorial (in addition to the documentation already linked above):

- [Flask Mega-tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) and [Designing a RESTful API with Python and Flask](https://blog.miguelgrinberg.com/post/designing-a-restful-api-with-python-and-flask) by Miguel Grinberg
- [Developing RESTful APIs with Python and Flask](https://auth0.com/blog/developing-restful-apis-with-python-and-flask/) by Bruno Krebs
- [Creating APIs with Python and Flask](https://programminghistorian.org/en/lessons/creating-apis-with-python-and-flask) by Patrick Smyth
