---
layout: post
title:  Explaining how OAuth works with Spotify as an example
date:   2019-01-23 9:05:05 +0200
tags:   oauth python spotify
cover_image: https://images.unsplash.com/photo-1472148083604-64f1084980b9?ixlib=rb-1.2.1&auto=format&fit=crop&w=2850&q=80
---

Have you ever wondered what is OAuth, how it works, and why any more or less popular website implements it? In this article, we'll explore those questions and will write code in Python to use OAuth with Spotify.

# What is it?

According to [Wikipedia](https://en.wikipedia.org/wiki/OAuth), OAuth is

> OAuth is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords.

Sounds pretty logical. Let's use a Spotify to make it more specific.

> OAuth is commonly used as a way for Internet users to grant *websites or applications* (your website or application) *access to their information* (like **their favorite artists**, or **ability to add a new artist to favorites**) on *other websites* (**Spotify**) but without giving them the passwords.

One more thing. "OAuth is an *open standard*" which means that it's going to work about the same for all other websites (like Facebook, Twitter) that are implementing OAuth.

# How it works

On the high level, you (are also called **client**, and the website you are trying to OAuth with is called **provider**), get a user's **access token** (which is essentially a password) and with it, you can use all the accesses that you've listed in a so-called "**scope**".

1. A user clicks a link to the provider website that will request authorization
2. The user sees an authorization screen

![Authorization screen on Spotify](assets/2018-12-24-how-oauth-works-with-spotify-as-an-example/spotify-oauth-screen.png)

3. The user accepts your scope
4. The user is redirected to a "redirect URL" with an access token
5. Now you can make requests for the data you need with the access token

But what do we need to make it work?

# Create a "Client Application"

First of all, you need to create a "Client Application". Its main purpose is to know where to redirect the user on step #4.

Another purpose of the Client Application is to show the name and a brief description of your project on the authorization screen on step #2.

You can google find a place where to do this with a query something like "#{PROVIDER_NAME} oauth app".

# Build a link to the "Provider"

First, you need to find how to build a link. Google can help us again. Just google "#{PROVIDER_NAME} oauth URL". You should find something like official documentation from the provider.

Once we've found how to build the link you would most likely need to use a couple parameters in it:

-  `client_id` — an identifier of your client application so that the provider knows where to redirect the user
- `redirect_uri` — the famous redirect URL (the provider could use the one specified in the Client Application but usually you can enter multiple URLs there and this parameter is for the clarification)
- `scope` — a list of accesses that your project needs (you can find scopes in the official documentation).

Please refer to the official documentation for exact names of the parameters listed above.

I will show what I would have to build in my case. Regarding the scope, I need `user-read-email` and `user-follow-read` to read user's email and what they follow. To start I would point to the URL where the server is running

```python
provider_url = "https://accounts.spotify.com/authorize"

from urllib.parse import urlencode
params = urlencode({
    'client_id': 'MY_CLIENT_ID',
    'scope': ['user-read-email', 'user-follow-read'],
    'redirect_uri': 'http://127.0.0.1:5000/spotify/callback',
    'response_type': 'code'
})

url = provider_url + '?' + params
url
# => 'https://accounts.spotify.com/authorize?client_id=MY_CLIENT_ID&scope=%5B%27user-read-email%27%2C+%27user-follow-read%27%5D&redirect_uri=http%3A%2F%2F127.0.0.1%3A5000%2Fspotify%2Fcallback&response_type=code'
```

# Create an endpoint for the redirect URL

On the step #4 user is redirected to the redirect URL. This URL is where an access token with user's info will come to.

When I've been building my link to the provider, I've used `'http://127.0.0.1:5000/spotify/callback'` as a redirect URL. This link needs to point to an endpoint on whatever server you have.

Here's a [quick](https://kholinlabs.com/what-to-do-when-you-need-a-web-app-quickly) example how to set up an endpoint in Flask

```python
from flask import Flask
app = Flask(__name__)

@app.route("/spotify/callback")
def spotify_callback():
    return "You finally called me back!"
```

In the callback you've just created, you can do whatever you want with the user's data. You can create a user's profile on your project, send him an email or whatever. But if you want to get more data from this user later it's better to save a token that comes with the request to your database.

# Testing whether it's working

![Testing](https://images.unsplash.com/photo-1518349619113-03114f06ac3a?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1000&q=80)

Now that you got your access token, you can do the things that you've requested in the scope.

I've requested [`user-follow-read`](https://developer.spotify.com/documentation/general/guides/scopes/#user-follow-read) so now I can read what artists the user is following to [notify when a new album comes out](http://musicnotifier.com/). Spotify has [an endpoint](https://developer.spotify.com/documentation/web-api/reference/follow/follow-artists-users/) to fetch user's followed artists so let's just use it

```python
artists = []
all_artists_loaded = False
limit = 50
access_token = fetch_access_token() # your function to pull the token
                                    # from the place where you saved it
url = f"https://api.spotify.com/v1/me/following?type=artist&limit={limit}&access_token={access_token}"

while not all_artists_loaded:
    response = requests.get(url).json()['artists']
    current_request_artists = response['items']
    artists += current_request_artists
    if response['next']:
        url = response['next'] + f"&access_token={access_token}"
    else:
        all_artists_loaded = True

print(artists)
```

If you'd like more info on how to fetch latest albums you can read [my previous article](https://kholinlabs.com/getting-your-releases-from-deezer).

# Save the refresh token too

Sadly, access token usually expires. But luckily we can request a new one with the refresh token that comes with the request token.

Once the access token is expired you need to send a request with the refresh token to the endpoint that is specified in the docs. We can do that with [requests](http://docs.python-requests.org/en/master/) library in Python

```python
refresh_token = pull_refresh_token() # your function to pull the token
                                     # from the place where you saved it
client_id = "YOUR_CLIENT_ID" # that you received once you created
                             # the Client Application
client_secret = "YOUR_CLIENT_SECRET" # that you received once you created
                                     # the Client Application too

refresh_url = "https://accounts.spotify.com/api/token" # from the docs

payload = {
    'refresh_token': refresh_token,
    'grant_type': 'refresh_token'
}

auth_header = base64.b64encode(six.text_type(client_id + ':' + client_secret).encode('ascii'))
headers = {'Authorization': 'Basic %s' % auth_header.decode('ascii')}

response = requests.post(refresh_url, data=payload, headers=headers)

token_info = response.json()
```

[Source](https://github.com/plamere/spotipy/blob/4c2c1d763a3653aa225c4af848409ec31286a6bf/spotipy/oauth2.py#L231) of the code above.

The URL (as well as possible other required parameters) have to be, once again, found in the official documentation.

# Why everyone implements OAuth?

It's just convenient for the end-user. OAuth takes the burden of creating passwords or fake accounts from users and places it on developers. But now it shouldn't be too hard for you to use any provider and leverage the full power of users' data.

# Don't invent a bicycle

![Bicycle](https://images.unsplash.com/photo-1518050227004-c4cb7104d79a?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=3900&q=80)

There are always existing solutions to almost any problem you face. Always make sure to google before inventing the bicycle. If you need authentication you, there are already [libraries for that](https://kholinlabs.com/django-authentication-via-google-deezer-and-spotify). If you need to access some API, like a needed to use Spotify's, there are [libraries for that too](https://spotipy.readthedocs.io/en/latest/#).

If you'd like to read about OAuth more in-depth there is [an official guide](https://www.oauth.com/) that covers probably everything written in a very easy to read form.

Also, this is the fourth part of a series of articles about the [MuN](http://musicnotifier.com). Stay tuned for part 5. You can find [the code of this project](https://github.com/hmlON/mun), as well as my other projects, on my [GitHub page](https://github.com/hmlON). Leave your comments down below and follow me if you liked this article.
