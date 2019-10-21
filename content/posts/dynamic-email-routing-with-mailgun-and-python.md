---
title: "Dynamic Email Routing With Mailgun and Python"
date: 2019-10-19T16:50:58-03:00
draft: false
authors: ['mauricio']
tags: []
---

# The problem
[destroy.systems](/) is not a huge company. Actually, it's just [some folks](/authors) who want to write about technical stuff. Not being a huge company also means that we don't have lots of resources to spare. We bought a domain and that's already a lot, considering that it won't generate any revenue. But, anyway, we enjoy having some luxury, like our own customized emails.

# What do we want to reach?
Having our domain, the first immediate step was to configure [Mailgun](www.mailgun.com), a service that, among other things, redirects emails based on rules. This means the following, if I send an email to mauricio@destroy.systems it will be redirected to my personal inbox, where I can create rules and save them in specific folders. As far as I know, we can create as much redirection rules as we want. But there are two problems:
- The creation of such rules is manual (there are some APIs that can be used as well, but for this case let's consider that it's manual)
- We can't have dynamic rules

## But what exactly are _dynamic rules_?

I don't even know if this is a thing, but imagine the following: Let's say that my email is personalemail@gmail.com, to which we can [add some extra information](https://thenextweb.com/google/2017/08/17/how-the-plus-sign-can-save-your-gmail-inbox-from-becoming-a-pit-of-doom/) in the address, by adding _+something_ after the username. So, if we wanted to track our emails from Github, for example, we can subscribe to github with personalemail+github@gmail.com, and with that we can filter our emails by tracking those that were sent to that address, instead of my default one.
Now imagine that I want to create custom addresses for any services that I have, related to [destroy.systems](/), such as GitHub, CircleCI, Mailgun and any others. I'd love them to be nameofservice@domain, and then redirect them to personalemail+nameofservice@gmail.com. This is easily achievable by creating new rules on mailgun, but considering that the username part would be exactly the same as the tag in my personal email, couldn't we create an automated way of doing that?

# Available tools

Yes, we can! Among all of the features Mailgun offers, we have some interesting things like redirecting received emails to an API through a POST method, where the email contents are in the request body, and also sending emails not only with SMTP, but through its own API as well. By combining these two features, we can create an additional layer in the redirection process.

# Implementation

PS: This article focuses on creating the service itself, but not hosting it somewhere. This subject will be covered in a future post (but it will involve Terraform, Google Cloud and some Serverless magic).

The idea is to have a flow pretty much like this:

{{< imgur 9lF7nGN >}}

We'll create a single rule on Mailgun, to be processed after all of the existing rules, and set it like this:

{{< imgur 0xVXjlb >}}

Be sure to set a priority higher than the rules that you already have. This has to be the last rule to run, otherwise it will route emails that you wanted to apply a different filter. See below:

{{< imgur q5M1sk5 >}}

Cool, we have our rules properly set. Now, let's move on to the actual processing piece, which will route stuff for us. We'll use Flask to handle the POST request and send it to a function that we'll write. This function will be responsible for processing the request received, extracting the important information and sending to our account. (The separation between the Flask part and the actual function is intentional. It will be more explored in the future post about hosting the function.)



# Final results
