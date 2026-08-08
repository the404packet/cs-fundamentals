## Flask

* Flask is a lightweight Python web framework.
* Flask uses **WSGI (Web Server Gateway Interface)**.
* Flask uses **Jinja2** as its template engine.

---

# WSGI

**WSGI = Web Server Gateway Interface**

It acts as an interface between:

```text
Client
   |
Web Server
   |
WSGI
   |
Flask Application
```

The Flask application contains the business logic, but there needs to be a standard way through which the web server communicates with the Python application.

That interface is **WSGI**.

---

# Basic Flask Application

```python
from flask import Flask

app = Flask(__name__)
```

`Flask(__name__)` creates the Flask application.

`__name__` tells Flask where the application is located so that it can find:

* templates
* static files
* other resources

---

## Basic Route

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def welcome():
    return "Hello World!"


if __name__ == '__main__':
    app.run()
```

`@app.route('/')`

* Decorator used to bind a URL to a Python function.

Example:

```python
@app.route('/hello')
def hello():
    return "Hello"
```

Opening:

```text
http://localhost:5000/hello
```

calls:

```python
hello()
```

---

# Running Flask

```python
if __name__ == '__main__':
    app.run()
```

This is the starting point of the program.

---

## Debug Mode

```python
app.run(debug=True)
```

`debug=True`:

* Automatically reloads server when code changes.
* Shows detailed error messages.
* Useful during development.

Do not normally enable debug mode in production.

---

# Dynamic Routes

We can pass values through URLs.

```python
@app.route('/pass/<int:score>')
def passed(score):
    return "The passed score is " + str(score)
```

Example URL:

```text
/pass/80
```

Output:

```text
The passed score is 80
```

---

## Route Variable Types

```python
<string:name>
<int:id>
<float:price>
<path:path>
```

Example:

```python
@app.route('/user/<string:name>')
def user(name):
    return f"Hello {name}"
```

---

# redirect() and url_for()

Import:

```python
from flask import redirect, url_for
```

`redirect()` redirects the user to another route.

`url_for()` generates the URL corresponding to a function.

Example:

```python
@app.route('/pass/<int:score>')
def passed(score):
    return "Passed with score " + str(score)


@app.route('/fail/<int:score>')
def failed(score):
    return "Failed with score " + str(score)
```

Now:

```python
@app.route('/result/<int:score>')
def result(score):

    if score >= 50:
        result = "passed"
    else:
        result = "failed"

    return redirect(
        url_for(result, score=score)
    )
```

Important:

```python
url_for('passed')
```

uses the **function name**, not the URL.

Example:

```python
@app.route('/pass/<int:score>')
def passed(score):
```

Here:

```python
url_for('passed')
```

refers to:

```python
def passed()
```

---

# Jinja2

Jinja2 is Flask's template engine.

It combines:

```text
HTML Template
+
Data from Flask
=
Dynamic Web Page
```

Example:

Python:

```python
return render_template(
    'index.html',
    name="Somanshu"
)
```

HTML:

```html
<h1>Hello {{ name }}</h1>
```

Output:

```text
Hello Somanshu
```

---

# render_template()

Import:

```python
from flask import render_template
```

Flask expects HTML templates inside the:

```text
templates/
```

folder.

Folder structure:

```text
project/
│
├── app.py
│
├── templates/
│   ├── index.html
│   └── result.html
│
└── static/
    ├── css/
    │   └── style.css
    └── js/
        └── script.js
```

Example:

```python
@app.route('/')
def home():
    return render_template('index.html')
```

---

# Passing Data to HTML

Python:

```python
@app.route('/')
def home():

    name = "Somanshu"
    age = 21

    return render_template(
        'index.html',
        name=name,
        age=age
    )
```

HTML:

```html
<h1>{{ name }}</h1>

<p>Age: {{ age }}</p>
```

---

# Jinja2 Syntax

## Expression

Used to print values.

```jinja2
{{ variable }}
```

Example:

```html
<h1>{{ name }}</h1>
```

---

## Statements

Used for:

* if
* for
* blocks
* template inheritance

Syntax:

```jinja2
{% statement %}
```

Example:

```jinja2
{% if marks >= 50 %}

