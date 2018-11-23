---
layout: post
title:  "What to do when you need a web app quickly"
description:  "A quick guide to Flask"
date:   2018-10-28 21:30:05 +0200
categories: flask numpy
cover_image: https://images.unsplash.com/photo-1531083894382-04a4f09c998f?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=5378eb57b9dafc68991c2251fcd05bac&auto=format&fit=crop&w=2250&q=80
---

Recently, I needed a piece of software where you can input some numbers, validate and process them, and print some result. Since I needed to do scientific computing with it I decided to use Python with [NumPy](http://www.numpy.org/) and [SciPy](https://www.scipy.org/) and because I needed validation I decided that HMTL input validations would be the easiest to use.

Then I had a choice from a ton of [Python web frameworks](https://wiki.python.org/moin/WebFrameworks) and since “microframework” sounded like exactly what I need and I’ve heard the name before I decided to give [Flask](http://flask.pocoo.org/) a try.

## Getting started

All you need to get started after you’ve [installed Flask](http://flask.pocoo.org/docs/1.0/installation/) is only 1 python file.

``` python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def index():
    return "Hello there!"
```

Start it with a simple command:

``` bash
FLASK_APP=app.py FLASK_ENV=development flask run
```

That’s all you need to render a simple string on the “/” page. But who the fuck needs to render a string? Even if all you need is only to render a string it would look awful without some CSS. So we’ll go deeper and get into rendering templates.

## Rendering templates

In case you want to render a template you can use a built-in method called, you guessed it, `render_template`.

``` python
from flask import Flask, render_template
app = Flask(__name__)

@app.route("/")
def index():
    return render_template('index.html')
```

You’ll also need to create `index.html` page in `/templates` folder.

``` html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Computer math algorithms</title>
</head>
<body>
  <h1>Computer math algorithms</h1>
  <ul>
    <li><a href="/equations">System of linear equations</a></li>
  </ul>
</body>
</html>
```

As you can see, I’ve added a link to `/equations`, but if you click the link now all you’ll see is “404 Not Found” page. To fix this you’ll need to add another route and another template the same way we did before.

``` python
@app.route("/equations")
def equations():
    return render_template('equations.html')
```

## The first form

The current project is going to solve a system of linear equations. First of all, we need to know how many equations there are. To get that number we can use a form with one number input.

``` html
<form action="/equations">
  <label for="equations_number">Number of equations:</label>
  <input type="number" name="equations_number" required min="1" max="100">
  <br>
  <input type="submit" value="Submit">
</form>
```
By the way, for some simple CSS I recommend reading [a great website](http://bettermotherfuckingwebsite.com/). To find CSS used there you can use developers tools.

![](https://cdn-images-1.medium.com/max/2728/1*drvZxl2vP8LENJ3Q1Z5Ucw.png)

This form can’t be submitted without number of equations because of the required attribute and on the input tag. Also, the number cannot be less than 1 and more than 100 because of `min` and `max` attributes respectively.

The number could be bigger than 100, but the website won’t be usable (it’s not really usable after 10 already). In that case, I would add import from some sort of spreadsheet file.

After entering any valid number, click on “Submit” button and a parameter will be added to the URL `/equations?equations_number=2`.

## Dealing with arguments

To read that parameter on the backend `request.args` can be used like so:

``` python
from flask import request

@app.route("/equations")
def equations():
    equations_number = request.args.get('equations_number', type = int)
    return render_template('equations.html', equations_number=equations_number)
```

Here we’re passing a name of the needed parameter to request.args.get and a type so that it converts to integer. equations_number argument is then passed to render_template so that it is available in the template.

## Is this a system of linear equations?

All we need for the system of equations is a matrix of coefficients `A` and a column vector `b` so that the equation would be `Ax=b`. Example:

```
x + 2y = 3
4x + 5y = 6
```

In this case, we would have the following `A` and `b`:

```
A = [[1, 2], [4, 5]]
b = [[3], [6]]
```

## Making a form with a two-dimensional array input

Right now we need a form for `A` and `b`.

``` html
{% raw %}
<form action="/equations">
  {% for i in range(equations_number) %}
    {% for j in range(equations_number) %}
      <label for="a">A[{{i+1}}, {{j+1}}]:</label>
      <input type="number" name="a" class="matrix-number matrix-number--a" required>
    {% endfor %}
    <span class="b">
      <label for="b">b[{{i}}]:</label>
      <input type="number" name="b" class="matrix-number" required>
    </span>
    <br>
  {% endfor %}
  <input type="submit" value="Solve">
</form>
{% endraw %}
```

What this code does is it creates a form and `equations_number` times renders input for each equation. And each equation needs `equations_number` coefficients (that’s what for loop with `j` is for) and one number after the equals sign (`b`).

Another interesting thing to note here is that input with `name="a"` gets rendered `equations_number^2` times and equations_number times for `name="b"`. Flask supports this types of forms and to read data from them `request.args.getlist` can be used like so:

``` python
b = request.args.getlist('b', type=int)
```

We could do the same for the `A` array, but we need it to be two-dimensional. This is where NumPy comes in handy with a `reshape` method. All you need to provide for it is an array and a tuple of the shape you want.

``` python
import numpy as np
shape = (len(b), len(b))
A = request.args.getlist(‘a’, type=int) # 1-dimensional
A = np.reshape(A, shape)                # 2-dimensional
```

## Conditional rendering

I would like to hide equations number form after it is submitted. To do that we can use “conditional rendering”. It’s pretty straightforward:

``` html
{% raw %}
{% if equations_number %}
  <form action="/equations">... form for `A` and `b` ...</form>
{% else %}
  <form action="/equations">... form for `equations_number` ...</form>
{% endif %}
{% endraw %}
```

![](https://cdn-images-1.medium.com/max/1600/1*3MdaFhyw5-d7ogP6ia0Fwg.png)

## Finally solving the system

Once we have all the coefficients we can solve the system of equations. I could try to bother with how you can invert matrix `A` and multiply it with a vector `b`, but I won’t. Today we’ll use NumPy once again. This library has a function for exactly what we want — solving a system of linear equations and it is called [`numpy.linalg.solve`](https://docs.scipy.org/doc/numpy-1.15.0/reference/generated/numpy.linalg.solve.html). All it needs is our matrix `A` and `b`, which we already have.

``` python
x = np.linalg.solve(A, b)
```

This is where things get a bit tricky. Since we’re using this one action for everything related to solving the system we have add some ifs. Here’s the final code:

``` python
equations_number = request.args.get('equations_number', type = int)
b = request.args.getlist('b', type=int)
A = np.array(request.args.getlist('a', type=int)).reshape(len(b), len(b))
x = np.array([])
if len(b) > 0:
    x = np.linalg.solve(A, b)
return render_template('equations.html', equations_number=equations_number, A=A, b=b, x=x)
```

Let’s check it with some screens we’re using this action for:

* Start: `equations_number` is `None`, and all other variables are empty arrays

* Number of equations is entered: `equations_number` is present, but all other variables are still empty arrays

* A and b are entered: `equations_number` is `None` again, `b` is an array, `A` is a 2D array and `x` is an answer for the system of equations

With that we can render our final form:

``` html
{% raw %}
{% if x.size > 0 %}
  <p>A = {{ A }}</p>
  <p>B = {{ b }}</p>
  <p>x = {{ x }}</p>
{% elif equations_number %}
  <form action="/equations">
    {% for i in range(equations_number) %}
      {% for j in range(equations_number) %}
        <label for="a">A[{{i+1}}, {{j+1}}]:</label>
        <input type="number" name="a" class="matrix-number matrix-number--a" required>
      {% endfor %}
      <span class="b">
        <label for="b">b[{{i}}]:</label>
        <input type="number" name="b" class="matrix-number" required>
      </span>
      <br>
    {% endfor %}
    <input type="submit" value="Solve">
  </form>
{% else %}
  <form action="/equations">
    <label for="equations_number">Number of equations:</label>
    <input type="number" name="equations_number" required min="1" max="100">
    <br>
    <input type="submit" value="Submit">
  </form>
{% endif %}
{% endraw %}
```

![](https://cdn-images-1.medium.com/max/2688/1*35qefjzGF59MzHkWWcrAPQ.png)

## Areas of improvement (homework?)

* Separate views and actions for different screens of equation

* [A base layout all views](http://flask.pocoo.org/docs/1.0/tutorial/templates/#the-base-layout)

* Form accessibility for bigger inputs and prettier output

## Conclusion

In this story I’ve covered:

* Setting up your Flask application

* Rendering templates

* Making HTML forms

* How to read arguments with Flask (regular, arrays, and even 2-dimensional arrays)

* Conditional rendering

* How a system of linear equations can be represented and solved with NumPy

You can find [the full code](https://github.com/hmlON/KPI/tree/master/Computer%20math%20algorithms) of this project, as well as my other projects, on my [GitHub page](https://github.com/hmlON). If you liked this article you can follow me and if you didn’t — you can leave an angry comment down below.
