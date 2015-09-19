---
layout: post
title:  "Ionic Environment Setup"
date:   2015-09-5 18:01:11
categories: bluemix ionic
---

The following are setup instructions for [Raymond Camden's Blog](http://www.raymondcamden.com/2015/08/05/a-real-world-app-with-ibm-bluemix-node-cordova-and-ionichttp://www.raymondcamden.com/2015/08/05/a-real-world-app-with-ibm-bluemix-node-cordova-and-ionic)
about creating a Bluemix enabled Ionic mobile app.

### Setup 

  Follow the below instructions to get your environment setup for working with Bluemix and Ionic.
  Note that the `$` symbolizes that you are in the command line. Duh.

* You will need the mobile SDKs for Android or iOS.  If you do not have one installed then grab the
  Android SDK and get its install going right away (this will take... a while)
  
  Install the [Android Studio](https://developer.android.com/sdk/index.html)
  
  #or
  
  Get the [Android SDK](https://developer.android.com/sdk/index.html#Other) zip 
  
  [Android SDK install instructions](https://developer.android.com/sdk/installing/adding-packages.html)
    
* [Sign up](https://ibm.biz/BluemixATX "Signup for Bluemix") with Bluemix so that we can use the Visual Recognition service.
    
* [Download](https://github.com/cloudfoundry/cli/releases) the Cloud Foundry command line interface.  This can be run in OSX Terminal or Win Command Prompt (Cygwin not supported)
    
  `$ cf api https://api.ng.bluemix.net`
    
  `$ cf login`

* [Node](https://nodejs.org/en/) will need to be installed so that you can use npm

* Install Cordova and Ionic

  `$ sudo npm install -g cordova ionic`
    
* Get the code for the workshop.  If you already have git installed this is easy:
 
  `$ git clone https://github.com/cfjedimaster/IonicBluemixDemo`
  
  If you don't then simply [download the zip from here](https://github.com/cfjedimaster/IonicBluemixDemo/archive/master.zip)
  
  After cloning or unzipping you should now have a directory named IonicBluemixDemo with the code
  
* Install the node packages for the demo
  
  `$ cd IonicBluemixDemo/server`
  
  `$ npm install`

  `$ cd ..` just to get back to the root dir
      
* Now, this is jumping ahead a bit, but lets create our Ionic mobile project.  Make sure you are in 
  the `/IonicBluemixDemo` root

  `ionic start mymobileapp ./mobile/www`

  `cd mymobileapp`
  
* Get the cordova plugings that you will need for the workshop

  `cordova plugin add cordova-plugin-camera`
  
  `cordova plugin add cordova-plugin-file-transfer`

* Special note for you Xcode users.  Edit the info.plist and add this key

{% highlight xml %}
<key>NSAppTransportSecurity</key>
	<dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
		<key>NSExceptionDomains</key>
		<dict>
			<key>localhost:3000</key>
			<dict>
				<key>NSIncludesSubdomains</key>
				<true/>
				<key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
				<true/>
				<key>NSTemporaryExceptionMinimumTLSVersion</key>
				<string>TLSv1.1</string>
			</dict>
		</dict>
	</dict>
{% endhighlight %}
