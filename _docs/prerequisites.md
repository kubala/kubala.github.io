---
title: System Setup
description: Setting up system for goma server
next-link: /docs/setting-up-rbe
next-name: Setting Up RBE
prev-link: /docs/intro
prev-name: Introduction
---

# Prerequisites

## System Setup

Goma itself is written in Go therefore it can be run on any system with Go runtime installed. The RBE backend we're going to use is written in Python therefore there are no strict platform requirements either. However, to keep this guide simple we'll assume to run everything on machine running Ubuntu 20.04. Virtual machine is ok to run basic tests although make sure you have plenty of disk storage available - 50 GB is absolute minimum. For more complex tests 100GB is a must.

## Accounts

You'll need a gmail account that has an access to the Google Cloud Platform. Please sign up to the GCP if you don't have an account there yet. Don't worry - you won't be charged for any of the Goma related operations. You'll need it only to create a service account that will be used for the authentication purposes.

## Software

Install some basic software packages:

{% highlight shell %}
sudo apt install git golang python python3-venv
{% endhighlight %}

You'll need them later to run Goma and RBE backend.

## Versions

When this document was created both Goma and BuildGrid were under more or less active development. To ensure that everything works as expected this document assumes following component versions to be used:

- Goma: [v0.0.13](https://chromium.googlesource.com/infra/goma/server/+/refs/tags/v0.0.13)
- BuildGrid [0.0.2](https://gitlab.com/BuildGrid/buildgrid/-/tags/0.0.2)

I'll do my best to keep this document up to date with most recent versions of mentioned tools but for now please stick to these particular releases as I cannot guarantee any newer releases will be working with each other.
