---
title: "Dynamic Email Routing With Mailgun and Python"
date: 2019-10-28T12:50:52-03:00
draft: false
authors: ['mauricio']
tags: ['automation', 'mailgun', 'SMTP', 'custom domain']
---
# First things first

This is a three part article. The index below will be updated with the respective links as soon as the upcoming parts are completed an published.

1. [Creating the Base Function and Receiving POSTs](.)
2. Automated Hosting of the Cloud Function
3. Routing Received Emails

# What do we want to achieve?

[destroy.systems](/) is not a huge company. It's not even a company. Actually, it's just [some folks](/authors) who want to write about technical stuff and share interesting experiments. Not being a huge company also means that we don't have lots of resources to spare, we bought a domain and that's already a lot, considering that it won't generate any revenue. Having the domain, we would love to have some custom emails but, for sure, on the cheap side. We would like to have addresses for services as well (github@destroy.systems, circleci@destroy.systems, for example) and have this automatically routed to somewhere else.

Then you'll say: _you can actually do that, there's this fancy tool called [Mailgun](https://www.mailgun.com) that probably achieves what you folks want._

True. We use it, and it's awesome. But stay with me for a second.

Let's say that my email is personalemail@gmail.com, to which we can [add some extra information](https://thenextweb.com/google/2017/08/17/how-the-plus-sign-can-save-your-gmail-inbox-from-becoming-a-pit-of-doom/) in the address, by adding _+something_ after the username. So, if we wanted to track our emails from Github, for example, we can subscribe to github with personalemail+github@gmail.com, and with that we can filter our emails by tracking those that were sent to that address, instead of my default one.

This is such a great feature that Gmail offers. Now imagine if I wanted to do something similar _with my own domain_. Or, even better, create addresses __on the fly__. I'd love to have an address nameofservice@destroy.systems, and then redirect them to personalemail+nameofservice@gmail.com. This is easily achievable by creating new rules on mailgun, the problem is that we would need to create a new rule for every new service. And we couldn't even have a generic case, as the receiver's address cannot be parametrized, everything would have to go to the same address. We need to use something else.

# What are the tools available?

Mailgun has an API. By using this and some scripting we can achieve what we want. Then, if we host this script somewhere, we can have it permanently accessible. Think about _cloud_, _serverless_ and all of these fancy words. We'll talk about that in part 2.

The idea is to use Mailgun to capture all of the emails, forward it to a service that we created and then, within this service, we can modify the email and, with Mailgun API, send it to our personal email account, adding the required tags (or that _+something_) that will allow us to filter what we want:

{{< imgur 9lF7nGN >}}

# Setting up Mailgun

First, we need to configure a route to catch all emails and forward it to an address (use a dummy address for now, we'll have an actual one soon):

{{< imgur 0xVXjlb >}}

Then, set the priority number higher than all the other existing rules (it will be the last rule to be executed): 

{{< imgur q5M1sk5 >}}

# Crack the Code

## Preparing the environment

Okay, we have the rules set. It's coding time.

To keep things pretty, create a virtualenv and add our requirements:

{{< highlight bash >}}
$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip install flask
$ pip freeze > requirements.txt
{{</ highlight >}}

After doing that, we'll have a `requirements.txt` quite similar to the one below:

{{< highlight text >}}
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.10.3
MarkupSafe==1.1.1
Werkzeug==0.16.0
{{</ highlight >}}

## Starting the basic app and Flask wrapper

Having our virtualenv properly set, with the requirements installed and listed, let's create a Flask wrapper for our function, to deal with HTTP requests. Creating this wrapper now is important to simulate how our app will behave when it's hosted in the almighty cloud.

{{< highlight python "linenos=inline" >}}
from flask import Flask
app = Flask(__name__)

@app.route('/', methods=['GET'])
def call():
    return "Hello World!"
{{</ highlight >}}

If in one terminal we run our app...:

{{< highlight bash >}}
$ FLASK_APP=app.py flask run
 * Serving Flask app "app.py"
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
{{</ highlight >}}

... and in another we curl the address informed, we should see a nice Hello World message:

{{< highlight bash >}}

$ curl http://127.0.0.1:5000/
Hello World!

{{</ highlight >}}

## Expand and separate concerns

Cool, we're dealing with HTTP requests. Let's move on to the actual function. If you're curious, take a look at the [Mailgun API](https://documentation.mailgun.com/en/latest/api_reference.html) to understand better what it's capable of. Most of what we'll use, if not all, is in [here](https://documentation.mailgun.com/en/latest/api-sending.html#sending), I'll move straight to it.

Let's create a new file, which I'll call `process.py`, to write our function. It will start simple as this:

{{< highlight python "linenos=inline" >}}
def process():
    return "Hello World!"
{{</ highlight >}}

And now, in our `app.py`, instead of just returning a string, let's import this function and call it:

{{< highlight python "linenos=inline,hl_lines=1 8" >}}
from process import process

from flask import Flask
app = Flask(__name__)

@app.route('/', methods=['GET'])
def call():
    return process()
{{</ highlight >}}

If all went well, we're still reading a `Hello World!` message when we curl it. Cool.

To deal with the actual contents of our emails, we'll need to use an [object from Flask that allows us to deal with requests](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request), called `request`. So, our `app.py` will look like this:

{{< highlight python "linenos=inline,hl_lines=3 8" >}}
from process import process

from flask import Flask, request
app = Flask(__name__)

@app.route('/', methods=['GET'])
def call():
    return process(request)
{{</ highlight >}}

And our `process` function now has to receive it:

{{< highlight python "linenos=inline,hl_lines=1" >}}
def process(request):
    return "Hello World!"
{{</ highlight >}}

## Serious business (?)

From this point on, we'll need to receive data in a similar way to what we should receive from Mailgun. As our app isn't reachable by Mailgun servers yet, we need to simulate this data somehow. I went the extra mile, ran some requests from Mailgun and, at this point, I believe that I can accurately simulate their requests with a `curl` command, which you can see and copy from the gist below. If at any point I notice that it's not accurate, I'll update the gist. So, starting now, we'll no longer just `curl http://127.0.0.1:5000`, we'll run the gist.

{{< gist mauodias a26d7dd0e313bb93cf5fa7d0c66afec9 >}}

To save the gist, you can copy it and paste to a file, or run the commands below:

{{< highlight bash >}}

$ curl -o curl.sh -L https://git.io/JeuUh
$ chmod +x curl.sh

{{</ highlight >}}

## Drop the GETs, get the POSTs

All good, all great. But nothing changed in our browser, we still see the good ol' `Hello World!`. Let's do some stuff with the `request` object. First, to prepare our function to receive data from Mailgun, we need to be able to receive `POST`, and not only `GET`. So, we make a small change in our `app.py`, like below:

{{< highlight python "linenos=inline,hl_lines=1,linenostart=6" >}}
@app.route('/', methods=['GET', 'POST'])
def call():
    return process(request)
{{</ highlight >}}

We're leaving the `GET` only for testing purposes, as Mailgun sends only `POST`.

[Here](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request) we can find the documentation for the `request` object. For this application, all the information we need will be on `request.form`, because Mailgun sends it as form data. To make it easier to handle the information, we can extract just the form data. (Flask relies on the [Werkzeug WSGI web application library](https://werkzeug.palletsprojects.com/en/0.16.x/), so the output of `form` is actually an [`ImmutableMultiDict`](https://werkzeug.palletsprojects.com/en/0.16.x/datastructures/#werkzeug.datastructures.ImmutableMultiDict) object. We can use the `to_dict()` method to convert it to a simple dictionary.) And, after that, we can start gathering the info that we'll need. For now, we won't use more than the actual message, subject and the sender and receiver email.

So, our `process.py` file will look like this, extracting the info from the request and returning as a nicely formatted string:

{{< highlight python "linenos=inline" >}}
def process(request):
    sender = str(data['Sender'])
    receiver = str(data['To'])
    subject = str(data['Subject'])
    message = str(data['body-html'])
    response = 'From: {}\nTo: {}\nSubject: {}\nMessage: {}'.format(sender, receiver, subject, message)
    return response
{{</ highlight >}}

Now, if we run our app and, in another terminal, we run the `curl.sh` script from the gist, this is what we should see:

{{< highlight bash >}}

$ ./curl.sh
From:  bob@domain.com
To:  Alice <alice@domain.com>
Subject:  Re: Sample POST request
Message:  <html>  <head>    <meta content=...

{{</ highlight >}}

# Wrapping up (already?)

It feels like a lot is missing, right? That's because it is.

We've gone a long way already, and by now we're ready to begin receiving requests from Mailgun. Unfortunately, our app isn't publicly available yet, it only runs in our machines. So, in the next part, we'll focus on it, our dynamic router will run in the cloud, and we'll do it in a way that the deploy process is completely automated. If we ever need to change anything in the code of our app, we'll be able to do it with a single commit.