{% endif %}
```

---

## Comments

```jinja2
{# This is a Jinja comment #}
```

---

# Jinja2 if

Python:

```python
@app.route('/result')
def result():

    marks = 80

    return render_template(
        'result.html',
        marks=marks
    )
```

HTML:

```html
{% if marks >= 90 %}

    <h2>Grade A+</h2>

{% elif marks >= 75 %}

    <h2>Grade A</h2>

{% elif marks >= 50 %}

    <h2>Passed</h2>

{% else %}

    <h2>Failed</h2>

{% endif %}
```

Every `if` must end with:

```jinja2
{% endif %}
```

---

# Jinja2 for Loop

Python:

```python
@app.route('/students')
def students():

    students = [
        "Aman",
        "Rahul",
        "Riya"
    ]

    return render_template(
        'students.html',
        students=students
    )
```

HTML:

```html
<ul>

{% for student in students %}

    <li>{{ student }}</li>

{% endfor %}

</ul>
```

Every `for` loop ends with:

```jinja2
{% endfor %}
```

---

# Looping Through List of Dictionaries

Python:

```python
users = [
    {
        "name": "Aman",
        "age": 21
    },
    {
        "name": "Rahul",
        "age": 22
    }
]

return render_template(
    'users.html',
    users=users
)
```

HTML:

```html
{% for user in users %}

    <h3>{{ user.name }}</h3>

    <p>{{ user.age }}</p>

{% endfor %}
```

Dictionary values can also be accessed using:

```jinja2
{{ user['name'] }}
```

---

# if Inside for Loop

```html
{% for user in users %}

    <h3>{{ user.name }}</h3>

    {% if user.age >= 18 %}

        <p>Adult</p>

    {% else %}

        <p>Minor</p>

    {% endif %}

{% endfor %}
```

---

# Jinja Loop Variables

Jinja provides a special:

```text
loop
```

object.

Example:

```html
{% for student in students %}

    {{ loop.index }}. {{ student }}

{% endfor %}
```

Useful values:

```jinja2
loop.index
```

Starts from `1`.

```jinja2
loop.index0
```

Starts from `0`.

```jinja2
loop.first
```

True for first iteration.

```jinja2
loop.last
```

True for last iteration.

---

# while Loop in Jinja2

Jinja2 normally does **not support while loops**.

Instead of performing complicated loops inside HTML:

```text
Do logic in Python
        ↓
Pass processed data to Jinja
        ↓
Use for loop to display it
```

Example:

Python:

```python
numbers = range(5)

return render_template(
    'index.html',
    numbers=numbers
)
```

HTML:

```html
{% for number in numbers %}

    <p>{{ number }}</p>

{% endfor %}
```

Good principle:

```text
Business Logic → Python
Presentation Logic → Jinja2
```

---

# HTML Forms with Flask

Example HTML:

```html
<form action="/submit" method="POST">

    <input
        type="number"
        name="science"
    >

    <input
        type="number"
        name="maths"
    >

    <button type="submit">
        Submit
    </button>

</form>
```

The important attribute is:

```html
name="science"
```

because Flask uses this name to access the value.

---

# request

Import:

```python
from flask import request
```

`request` contains information about the incoming HTTP request.

Example:

```python
@app.route('/submit', methods=['GET', 'POST'])
def submit():

    if request.method == 'POST':

        science = float(
            request.form['science']
        )

        maths = float(
            request.form['maths']
        )

        total = (science + maths) / 2

        return str(total)

    return render_template('index.html')
```

---

# request.form

Used to read form data sent using POST.

HTML:

```html
<input
    type="number"
    name="science"
>
```

Python:

```python
science = request.form['science']
```

The value of:

```html
name=""
```

is used as the key.

---

# GET and POST

Route can specify allowed HTTP methods.

```python
@app.route(
    '/submit',
    methods=['GET', 'POST']
)
```

Check request method:

```python
if request.method == 'POST':
```

---

## GET

Usually used to:

* retrieve data
* load pages
* send simple query parameters

Example:

```text
/search?name=python
```

---

## POST

Usually used to:

* submit forms
* create data
* send sensitive or larger request bodies

Example:

```html
<form method="POST">
```

---

# Complete Form Example

## app.py

```python
from flask import (
    Flask,
    render_template,
    request,
    redirect,
    url_for
)

app = Flask(__name__)


@app.route('/')
def home():
    return render_template('index.html')


@app.route('/pass/<float:score>')
def passed(score):

    return render_template(
        'result.html',
        score=score,
        result="PASS"
    )


@app.route('/fail/<float:score>')
def failed(score):

    return render_template(
        'result.html',
        score=score,
        result="FAIL"
    )


@app.route('/submit', methods=['POST'])
def submit():

    science = float(
        request.form['science']
    )

    maths = float(
        request.form['maths']
    )

    total = (
        science + maths
    ) / 2

    if total >= 50:

        return redirect(
            url_for(
                'passed',
                score=total
            )
        )

    return redirect(
        url_for(
            'failed',
            score=total
        )
    )


if __name__ == '__main__':
    app.run(debug=True)
```

---

# index.html

```html
<!DOCTYPE html>

<html>

<head>

    <title>
        Marks Calculator
    </title>

</head>

<body>

    <h1>
        Enter Marks
    </h1>

    <form
        action="{{ url_for('submit') }}"
        method="POST"
    >

        <label>
            Science
        </label>

        <input
            type="number"
            name="science"
            required
        >

        <br><br>

        <label>
            Maths
        </label>

        <input
            type="number"
            name="maths"
            required
        >

        <br><br>

        <button type="submit">
            Submit
        </button>

    </form>

</body>

</html>
```

---

# result.html

```html
<!DOCTYPE html>

<html>

<body>

    <h1>
        Result: {{ result }}
    </h1>

    <h2>
        Score: {{ score }}
    </h2>

</body>

</html>
```

---

# Static Files

Flask stores:

* CSS
* JavaScript
* Images

inside:

```text
static/
```

Example structure:

```text
static/
│
├── css/
│   └── style.css
│
├── js/
│   └── script.js
│
└── images/
    └── logo.png
```

---

# Adding CSS

File:

```text
static/css/style.css
```

Example:

```css
body {
    font-family: Arial;
    background-color: #f5f5f5;
}

h1 {
    text-align: center;
}
```

Connect CSS in HTML:

```html
<link
    rel="stylesheet"
    href="{{ url_for(
        'static',
        filename='css/style.css'
    ) }}"
