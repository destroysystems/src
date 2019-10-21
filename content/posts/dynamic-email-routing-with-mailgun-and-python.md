---
title: "Dynamic Email Routing With Mailgun and Python"
date: 2019-10-19T16:50:58-03:00
draft: true
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

It can be pretty much like this:

{{  }}

# Final results
