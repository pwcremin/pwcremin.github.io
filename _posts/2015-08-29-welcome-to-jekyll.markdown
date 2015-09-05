---
layout: post
title:  "Python CRUD server with Cloudant"
date:   2015-09-5 18:01:11
categories: jekyll update
---

Building a Python CRUD server on IBM Bluemix is straight forward.  After some quick setup, we
will get right into the code.

### Setup

* [Sign up](https://console.ng.bluemix.net/ "Signup for Bluemix").  With Bluemix we can easily setup Python hosting and a Cloudant database.

* [Download](https://github.com/cloudfoundry/cli/releases) the Cloud Foundry command line interface.  This can be run in OSX Terminal or Win Command Prompt (Cygwin not supported)

  * `$ cf api https://api.ng.bluemix.net`

  * `$ cf login`

* Get the starter code for a Python Flask server

  * `$ git clone https://github.com/pwcremin/pythonstarter calltoarms`


Ok, we are almost ready to go, but first we need a Cloudant service from Bluemix

* `$cf marketplace -s cloudantNoSQLDB`

You can see from the output that Bluemix will give you plenty without there being any cost, so lets get this service going:

* `cf create-service cloudantNoSQLDB Shared [your app name]-cloudantnosqldb`
  * replace [your app name] with a unique name


Update your manifest.yml so that the service you just created will be bound to your app:

{% highlight yaml %}
applications:
- services:
  - [your app name]-cloudantNoSQLDB
- disk_quota: 1024M
  host: [your app name]
  name: [your app name]
  path: .
  domain: mybluemix.net
  instances: 1
  memory: 128M
{% endhighlight %}  

Now all we have to do is push our app and we will have a host Python server!

`$ cf push [your app name]`

![My helpful screenshot](/assets/pythoncrud/a.jpg)
