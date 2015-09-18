---
layout: post
title:  "Ionic Environment Setup"
date:   2015-09-5 18:01:11
categories: bluemix ionic
---

### Setup 

  Follow the below instructions to get your environment setup for working with Bluemix and Ionic.
  Note that the `$` symbolizes that you are in the command line. Duh.
  
* [Node](https://nodejs.org/en/) will need to be installed so that you can use npm

* Install Cordova and Ionic

  `$ sudo npm install -g cordova ionic`
  
  `$ sudo npm install -g plugman`
  
  plugman is a command line tool to install plugins for cordova

* You will need the mobile SDKs for Android or iOS.
  
  #Android:
  
    Windows: [installer_r24.3.4-windows.exe](http://dl.google.com/android/installer_r24.3.4-windows.exe)
  
    Mac OS X: [android-sdk_r24.3.4-macosx.zip](http://dl.google.com/android/android-sdk_r24.3.4-macosx.zip)

  #Xcode
   
    It is best to have the full Xcode installed, but to save time just grab the command line tools:
    
    `$ xcode-select --install`
    
* Get the code for the workshop.  If you already have git installed this is easy:
 
  `$ git clone https://github.com/cfjedimaster/IonicBluemixDemo`
  
  If you don't then simply [download the zip from here](https://github.com/cfjedimaster/IonicBluemixDemo/archive/master.zip)

* [Sign up](https://ibm.biz/BluemixATX "Signup for Bluemix") with Bluemix so that we can easily setup Python hosting and a Cloudant database.
  
* [Download](https://github.com/cloudfoundry/cli/releases) the Cloud Foundry command line interface.  This can be run in OSX Terminal or Win Command Prompt (Cygwin not supported)
  
    `$ cf api https://api.ng.bluemix.net`
  
    `$ cf login`