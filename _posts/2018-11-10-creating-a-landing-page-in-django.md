---
layout: post
title:  "Creating a landing page in Django"
date:   2018-11-12 22:57:57 +0200
tags:   django python landing
---

In this tutorial we will create a landing page for a music releases notifications application in [Django](https://www.djangoproject.com/). To begin, we need to create a Django project.

![Landing](https://images.unsplash.com/photo-1516849947868-a9dfa8bdbf91?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=c34fe623c2853b7de245758d5eda899b&auto=format&fit=crop&w=3000&q=80)

## Creating a Django project

I will assume you have Django installed but in case you don't, you can use the [official installation guide](https://docs.djangoproject.com/en/2.1/topics/install/).

To create a project run the following command in your terminal

```shell
django-admin startproject mun
```

And after the project is created go to its directory via `cd` command

```shell
cd /mun
```

And starting a server is as simple as

```shell
python manage.py runserver
```

After visiting `127.0.0.1:8000` you should see a screen that everything is great...

![Django default landing page](assets/2018-11-10-creating-a-landing-page-in-django/django_new_app.png)

Except it's not your landing page...

## Where do we start?

To add a landing page we need to create an "app" which is a Django concept for things that are not coupled to your app and can be extracted from it (for example authentication which can be used in other applications too).

But let's not get into that too much and just create a simple app called "pages" which will only contain static pages like "landing", "contact us" and "about".

```shell
python manage.py startapp pages
```

When the app is ready, we need to add it to Django's `INSTALLED_APPS`. We'll Just append our app to the end of those other ones because we're going to need some of those.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'pages',
]
```

The newly created `pages` app has a new `pages`  folder with a couple of python files. Particular ones that interest us are `pages/urls.py` and  `pages/views.py`.

## URLs

When a request comes to the server, Django has to find a "view", a function that will process the request and send the response. The way Django finds how to connect a URL that came with the request and a view is via `urlpatterns` which are located in your project's main directory. In my case, it's  `mun/urls.py`.

In order to show our landing page on `127.0.0.1:8000` and essentially on our main domain we need to add the following code to `pages/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
	path('', views.index, name='index'),
]
```

A path is what comes after a domain in a URL. For example, in the URL `https://www.google.com/search?q=mun` the path would be `/search?q=mun`. Since we want our landing page to be on the `/` path the first parameter we provide to the `path` method is `''`. The second argument is what view we need to render which is an `index` view in our case.

We also need to add `urlpatterns` from our app to the main project's `urlpatterns` with a built-in function `include`

``` python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('', include('pages.urls')),
    path('admin/', admin.site.urls),
]
```

## The view

If you recall, the `index` view is not yet created so let's go ahead and create it in `pages/views.py`

```python
from django.shortcuts import render

def index(request):
	return render(request, 'pages/index.html')
```

Our `index` action simply renders a  `'pages/index.html'` template which is again, not yet created. So let's create it.

## The template

I want the landing page to contain the project's name, a brief description and a list of features.

