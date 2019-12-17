---
title: "Deploying scripts in the cloud with Terraform, CircleCI and GCP (but other cloud providers should work just fine)"
date: 2019-12-17T19:19:04-03:00
draft: true
authors: ['mauricio']
tags: ['automation', 'terraform', 'GCP', 'CircleCI']
---

Aaaaalrighty then! Let's move on with our series about creating cool stuff and throwing it in the interwebz for very cheap or, depending on the usage, even free! This is the second of a three part series (which I called three part _article_ earlier, but it doesn't make a lot of sense) containing the following topics:

1. [Creating the Base Function and Receiving POSTs](../dynamic-email-routing-with-mailgun-and-python/)
2. [Automated Hosting of the Cloud Function](.)
3. Routing Received Emails

I will make this part very generic, focusing only in Terraform and GCP, so feel free to completely forget what was done in the first part. We'll put everything together in the last. I'll even use a different script to avoid any confusion. It will be easy to just change it later.

# GCP

This is the part where we can potentially spend money. As I mentioned before, it strongly depends on the usage. If it is low, it may even be free. Otherwise, may cost a few pennies.

## Subscription

## Find and authorize APIs

## Generating the keys

# CircleCI

## Link CI and repos

## Workflow definitions

## Creating environment variables

# Terraform

## Folder organization

## Resources required

## On states

## Local tests (a.k.a. init, plan and validate)
