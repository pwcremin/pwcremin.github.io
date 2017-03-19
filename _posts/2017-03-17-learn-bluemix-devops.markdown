---
layout: post
comments: true
title:  "Devops Walkthrough"
date:   2017-03-17 18:01:11
categories: bluemix devops
---


After logging into Bluemix, open the menu and select Apps|Dashboard. This gives you an overview of your running apps and services with options to create new ones.  We are going to create a new app so click on Create App.

Under All Services select Apps|Boilerplates. Bluemix gives you a number of boilerplates to help get you started. Select the Node.js Cloudant DB Web Starter.  This will set us up with an app that is bound to a Cloudant database and starter code for our Node server.

<br>
<br>
<img src="/assets/devops/node-boilerplate.png" width="200"/>
<br>
<br>

Your app just needs a unique name and then hit Create.  After a few moments your server will be running using the starter code.  Click on your server's url to see the running server.


<br>
<br>
<img src="/assets/devops/server-link.png" width="200"/>
<br>
<br>

Lets take a look at some of the Bluemix Devops features.  You are going to need to have a Github account so please create one if needed.
Click on Overview.  This gives you a breakdown of your running app, and in the bottom right corner you can enable Continuous Delivery. Do it.

<br>
<br>
<img src="/assets/devops/continuous-delivery.png" width="200"/>
<br>
<br>

The toolchain is preconfigured for continuous delivery, source control, blue-green deployment, functional testing, issue tracking, online editing, and alert notification.  This is a setup that many companies need dedicated employees to develop and maintain, but you get with ease using Bluemix. Create the toolchain. (Note, if you haven't done this before you are going to need to enter your Github credentials)

<br>
<br>
<img src="/assets/devops/toolchain-overview.png" width="200"/>
<br>
<br>

To see the toolchain in action, open the Eclipse Orion IDE.  This online editor will allow us to make code changes without the need for setting up your local environment.  You can also debug your live server and view logs files using the tool.


<br>
<br>
<img src="/assets/devops/eclipse-ide.png" width="200"/>
<br>
<br>

Simply make a few changes to views/index.html and then, on the lef side, select the git button <img src="/assets/devops/git-button.png" width="40"/> button so that we can commit and push the change.  Also, go back to you toolchain and open the Delivery Pipeline in a differnt window so that we can see it in action.

<br>
<br>
<img src="/assets/devops/delivery-pipeline.png" width="200"/>
<br>
<br>

Commit your change, push it, and then quickly jump to your Delivery Pipeline window.  The pipeline recogize that there is a new commit and kick off the build and deploy stages.  Here you would also want to add a stage for running test cases so that every commit to your repo gets tested.

<br>
<br>
<img src="/assets/devops/delivery-stages.png" width="200"/>
<br>
<br>
