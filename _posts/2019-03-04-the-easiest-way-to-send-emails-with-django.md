---
layout: post
title:  The easiest way to send emails with Django (using SES from AWS)
date:   2019-03-04 09:44:44 +0200
tags:   aws, ses, django, python
cover_image: https://images.unsplash.com/photo-1524125833033-bc59e523d3e5?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2270&q=80
---

Do you want to send emails in your application? Notifications, newsletters, promotions, there are so many reasons to send emails.

After reading this article you so be able to send any sorts of emails from your Django application.

There are many ways you can send emails using different email sending services. But in this guide, I'll cover one of the most popular services, SES, or Simple Email Service. It has been developed by Amazon and can be used for free for up to 62000 emails per month.

To work with SES we’ll use [`django-ses`](https://github.com/django-ses/django-ses) library. It is very easy to set up and use. 

But before we can start we need to...

# Sign up on AWS

If you don't already have an AWS account you need to get one. You simply won't be able to use SES without it.

To sign up you need to go to [the AWS sign up page](https://portal.aws.amazon.com/billing/signup#/start). 

# Setting up

I’m assuming you already have a Django project. To install `django-ses` library you need to run the following command

```bash
pip install django-ses
```

Once the library is installed you need to tell Django to use this library as email backend by adding the following line to your `settings.py`

```python
EMAIL_BACKEND = 'django_ses.SESBackend'
```

Then you need to get credentials for SES

1. Go to AWS
2. Choose IAM service
3. Add user
4. Choose a name for the user and “Programmatic access” type (to get access key ID and secret access key)
5. Attach `AmazonSESFullAccess` permission
6. Create user

After that, you should see “Access key ID” and “Secret access key”. Add them to `settings.py` like that

```python
AWS_ACCESS_KEY_ID = 'YOUR-ACCESS-KEY-ID'
AWS_SECRET_ACCESS_KEY = 'YOUR-SECRET-ACCESS-KEY'
```

# Sending an email

![](https://images.unsplash.com/photo-1479920252409-6e3d8e8d4866?ixlib=rb-1.2.1&auto=format&fit=crop&w=2250&q=80)

Now that we have all the credentials in place, we can send an email. We can do this using Django core functionality ([`django.core.mail.send_mail`](https://docs.djangoproject.com/en/2.1/topics/email/) to be more specific).

```python
from django.core.mail import send_mail
send_mail(
    'Subject here',
    'Here is the message.',
    'from@example.com',
    ['to@example.com']
)
```

And this email should be sent using SES.

So let’s jump into Django’s shell and try it out.

```python
python manage.py shell
>>> from django.core.mail import send_mail
>>> send_mail(
...    'Subject here',
...    'Here is the message.',
...    'from@example.com',
...    ['to@example.com']
...)
boto.ses.exceptions.SESAddressNotVerifiedError: SESAddressNotVerifiedError: 400 Email address is not verified.
<ErrorResponse xmlns="http://ses.amazonaws.com/doc/2010-12-01/">
  <Error>
    <Type>Sender</Type>
    <Code>MessageRejected</Code>
    <Message>Email address is not verified. The following identities failed the check in region US-EAST-1: from@example.com</Message>
  </Error>
  <RequestId>3966220a-3c6c-11e9-909d-889357e40867</RequestId>
</ErrorResponse>
```

Oh no! We got an error… Who would’ve thought that we couldn’t send emails from `from@example.com`. Let’s fix this.

# Sending emails from your personal email address

![](https://images.unsplash.com/photo-1529454488831-3c13b1640074?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2250&q=80)

To begin with, we can send an email from your personal email address. This should be pretty acceptable if you’re building an MVP.

If you don’t want for everyone to see your personal email, you can create a separate email address for your application like `your.application@gmail.com`. If people will start blocking your email for some reason, it will not affect your personal email health.

To send emails from your email address you need to

1. Go to AWS
2. Choose SES service
3. Go to “Email Addresses" tab
4. Click "Verify a New Email Address"
5. Enter your email address of choice
6. Go to the inbox for the email you’ve specified
7. Confirm that you want to use your email in SES

Once you’re done, you should be able to send an email from the Django shell

```python
>>>send_mail(
...    'Subject here',
...    'Here is the message.',
...    'your.application@gmail.com',
...    ['your.email.address+to.test@example.com']
...)
1
```

After that, you should receive an email

![img](/assets/2019-03-04-the-easiest-way-to-send-emails-with-django/email_from_your_personal_email_address.png)

You might’ve noticed a number that has been returned by the `send_mail` method. This is the number of successfully sent emails, which in our case was 1.

If we try to send an email to some address and it will fail then the failed email will not get counted

```python
>>> send_mail(
...     'Subject here',
...     'Here is the message.',
...     'your.application@gmail.com',
...     ['your.email.address+to.test@example.com', 'not.existing.email.for.real.asdvsrg@gmail.com']
... )
1
```

This `send_mail` call returned 1 because it could not send an email to the second address (`not.existing.email.for.real.asdvsrg@gmail.com`) because it does not exist. I just made it up. 

# Sending emails from a custom domain

![](https://images.unsplash.com/photo-1543448360-6b80281205f2?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2250&q=80)

Once your MVP is working and you have a domain for your application it's time to send emails from this domain. To do this you need to add this domain to SES

1. Go to AWS
2. Choose SES service
3. Go to the "Domains" tab
4. Click “Verify a new domain"

If you already have your domain on AWS Route53 then you just click through and your domain will be ready.

If you have your domain set up somewhere else you need to follow specified instructions.

Once you do follow the instructions correctly, you should see green verified status and the domain should be ready for sending emails via SES. We can go back to our Django shell and test sending emails from the new and shiny domain

```python
>>>send_mail(
...    'Subject here',
...    'Here is the message.',
...    'hello@yourdomain.com',
...    ['your.email.address+to.test@example.com']
...)
1
```

Here’s your email

![img](/assets/2019-03-04-the-easiest-way-to-send-emails-with-django/email_from_your_domain.png)

Pretty neat, right? But it could be even more pretty…

#  Custom name in the email 

Have you ever noticed a pretty email name? Something like this

![img](/assets/2019-03-04-the-easiest-way-to-send-emails-with-django/email_with_custom_name_example.png)

It even includes a link to unsubscribe. You can be so cool too. This is very easy. You just need to modify the sender name. Add a desired name in the beginning in double quotes and an email address the email will be sent from in the end in `<>` like this `”Desired name” <email@example.com>`.

```python
>>>send_mail(
...    'Subject here',
...    'Here is the message.',
...    '"Hello, you" <hello@yourdomain.com>',
...    ['your.email.address+to.test@example.com']
...)
1
```

Now you have a cool name too

![img](/assets/2019-03-04-the-easiest-way-to-send-emails-with-django/email_with_custom_name.png)

And it can be whatever you want it to.

# Conclusion

Now you should know how

- to set up `django-ses` library
- use it to send emails
- set up emails in SES
- set up domains in SES
- make email sender info prettier

This is the sixth part of a series of articles about the [MuN](http://musicnotifier.com/). Stay tuned for part 7. You can find [the code of this project](https://github.com/hmlON/mun), as well as my other projects, on my [GitHub page](https://github.com/hmlON). Leave your comments down below and follow me if you liked this article.