>
```

Common shorter form:

```html
<link
    rel="stylesheet"
    href="{{ url_for('static', filename='css/style.css') }}"
>
```

---

# Adding JavaScript

File:

```text
static/js/script.js
```

Example:

```javascript
function hello() {
    alert("Hello");
}
```

Connect JS:

```html
<script
    src="{{ url_for('static', filename='js/script.js') }}">
</script>
```

Now HTML can use:

```html
<button onclick="hello()">
    Click
</button>
```

---

# Adding Images

Image:

```text
static/images/logo.png
```

HTML:

```html
<img
    src="{{ url_for('static', filename='images/logo.png') }}"
>
```

---

# Passing Flask Data to JavaScript

Python:

```python
@app.route('/')
def home():

    username = "Somanshu"

    return render_template(
        'index.html',
        username=username
    )
```

HTML:

```html
<script>

const username =
    {{ username | tojson }};

console.log(username);

</script>
```

`tojson` safely converts Python values into JavaScript-compatible JSON.

---

# Template Inheritance

Template inheritance avoids writing common HTML repeatedly.

Example:

```text
Navbar
Footer
CSS
JS
Page structure
```

can be stored in:

```text
base.html
```

---

# base.html

```html
<!DOCTYPE html>

<html>

<head>

    <title>

        {% block title %}

            Flask App

        {% endblock %}

    </title>

    <link
        rel="stylesheet"
        href="{{ url_for('static', filename='css/style.css') }}"
    >

</head>

<body>

    <nav>

        <a href="{{ url_for('home') }}">
            Home
        </a>

    </nav>


    {% block content %}

    {% endblock %}


    <script
        src="{{ url_for('static', filename='js/script.js') }}">
    </script>

</body>

</html>
```

---

# Extending Template

`index.html`

```html
{% extends "base.html" %}


{% block title %}

Home

{% endblock %}


{% block content %}

<h1>
    Welcome
</h1>

<p>
    This is the home page.
</p>

{% endblock %}
```

Important Jinja keywords:

```jinja2
{% extends "base.html" %}
```

Used to inherit another template.

```jinja2
{% block content %}
```

Defines a replaceable section.

```jinja2
{% endblock %}
```

Ends the block.

---

# Important Flask Imports

```python
from flask import Flask
```

Create Flask application.

```python
from flask import render_template
```

Render HTML templates.

```python
from flask import request
```

Read incoming request data.

```python
from flask import redirect
```

Redirect to another URL.

```python
from flask import url_for
```

Generate URL from Flask function name.

---

# Important Revision

```text
Flask
│
├── WSGI
│
├── Routing
│   └── @app.route()
│
├── Jinja2
│   ├── {{ expression }}
│   ├── {% statement %}
│   └── {# comment #}
│
├── Templates
│   └── templates/
│
├── Static Files
│   ├── CSS
│   ├── JS
│   └── Images
│
├── Request
│   ├── request.method
│   └── request.form
│
├── Response
│   ├── return
│   └── render_template()
│
└── Navigation
    ├── redirect()
    └── url_for()
```

---

# Key Interview Points

**What is Flask?**

Flask is a lightweight Python web framework used to build web applications and REST APIs.

---

**What is WSGI?**

WSGI is a standard interface that allows a web server to communicate with a Python web application.

---

**What is Jinja2?**

Jinja2 is Flask's template engine used to generate dynamic HTML pages by combining HTML templates with data from Python.

---

**Difference between `{{ }}` and `{% %}`?**

```jinja2
{{ }}
```

Used to display expressions or variables.

```jinja2
{% %}
```

Used for control statements such as `if`, `for`, `block`, etc.

---

**What does `url_for()` do?**

Generates a URL using the Flask view function name.

---

**Why use `render_template()`?**

It renders an HTML file from the `templates` directory and allows Flask to pass dynamic data to it.

---

**Where are CSS and JS stored?**

Inside the:

```text
static/
```

directory.

---

**What does `debug=True` do?**

* automatically reloads Flask when code changes
* provides detailed debugging information

Used during development.
