---
layout: post
title:  "Django authentication via Google, Deezer, and Spotify"
date:   2018-11-21 22:25:25 +0200
tags:   django python authentication
---

In this tutorial we will add an ability to sign up, sign in via Google, Deezer, and Spotify, view a success screen, and sign out afterward. Our project is going to be about music releases notifications.

I've already created a Django project with a landing page in [a previous tutorial](https://kholinlabs.com/creating-a-landing-page-in-django).

![red and pink houses with brown wooden doors](https://images.unsplash.com/photo-1493146165056-bb822d9422dd?ixlib=rb-0.3.5&s=dc735fbec7584ac515dfe9047dbdbe56&auto=format&fit=crop&w=1650&q=80)

How will we do that?

We're going to use a [social-app-django library](https://github.com/python-social-auth/social-app-django). It supports [a huge number of auth providers](https://python-social-auth.readthedocs.io/en/latest/intro.html#auth-providers). They are also called "Backends" in the library. As you could guess, we will need Google, Deezer, and Spotify.

To install the library please follow [an official installation guide](https://python-social-auth.readthedocs.io/en/latest/installing.html).

## Adding social-app-django to your Django project

First of all, you need to add the library to our `INSTALLED_APPS` in your project's `settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # ...
    'social_django',
    # ...
    'pages',
]
```

After that, migrate your database by running

```shell
python3 manage.py migrate
```

Then add `social_django.urls` to your project's `urls.py`

```python
urlpatterns = [
    # ...
    path('social/', include('social_django.urls')),
    # ...
]
```

And `SOCIAL_AUTH_URL_NAMESPACE` to your project's `settings.py` again

```python
SOCIAL_AUTH_URL_NAMESPACE = 'social'
```

Now you should have nothing, but we're ready to add an auth provider to the project. For more information, you can read [the official python-social-auth installation guide for Django](https://python-social-auth.readthedocs.io/en/latest/configuration/django.html). 

## Authentication via Google(+)

![](https://images.unsplash.com/photo-1529612700005-e35377bf1415?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=45b2666d6486f1e881e2fe341b0728e8&auto=format&fit=crop&w=1950&q=80)

First, you need to get credentials for the application

1. Visit [Google API Console](https://console.developers.google.com/)
2. Go to credentials
3. Create credentials
4. Add `http://127.0.0.1:8000/social/complete/google-plus/` to **Authorized redirect URIs**
5. Here you'll have to add your production redirect URL once you have one

Now you should see **Client ID** and **Client secret** in your app. Add them to project's `settings.py`

```python
SOCIAL_AUTH_GOOGLE_PLUS_KEY = 'Your Client ID'
SOCIAL_AUTH_GOOGLE_PLUS_SECRET = 'Your Client secret'
```

Add Google to `AUTHENTICATION_BACKENDS`  in project's `settings.py`

```python
AUTHENTICATION_BACKENDS = (
    'social_core.backends.google.GooglePlusAuth',
)
```

And finally, add a link to sign in via Google to your menu or header or wherever you want your users to click to sign in

```html
{% raw %}
<a href="{% url "social:begin" "google-plus" %}">Google</a>
{% endraw %}
```

For more reading about how authentication via Google works here's [the official guide](https://developers.google.com/identity/protocols/OAuth2).

## Authentication via Deezer

![](https://images.unsplash.com/photo-1527150122806-f682d2fd8b09?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=ada3761ade3c12bdfc3af876f4e28667&auto=format&fit=crop&w=2089&q=80)

The process for Deezer authentication is pretty similar to Google's one. First, getting credentials

1. Visit [Deezer for developers](https://developers.deezer.com/myapps)
2. Create an app
3. Set `http://127.0.0.1:8000/social/complete/deezer/` to **Redirect URL after authentication**
4. And again, here you'll have to add your production redirect URL once you have one

Now you should see **Application id** and a **Secret Key** in your app. Add them to project's `settings.py`

```python
SOCIAL_AUTH_DEEZER_KEY = 'Your Application id'
SOCIAL_AUTH_DEEZER_SECRET = 'Your Secret Key'
```

Again, add Deezer to `AUTHENTICATION_BACKENDS` in project's `settings.py`

```python
AUTHENTICATION_BACKENDS = (
    'social_core.backends.google.GooglePlusAuth',
    'social_core.backends.deezer.DeezerOAuth2',
)
```

And again, a link to sign in via Deezer

```html
{% raw %}
<a href="{% url "social:begin" "deezer" %}">Deezer</a>
{% endraw %}
```

#### One more thing

You might want to ask for some permissions. For example, Deezer does not provide user's email by default. You need to ask a permission for `email`. You can check out a full list of available permissions in [official permissions documentation](https://developers.deezer.com/api/permissions).

Adding required permissions is pretty simple with the  `social-app-django` library. You just need to add permission you want to a `scope` constant in the project's `settings.py`. The constant, in Deezer case, has to be called `SOCIAL_AUTH_DEEZER_SCOPE`

```python
SOCIAL_AUTH_DEEZER_SCOPE = ['basic_access', 'email']
```

## Authentication via Spotify

![](https://images.unsplash.com/photo-1532354058425-ba7ccc7e4a24?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=117db81ba4f77871de8d2320d6661f9e&auto=format&fit=crop&w=1650&q=80)

Spotify authentication is as similar to Google as Deezer is.

1. Visit [Spotify for developers](https://developer.spotify.com/dashboard/applications)
2. Create a Client ID
3. Set `http://127.0.0.1:8000/social/complete/spotify/` to **Redirect URIs**
4. And again, here you'll have to add your production redirect URL once you have one

Now you should see **Client ID** and a **Client Secret** in your app. Add them to project's `settings.py`

```python
SOCIAL_AUTH_SPOTIFY_KEY = 'Your Client ID'
SOCIAL_AUTH_SPOTIFY_SECRET = 'Your Client Secret'
```

Again, add Spotify to `AUTHENTICATION_BACKENDS` in project's `settings.py`

```python
AUTHENTICATION_BACKENDS = (
    'social_core.backends.google.GooglePlusAuth',
    'social_core.backends.deezer.DeezerOAuth2',
    'social_core.backends.spotify.SpotifyOAuth2',
)
```

And again, a link to sign in via Spotify

```html
{% raw %}
<a href="{% url "social:begin" "spotify" %}">Spotify</a>
{% endraw %}
```

Regarding permissions, I need `user-read-email` and `user-library-read`. You can check out a full list of available permissions in [official permissions documentation](https://developer.spotify.com/documentation/general/guides/scopes/).

```python
SOCIAL_AUTH_SPOTIFY_SCOPE = ['user-read-email', 'user-library-read']
```

## Life after sign in

Usually, after a sign in it's a good idea to show some sort of dashboard or feed or whatever is the main purpose of your application. I'm going to show a screen with all the latest music releases.

First, let's create a `releases` app

```bash
python3 manage.py startapp releases
```

Add the app to installed apps in the project's `settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'social_django',
    'pages',
    'releases',
]
```

Add URLs to the project's URLs in the project's `urls.py`

```python
urlpatterns = [
	# ...
    path('', include('releases.urls')),
    # ...
]
```

Define a `/releases` URL in `releases/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('releases', views.index, name='index'),
]
```

Create a view in `releases/views.py`

```python
def index(request):
    return render(request, 'releases/index.html')
```

Create a simple template to show that we care about our users in `releases/templates/releases/index.html` (I hate to have to write `releases` two times too)

```html
<h1>Latest Releases</h1>
<p>We care about you, but we don't have anything done yet.</p>
```

Now that we have somewhere to redirect a user after a sign in, there is only one thing left to do — set a `LOGIN_REDIRECT_URL` in the project's `settings.py`

```python
LOGIN_REDIRECT_URL = '/releases'
```

After this line is added all users should go directly to `/releases` URL after they sign in.

## Fixing duplicate users

![](https://images.unsplash.com/photo-1512746755088-5cf7d5565d9e?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=70f0d929ebd7ff35392a2166d97425f4&auto=format&fit=crop&w=2250&q=80)

Now that we have the ability to sign in with a couple different accounts there is one problem — we don't do anything with duplicate users, users that have identical email. We can quickly check that by signing up 2 times via two different services (with Google and Spotify for example).

Then we can check that there are 2 users by running a Django shell

```bash
python3 manage.py shell
```

And checking current count of users there

```python
from django.contrib.auth.models import User
User.objects.count()
```

We can also investigate what is the difference between them

```python
vars(User.objects.all())
```

To fix this problem we'll have to learn a bit more how `social-app-django` works.

When a user signs in they go through a so-called pipeline. Here's how the default pipeline looks like

```python
(
    # Get the information we can about the user and return it in a simple
    # format to create the user instance later. On some cases the details are
    # already part of the auth response from the provider, but sometimes this
    # could hit a provider API.
    'social_core.pipeline.social_auth.social_details',

    # Get the social uid from whichever service we're authing thru. The uid is
    # the unique identifier of the given user in the provider.
    'social_core.pipeline.social_auth.social_uid',

    # Verifies that the current auth process is valid within the current
    # project, this is where emails and domains whitelists are applied (if
    # defined).
    'social_core.pipeline.social_auth.auth_allowed',

    # Checks if the current social-account is already associated in the site.
    'social_core.pipeline.social_auth.social_user',

    # Make up a username for this person, appends a random string at the end if
    # there's any collision.
    'social_core.pipeline.user.get_username',

    # Send a validation email to the user to verify its email address.
    # Disabled by default.
    # 'social_core.pipeline.mail.mail_validation',

    # Associates the current social details with another user account with
    # a similar email address. Disabled by default.
    # 'social_core.pipeline.social_auth.associate_by_email',

    # Create a user account if we haven't found one yet.
    'social_core.pipeline.user.create_user',

    # Create the record that associates the social account with the user.
    'social_core.pipeline.social_auth.associate_user',

    # Populate the extra_data field in the social record with the values
    # specified by settings (and the default ones like access_token, etc).
    'social_core.pipeline.social_auth.load_extra_data',

    # Update the user record with any changed info from the auth service.
    'social_core.pipeline.user.user_details',
)
```

If you like to, you can read a description for each stage of the pipeline. But what interests us at the moment is the 7th step, `social_core.pipeline.social_auth.associate_by_email` which is disabled by default. What it does is it

> Associates the current social details with another user account with a similar email address

Sounds like exactly what we need. Just uncomment the line with this setting and copy the pipeline to your project's `settings.py`

```python
SOCIAL_AUTH_PIPELINE = (
    'social_core.pipeline.social_auth.social_details',
    'social_core.pipeline.social_auth.social_uid',
    'social_core.pipeline.social_auth.auth_allowed',
    'social_core.pipeline.social_auth.social_user',
    'social_core.pipeline.social_auth.associate_by_email',
    'social_core.pipeline.user.create_user',
    'social_core.pipeline.social_auth.associate_user',
    'social_core.pipeline.social_auth.load_extra_data',
    'social_core.pipeline.user.user_details',
)
```

You can read more about in [the official docs for pipelines](https://python-social-auth.readthedocs.io/en/latest/pipeline.html). We'll build a custom pipeline in the next part of the series.

## Signing out

![](https://images.unsplash.com/photo-1520033906782-1684d0e7498e?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=93f0c662e61a6cb38c09aa0d7696d12c&auto=format&fit=crop&w=1934&q=80)

You might want to add an ability for your users to sign out. I would not recommend doing that since this is just another opportunity for a user to leave your site but In case you still want to add it, it's pretty easy. 

Django has an already built-in support for authentication called `django.contrib.auth`. Right now, we'll need just the sign-out part of it.

First, add `django.contrib.auth.urls` to your project's `urls.py`

```python
urlpatterns = [
    # ...
    path('', include('django.contrib.auth.urls')),
    # ...
]
```

Then, set `LOGOUT_REDIRECT_URL` to `'/'` in your project's `settings.py` so that users are redirected to the homepage after they sign out

```python
LOGOUT_REDIRECT_URL = '/'
```

Now, all that is left is to add a sign-out link wherever you want

```html
{% raw %}
<a href="{% url 'logout' %}" class="header-link">Log out</a>
{% endraw %}
```

## Showing only relevant links

But right now your user can see all sign in links as well as a sign-out link. You probably wouldn't want a user to sign in or out 2 times and what does that even mean? So let's show only relevant links.

Django's user model has a special attribute for that called [`is_authenticated`](https://docs.djangoproject.com/en/2.1/ref/contrib/auth/#django.contrib.auth.models.User.is_authenticated) that we can use. We can just use a simple `if` in our template and check if a `user`, a variable which is already available in our templates, `is_authenticated`.

```html
{% raw %}
{% if user.is_authenticated %}
    <a href="/logout">Log out</a>
{% else %}
    <a href="{% url "social:begin" "google-plus" %}">Google</a>
    <a href="{% url "social:begin" "deezer" %}">Deezer</a>
    <a href="{% url "social:begin" "spotify" %}">Spotify</a>
{% endif %}
{% endraw %}
```

## Conclusion

In this story I’ve covered:

- how to use a `social-app-django` library
- how to create apps, set custom scopes in Google, Deezer, and Spotify and of course integrate those services into your application
- how to create a simple success page to show after the sign in
- what is a pipeline in `social-app-django` library
- and how to sign users out

This is the second part of the series of articles about the MuN. Stay tuned for part 3. You can find [the code of this project](https://github.com/hmlON/mun), as well as my other projects, on my [GitHub page](https://github.com/hmlON). Leave your comments down below and follow me if you liked this article.