First, let's start with the header. Our sign in buttons won't do anything right now so their `href` is `#` (which means it won't do anything) for now. We'll fix that in the next story.

```html
<header class="header">
  <div class="header-text">Sign in via:</div>
  <a href="#" class="header-link">Spotify</a>
  <a href="#" class="header-link">Deezer</a>
</header>
```

Second, let's add the brand name and a description

```html
<section class="landing-section landing-main">
  <h1 class="brand">MuN</h1>
  <h2>
    <span class="brand-text">Mu</span>sic
    <span class="brand-text">N</span>otifications
  </h2>
  <p class="landing-main-text">
    Got tired of missing out on releases from your favourite artists?
    <br>
    You have found a perfect solution...
  </p>
</section>
```

Third, a list of features:

```html
<section class="landing-section langing-features">
  <section class="landing-feature feature-music-services">
    <h2>Connect your favourite <span class="brand-text">mu</span>sic services</h2>
    <ul>
      <li>Spotify</li>
      <li>Deezer</li>
    </ul>
  </section>
  <section class="landing-feature feature-notifications">
    <h2>Get <span class="brand-text">n</span>otifications wherever you want them</h2>
    <ul>
      <li>Email</li>
      <li>Telegram</li>
    </ul>
  </section>
</section>
```

However, if we check our browser now we'll see a not so pleasant picture:

![A landing page without css](assets/2018-11-10-creating-a-landing-page-in-django/landing-with-no-html.png)

## A CSS is a static file

To add CSS to the template Django has a whole app called [`django.contrib.staticfiles`](https://docs.djangoproject.com/en/2.1/ref/contrib/staticfiles/). By default , Django looks for static files in `static` folders of all apps. To add styles to the template we need to tell Django that we're going to use static files and add a stylesheet tag like so:

```html
{% raw %}
{% load static %}
<link rel="stylesheet" type="text/css" href="{% static 'pages/style.css' %}">
{% endraw %}
```

We'll also need the following line so that our page looks good on mobile

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

And while we're at it, let's add a title for our page

```html
<title>MuN: Music Notifications</title>
```

Here's the full code glued together:

```html
{% raw %}
{% load static %}
<title>MuN: Music Notifications</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" type="text/css" href="{% static 'pages/style.css' %}">

<header class="header">
  <div class="header-text">Sign in via:</div>
  <a href="#" class="header-link">Spotify</a>
  <a href="#" class="header-link">Deezer</a>
</header>
<section class="landing-section landing-main">
  <div class="landing-section-content">
    <h1 class="brand">MuN</h1>
    <h2>
      <span class="brand-text">Mu</span>sic
      <span class="brand-text">N</span>otifications
    </h2>
    <p class="landing-main-text">
      Got tired of missing out on releases from your favourite artists?
      <br>
      You have found a perfect solution...
    </p>
  </div>
</section>
<section class="landing-section langing-features">
  <section class="landing-feature feature-music-services">
    <h2>Connect your favourite <span class="brand-text">mu</span>sic services</h2>
    <ul>
      <li>Spotify</li>
      <li>Deezer</li>
    </ul>
  </section>
  <section class="landing-feature feature-notifications">
    <h2>Get <span class="brand-text">n</span>otifications wherever you want them</h2>
    <ul>
      <li>Email</li>
      <li>Telegram</li>
    </ul>
  </section>
</section>
{% endraw %}
```

## Adding some style

![Some style](https://images.unsplash.com/photo-1523268755815-fe7c372a0349?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=78f4a9da162471fff0acce3b5be33393&auto=format&fit=crop&w=2250&q=80)

To begin with, I would like to have a full-screen picture in the background with the project's name and the description in the middle.

We can do the text in the middle with the flexbox. First, we need to make our `landing-section` container width and height to full screen. CSS has a special unit for that: viewport height (or `vh`) and viewport width (or `vw`). If we set them both to `100` it would take the whole screen (or viewport).

However, if we have too much content and the screen is too small (a mobile phone for example) then we'd need to show more content than `100vh`. We can use `min-height` instead of `height` for that.

```css
.landing-section {
  min-height: 100vh;
  width: 100vw;
}
```

Now that we have our container take up the full screen we can position the content in the center with the flexbox. First of all, we need to set `display: flex` to use flexbox. Then we can use `align-items: center` and `justify-content: center` to position the content in the center.

![Display flex without flex-direction](assets/2018-11-10-creating-a-landing-page-in-django/css-1.png)

There is only one problem left which is the direction of the content. The fix for it is pretty strait-forward — adding `flex-direction: column`.

![Landing page with flexbox](assets/2018-11-10-creating-a-landing-page-in-django/css-2.png)

For more flexbox, I recommend reading [this guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/).

Here's how our final `landing-section` class looks like

```css
.landing-section {
  min-height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
} 
```

## The image in the background

I've used [Unsplash](https://unsplash.com/) to find the image that would fit my preference.

![The image I found](https://images.unsplash.com/photo-1460667262436-cf19894f4774?ixlib=rb-0.3.5&s=54bdd105057cf056b35be4150ce317eb&auto=format&fit=crop&w=2250&q=80)

I set the image as a background with a little help of [CSS-Tricks](https://css-tricks.com/perfect-full-page-background-image/).

```css
.landing-main { 
  background: url(https://images.unsplash.com/photo-1460667262436-cf19894f4774) no-repeat center center fixed; 
  background-size: cover;
}
```

However, the image has light and dark colors and I couldn't use neither black nor white colors over it. So I decided to add a little overlay and make the text white (or rather light-grey because white is too strong).

```css
.landing-section-content {
  padding: 2rem;
  background: rgba(0,0,0,0.5);
  width: 100vw;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  color: #ccc;
}

.brand {
  margin: 0;
  color: #c55e00;
}

.brand-text {
  color: #c55e00;
}
```

And a bit of styling for the header...

```css
.header {
  width: 100vw;
  display: flex;
  justify-content: flex-end;
  top: 0;
  left: 0;
  position: absolute;

  background: rgba(0,0,0,0.5);
}

.header-link {
  margin: 1rem;
  border-bottom: 3px solid #c55e00;
  font-size: 1.2rem;
  color: #222;
  text-decoration: none;
  color: #ccc;
}

.header-text {
  margin: 1rem;
  font-size: 1.2rem;
  color: #ccc;
}
```

And here's what I got:

![Final version of the landing page](assets/2018-11-10-creating-a-landing-page-in-django/css-3.png)

If you have any ideas for the features part, please, write them in the comments because it currently looks pretty awful. 

## One more thing

We've got most things done by now, however, there is one more. It's the icon. I like [LogoMark](https://logomakr.com) website for this purpose. It is very simple to use and has a lot of beautiful icons and stuff.

![Making an icon in the LogoMark](assets/2018-11-10-creating-a-landing-page-in-django/logo.png)

When you download the icon, it gets saved in `.png` extension but `.ico` is better for an icon. [ICOConverter](http://icoconvert.com/) is a great website for it.

After you download the icon in `.icon` extension, just copy it to the `pages/static` folder of your project and add 1 line to the `index.html` template:

```html
{% raw %}
<link rel="shortcut icon" href="{% static 'favicon.ico' %}"/>
{% endraw %}
```

## Conclusion

In this story I’ve covered:

- how to create a Django project
- Django's apps concept
- how Django uses `urlpatterns` to find your view
- how to write HTML for your landing page
- how to style your landing page so that it looks good
- and the cherry on top of the cake — your icon

This is the first part of the series of articles about the MuN. Stay tuned for part 2. You can find [the code of this project](https://github.com/hmlON/mun), as well as my other projects, on my [GitHub page](https://github.com/hmlON). If you liked this article you can follow me and if you didn’t — you can leave an angry comment down below.
