---
layout: post
comments: true
title:  "Ionic App with Watson Visual Recognition"
date:   2015-09-5 18:01:11
categories: bluemix ionic
---

The following are setup instructions for [Raymond Camden's Blog](http://www.raymondcamden.com/2015/08/05/a-real-world-app-with-ibm-bluemix-node-cordova-and-ionichttp://www.raymondcamden.com/2015/08/05/a-real-world-app-with-ibm-bluemix-node-cordova-and-ionic)
about creating a Bluemix enabled Ionic mobile app.

### Setup 

  Follow the below instructions to get your environment setup for working with Bluemix and Ionic.  

* You will need the mobile SDKs for Android or iOS.  If you do not have one installed then grab the
  Android SDK and get its install going right away (this will take... a while)
  
  Install the [Android Studio](https://developer.android.com/sdk/index.html)
  
  #or
  
  Get the [Android SDK](https://developer.android.com/sdk/index.html#Other) zip 
  
  [Android SDK install instructions](https://developer.android.com/sdk/installing/adding-packages.html)
    
* [Sign up](https://ibm.biz/BluemixATX "Signup for Bluemix") with Bluemix so that we can use the Visual Recognition service.
    
* [Download](https://github.com/cloudfoundry/cli/releases) the Cloud Foundry command line interface.  
  This can be run in OSX Terminal or Win Command Prompt (Cygwin not supported).  You will need to restart
  your terminal or cmd window after the install (so that it gets the updated environment).
    
  `cf api https://api.ng.bluemix.net`
    
  `cf login`

* [Node](https://nodejs.org/en/) will need to be installed so that you can use npm

* Install Cordova and Ionic

  `sudo npm install -g cordova ionic`
    
* Get the code for the workshop.  If you already have git installed this is easy:
 
  `git clone https://github.com/cfjedimaster/IonicBluemixDemo`
  
  If you don't then simply [download the zip from here](https://github.com/cfjedimaster/IonicBluemixDemo/archive/master.zip)
  
  After cloning or unzipping you should now have a directory named IonicBluemixDemo with the code
  
* Install the node packages for the demo
  
  `cd IonicBluemixDemo/server`
  
  `npm install`

  `cd ..` just to get back to the root dir
      
* Now, this is jumping ahead a bit, but lets create our Ionic mobile project.  Make sure you are in 
  the `/IonicBluemixDemo` root

  `ionic start mymobileapp ./mobile/www`

  `cd mymobileapp`
  
* Get the cordova plugings that you will need for the workshop

  `cordova plugin add cordova-plugin-camera`
  
  `cordova plugin add cordova-plugin-file-transfer`

### Create your Bluemix app and bind the Visual Recognition service

Log into Bluemix, go to your Dashboard, and then select the Create App option.

![](/assets/ionic/createapp.png)

We are creating a mobile app (duh), so select Mobile

![](/assets/ionic/mobileapp.png)

Now, it is important that your app name is unique (meaning no other app in Bluemix can have that name).  This tends to be a confusing step for some 
reason, but we can do it! Your name plus about 4 random numbers should do it.  For example FightingIrish-8374 (don't use that... its an example).

![](/assets/ionic/appname.png)

It will take a few minutes for your app to be created.  Use this time to say hello to your neighbor.

At this point your app is created.  You will be asked to setup authentication, but that is outside 
the scope of this workshop so don't worry about it.

Select the arrow in the top left corner, and lets check out your app
![](/assets/ionic/arrow.png)

Ok, cool so we have an app in Bluemix.  What does this even mean? 

___When you create a mobile application, multiple services are provisioned under a single application context. A server-side application is also provisioned so that you can provide server-side functions, such as resource URIs and static files.___

For instance, the app that you just created has a Node runtime.  It is running some boilerplate code for now,
but you can see that your server is running by clicking on the route that was created for your app (highlighted below).

Now we want to add the Visual Recognition service to your app, so click "ADD A SERVICE OR API"


<img src="/assets/ionic/addservice.png" alt="Drawing" style="width: 400px;"/>

You might have to do some searching here, but once you find the Visual Recognition service, click on it

<img src="/assets/ionic/watson.png" alt="Drawing" style="width: 200px;"/>

And create

<img src="/assets/ionic/create.png" alt="Drawing" style="width: 200px;"/>

After you creat the service, you will be prompted to "restage" your app.  Go ahead and do this.  All
it means is that your app will be brought down and then back up.  This will update the environment with the details
for the new service you just added.

<img src="/assets/ionic/restage.png" alt="Drawing" style="width: 400px;"/>

### Enough horsing around.  Lets write some code

Click on 'Overview' on the far left so that we can get back to our app.  We want to add git, 
grab the boilerplate code, and start making some changes.

(If you are requested for you user info again, and that fails, then log out and in again)

<img src="/assets/ionic/addgit.png" alt="Drawing" style="height: 75px;"/>

After a few clicks your git repo will be created. Click on the git url and we will be taken
to DevOps services where we can edit our server code.

<img src="/assets/ionic/giturl.png" alt="Drawing" style="height: 150px;"/>

Now click on the big "Edit Code" button. Its so big I don't even need to show you a picture of it!
We can now see, and edit, our server.  Before we do that though, lets prove that it is running.
Click on the "open application" button.  This will start a new tab that goes to your route.

<img src="/assets/ionic/openapplication.png" alt="Drawing" style="height: 50px;"/>

We are about to start changing this code so that it does something interesting.  Enable "Live Edit"
so that we can do this without waiting 5 minutes for a full deploy after each change.

___Live Edit:
   You can make changes to a Node.js application running in Bluemix and test them in your browser right away. Any changes that you make in a synchronized desktop directory or in the Web IDE are propagated to the application’s file system immediately.___
   
![](/assets/ionic/livedit.png)

The message that you saw after opening your app is nothing more than the html in your public/index.html.
Open this file up and you see some generic text.  Make some changes to it... maybe a statement on your love for boy bands or Bob Hope.
After that you can go back to your webpage and see the sweet changes.  

We don't care about Bob Hope, we want to upload a file to our server and send it to Watson. 
Blow away whatever is in public/index.html and put this html in there instead

{% highlight html %}
<form action="/upload" method="POST" enctype="multipart/form-data">
  *in a robot voice* Select image to upload!<br>
  <input type="file" name="image"> 
  <input type="submit" value="Upload Image">
</form>
{% endhighlight %}

We need to add 2 more packages to our server; formidable for file uploads, and the Watson Developer Cloud for Visual Recognition.
Update your package.json with this

{% highlight json %}
{
	"name": "MobileBackendStarterApp",
	"version": "0.0.1",
	"description": "A template Mobile Backend application",
	"dependencies": {
		"express": "4.*",
		"passport": "*",
		"passport-imf-token-validation": "*",
		"imf-oauth-user-sdk": "*",
		"watson-developer-cloud": "*",
		"formidable": "*"
	},
	"engines": {
		"node": "0.10.26"
	},
	"repository": {},
	"scripts": {
	  "start": "node app.js"
	}
}
{% endhighlight %}

The. go ahead and redeploy so that the new packages will be installed by npm

![](/assets/ionic/deploy.png)

### Using Watson Visual Recognition
In the project listing click on the app.js file.  This is the entry point for our Node Express server.
 
Copy the below code and paste it in at line 52.  This creates a new route for us that will
send the uploaded file to our Visual Recognition service for analysis.

{% highlight javascript %}
var formidable = require('formidable');
var watson = require('watson-developer-cloud');
var fs = require('fs');

var visual_recognition = watson.visual_recognition({
    version: 'v1',
    use_vcap_services: true
});

app.post('/upload', function(req, result) {

    var form = new formidable.IncomingForm();
    form.keepExtensions = true;

    form.parse(req, function(err, fields, files) {
        var params = {
            image_file: fs.createReadStream(files.image.path)
        };

        visual_recognition.recognize(params, function(err, res) {
            if (err)
                res.send(err);
            else {
                var labels = res.images[0].labels.map(function(label)
                {
                    return label.label_name;
                });

                result.send(labels);
            }
        });

    });
});
{% endhighlight %}

Go to your webpage and lets upload an image again.  Make sure to pick a photo and not a graphic or drawing.
Watson will respond with a list of labels that it thinks fits the photo you uploaded.

### Build an Ionic app to hit your server

Refer back to [Camden's Blog](http://www.raymondcamden.com/2015/08/05/a-real-world-app-with-ibm-bluemix-node-cordova-and-ionic) for the mobile instructions.

* Special note for you Xcode users.  Edit the info.plist and add this key

{% highlight xml %}
<key>NSAppTransportSecurity</key>
<dict>
  <!--Include to allow all connections (DANGER)-->
  <key>NSAllowsArbitraryLoads</key>
      <true/>
</dict>
{% endhighlight %}
