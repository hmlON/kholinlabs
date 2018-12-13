---
layout: post
title:  Getting your latest releases from Deezer with Python
date:   2018-12-14 11:54:54 +0200
tags:   django python deezer
cover_image: https://images.unsplash.com/photo-1527150122806-f682d2fd8b09?ixlib=rb-0.3.5&ixid=eyJhcHBfaWQiOjEyMDd9&s=ada3761ade3c12bdfc3af876f4e28667&auto=format&fit=crop&w=2089&q=80
---

Have you ever felt like you've missed an album from your favorite artist? If yes then this article is just for you.

Here we will fetch your followed artists and their albums from [Deezer API](https://developers.deezer.com/api) to see whether you missed any releases or not. We will use Python to do all the work.

# Getting started

First, you'll need to know your ID. You can get it by opening Deezer in a browser and visiting your profile. 

Now, if you check the URL address you should see something like this 

```
https://www.deezer.com/en/profile/700513741
```

A number at the end of the URL is your ID (mine is `700513741`). Let's store it in a variable.

```python
deezer_id = 700513741
```

If you want to get IDs of users of your project you'll need to create a Deezer application and connect that application to your project. For more information on how to do that you can check out [this guide](https://kholinlabs.com/django-authentication-via-google-deezer-and-spotify).

# Fetching information about you

![Archive storage](https://images.unsplash.com/photo-1470173274384-c4e8e2f9ea4c?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1650&q=80)

To begin our fetching journey let's fetch ourselves first. To fetch a user we can use Deezer's [User endpoint](https://developers.deezer.com/api/user). We just need to send a request to a URL specified in the documentation and information about a user should be in the response.

But before doing any requests with Python we can try doing some requests in Deezer's [API explorer](https://developers.deezer.com/api/explorer).

The docs say that the URL for getting info about a user is `https://api.deezer.com/user/me`.  but if we try to perform a request to the endpoint we receive an error

> An active access token must be used to query information about the current user

This means that we have to get a token that is related to our user for Deezer to know who is "me". But we all like doing things in an easy way and getting that access token is just too hard. This is exactly why I asked you to get your ID by yourself. Now we can just replace `/me` with your ID and everything should be working. Here's what I got in the API explorer

![Successfull response with a user in the API explorer](assets/2018-12-10-getting-your-releases-from-deezer/api-explorer-1.png)

# Making real requests

Making requests in Python is very easy with [`requests` library](http://docs.python-requests.org/en/master/). If you don't have the library installed you can use an [official installation guide](http://docs.python-requests.org/en/master/user/install/#install).  Once you have `requests` installed we can proceed to coding.

To send a regular [`GET`](https://www.w3schools.com/tags/ref_httpmethods.asp)  request with `requests` the library provides us a `get` method which just needs a URL

```python
import requests
url = f"https://api.deezer.com/user/{deezer_id}"
user_response = requests.get(url)
```

The `user_response` variable should now contain lots of data that happened during the request. What interests us is what the body of the response. We can get it with `response.content` method

```python
user_response.content
# => b'{"id":700513741, "name":"hmlON", ...}'
```

As we can the response contains a JSON string. We could parse it with a [`json` library](https://docs.python.org/2/library/json.html) but `requests` have worked everything out for is — we can just use a `json()` method

```python
user_response.json()
# => {'id': 700513741, 'name': 'hmlON', ...}
```

Now we have a normal dictionary with which we can work. Let's finish our "hello world" program (Deezer API edition) before we continue making other requests

```python
name = user_response.json()['name']
print(f"Hello, {name}!")
```

# Fetching your favorite artists

![Your favorite artist's concert](https://images.unsplash.com/photo-1533174072545-7a4b6ad7a6c3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1650&q=80)

Your favorite artists can be found on a [User / Artists endpoint](https://developers.deezer.com/api/user/artists). As usual, it suggests to use `/me` in the URL but we're smarter then that and will use the ID. So let's test the endpoint.

```python
artists_url = f"https://api.deezer.com/user/{deezer_id}/artists"
artists_response = requests.get(artists_url)
artists_response.json()
# => {
#  "data": [
#    {
#      "id": "296861",
#      "name": "The Neighbourhood",
#	   ...
#    },
#    ...
#  ]
#  ...
#  "total": 30,
#  "next": "https://api.deezer.com/user/700513741/artists?index=25"
# }
```

The endpoint seems to be working. As we can see from the response, we have a couple of interesting fields here:

- data — your favorite artists that we wanted
- total — how many artists we are following
- next — a link to the next page of results

The `next` parameter is empty if a response returned the last page of data. This means that if we want to just take a glance at artists we can send requests as is. But if we want to fetch all data (like we do now) we would have to use this parameter.

The algorithm for fetching all records is pretty simple — we go to the next page while it is present. Let's go ahead and implement it

```python
artists = [] 			   # we'll store artists here as we go through the pages
all_artists_loaded = False # in the beginning nothing is loaded
url = f"https://api.deezer.com/user/{deezer_id}/artists"

while not all_artists_loaded:
    response = requests.get(url).json()
    artists += response['data']   # data attribute contains artists from the request
    if response.get('next'):      # there are more artists on the next page
        url = response['next']    # we will continue to fetch from the next page
    else:                         # there are no more followed artists
        all_artists_loaded = True # we will end the loop
```

The algorithm is pretty straightforward but there is one thing I would like to note. When I'm checking whether there is a next page I use `response.get('next')` instead of `response['next']`. The difference here is that when the next key is not present in the response the `response.get('next')` method will return nothing while the regular `response['next']` will raise an error.

```python
response = {'data': ['...']} # response without the 'next' attribute

response['next']
# raises KeyError: 'next'

response.get('next')
# => None
```

# Fetching your favorite artists' releases

![Vinil albums store](https://images.unsplash.com/photo-1530288782965-fbad40327074?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2250&q=80)

Now that we have all followed artists we can fetch their albums. The [Artist / Albums endpoint](https://developers.deezer.com/api/artist/albums) is quite suitable for this task.

This time we would need to loop over each artist and save all albums in an array. To fetch all releases for each artist we would use the same algorithm as we've used to fetch artists.

```python
releases = [] # we'll store all releases here

for artist in artists:
    # the old algorithm is still working here
    all_releases_loaded = False
    artist_id = artist['id']
    url = f"https://api.deezer.com/artist/{artist_id}/albums"

    while not all_releases_loaded:
        response = requests.get(url).json()
        releases += response['data']
        if response.get('next'):
            url = response['next']
        else:
            all_releases_loaded = True
```

Pretty much the same as before. But there is one problem which we will notice if we look at any release

```python
releases[-1]
# {
#  'cover': 'https://api.deezer.com/album/40571541/image',
#  'explicit_lyrics': True,
#  'fans': 52,
#  'genre_id': 116,
#  'id': 40571541,
#  'link': 'https://www.deezer.com/album/40571541',
#  'record_type': 'single',
#  'release_date': '2017-04-18',
#  'title': 'DEADMAN WUNDERLAND',
#  'tracklist': 'https://api.deezer.com/album/40571541/tracks',
#  'type': 'album'
# }
```

The problem is that we don't know who the artist is. We can solve it in two ways:

1. by storing artists in a dictionary with their ID as a key and adding `artist_id` to all releases
2. by storing full title (with artist's name) in the release

I will go with the second one since it would be easier to do but if you would like to implement the first solution or any other please share it in the comments.

To save the full title in a release we would need to store artist's songs in a temporary array. And when all albums are fetched we would need to add the full title to them.

```python
releases = [] # we'll store all releases here

for artist in artists:
    # this time we'll store releases in a temporary array
    current_artist_releases = []

    # nothing changed here
    all_releases_loaded = False
    artist_id = artist['id']
    url = f"https://api.deezer.com/artist/{artist_id}/albums"

    while not all_releases_loaded:
        response = requests.get(url).json()

        # now we're saving each page's response to current_artists_releases
        # instead of straight to releases
        current_artist_releases += response['data']
        
        # this stays the same too
        if response.get('next'):
            url = response['next']
        else:
            all_releases_loaded = True
            
    # and after we've fetched all releases
    # we can add full title to all of them
    for release in current_artist_releases:
        full_title = f"{artist['name']} — {release['title']}"
        release['full_title'] = full_title
        
    # and only after we every release from current artist
    # has a full title we can add them to the main releases
    releases += current_artist_releases
```

So now all releases should have a full title

```python
releases[-1]['full_title']
# => 'Aries — DEADMAN WUNDERLAND'
```

# So what have you missed?

![](https://images.unsplash.com/photo-1460687521562-9eead9abe9e8?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2250&q=80)

Let's check what was released during the last year. If you haven't checked it, you might be surprised by some interesting albums.

To begin, we need to filter releases so that only that were created during the last year are left. To do this we will use a standard [datetime library](https://docs.python.org/2/library/datetime.html) to get a date of the last year

```python
import datetime

today = datetime.date.today()
print(today)
# => 2018-12-13

year_ago = today - datetime.timedelta(days=365)
print(year_ago)
# => 2017-12-13
```

Then, to filter we need to compare the release date with our `year_ago` variable. The only problem here is that release date is a string

```python
releases[-1]['release_date']
# => '2017-04-18'
```

And we can't compare string with a date in Python. So a way out of this is to convert a string into a date. The datetime library has a function just for our case called [`datetime.strptime`](https://docs.python.org/3/library/datetime.html#datetime.datetime.strptime). It needs a string (the release date) and it's format.

As we can see, `release_date` format is `'year-month-day'`. With a little help of [documentation](https://docs.python.org/3/library/datetime.html#strftime-strptime-behavior), this converts into `'%Y-%m-%d'`

```python
datetime.datetime.strptime(releases[-1]['release_date'], '%Y-%m-%d').date()
# => datetime.date(2017, 4, 18)
```

Let's store the `release_date` as a date instead of a string for all `releases`.

```python
for release in releases:
    previous_date = release['release_date']
    new_date = datetime.datetime.strptime(previous_date, '%Y-%m-%d').date()
    release['release_date'] = new_date
```

Now we can finally filter our `releases` to get only the ones from last year

```python
latest_releases = [
    release for release in releases
	if release['release_date'] > year_ago
]
```

After that, all of the last year's releases are stored in the `latest_releases` variable. Now we need to sort those releases by date to have them listed chronologically

```python
latest_releases = sorted(latest_releases, key=lambda release: release['release_date'])
```

And finally, we can print those releases

```python
for release in latest_releases:
    date = release['release_date']
    title = release['full_title']
    print(f"{date}: {title}")
# ...
# 2018-10-25: Aries — BLOSSOM
# 2018-11-02: Panic! At the Disco — The Greatest Show
# 2018-11-02: The Neighbourhood — Hard To Imagine The Neighbourhood Ever Changing
# 2018-11-09: Muse — Simulation Theory (Super Deluxe)
# 2018-11-09: Imagine Dragons — Origins
# 2018-11-09: Imagine Dragons — Origins (Deluxe)
# 2018-11-09: Goody Grace — Nostalgia Is A Lie
# 2018-11-30: Arctic Monkeys — Tranquility Base Hotel & Casino
```

If you've followed along this far please share your latest releases for the last month in the comments.

# Conclusion

In this article I've covered:

- how to work with Deezer API
- how to make requests with Python
- how to convert strings into dates

Congratulations, now you know how to check for new releases from artists that you follow. If you're like me and hate missing out on new releases from your favorite artists, I've built [a website just for this purpose](https://musicnotifier.com/).

If you would like to run the code by yourself and don't want to scramble it from the article pieces by pieces, I've uploaded it into [a gist on GitHub](https://gist.github.com/hmlON/5cbe4b9723e430866c9653aff5fc6eee).

Also, this is the third part of a series of articles about the MuN. Stay tuned for part 4. You can find [the code of this project](https://github.com/hmlON/mun), as well as my other projects, on my [GitHub page](https://github.com/hmlON). Leave your comments down below and follow me if you liked this article.