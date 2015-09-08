---
layout: post
title:  "Python CRUD REST api with Cloudant"
date:   2015-09-5 18:01:11
categories: bluemix python
---

### Bluemix Setup 

* [Sign up](https://console.ng.bluemix.net/ "Signup for Bluemix").  With Bluemix we can easily setup Python hosting and a Cloudant database.

* [Download](https://github.com/cloudfoundry/cli/releases) the Cloud Foundry command line interface.  This can be run in OSX Terminal or Win Command Prompt (Cygwin not supported)

  `$ cf api https://api.ng.bluemix.net`

  `$ cf login`

* Create and bind your Cloudant service

  `$cf marketplace -s cloudantNoSQLDB`

  You can see from the output that Bluemix will give you plenty without there being any cost, so lets get this service going:

  `cf create-service cloudantNoSQLDB Shared [app_name]-cloudantNoSQLDB`
  
  `cf bind-service [app-name] [app_name]-cloudantNoSQLDB`
  
  TODO: may not need this - `cf restage [app-name]`


* Update your manifest.yml so that it knows you are using the new service:
  
  {% highlight yaml %}
  applications:
  - services:
    - [app_name]-cloudantNoSQLDB
  - disk_quota: 1024M
    host: [app_name]
    name: [app_name]
    path: .
    domain: mybluemix.net
    instances: 1
    memory: 128M
  {% endhighlight %}  

* Now all we have to do is push our app and we will have a hosted Python server!

  `$ cf push [app_name]`

  Once the server has successfully started, navigate to [app_name].mybluemix.net

* Get environment so that you can run locally
    
  `$ cf env [app_name]`
  
  Copy the VCAP_SERVICES block and stick it into a file named .env.vcap_services.json.
  Add this file to both your .cfignore and .gitignore.
  
  .env.vcap_services.json
  {% highlight json %}    
  {
   "VCAP_SERVICES": {
    "cloudantNoSQLDB": [
     {
      "credentials": {
       "host": "xxxx-bluemix.cloudant.com",
       "password": "xxxx",
       "port": 443,
       "url": "https://xxxx-bluemix:xxxx@xxxx-bluemix.cloudant.com",
        "username": "xxxx-bluemix"
      },
      "label": "cloudantNoSQLDB",
      "name": "[app_name]-cloudantNoSQLDB",
      "plan": "Shared",
      "tags": [
        "data_management",
        "ibm_created",
        "ibm_dedicated_public"
      ]
     }
    ]
   }
  }
  {% endhighlight %}  
  
   
  
### Create the CRUD Rest API

* Get the starter code for a Python Flask server.  This starter code will have everything we
  need to run our server in Bluemix later.

  `$ git clone https://github.com/pwcremin/pythonstarter [app_name]`

  replace [app_name] with a unique name

  `$ cd [app_name]`

* Install Flask and Cloudant python modules.  You will need [pip](https://pip.pypa.io/en/latest/installing.html#install-or-upgrade-pip) if it is not already installed.
  These modules are already in your requirements.txt
  
  `$ pip install flask`
  
  `$ pip install cloudant`

Now we have a pretty basic Flask server. Run this locally and hit [http://localhost:8000](http://localhost:8000) to make sure it works.

{% highlight python %}
import os
from flask import Flask, jsonify

# Read port selected by the cloud for our application
PORT = int(os.getenv('VCAP_APP_PORT', 8000))
# Change current directory to avoid exposure of control files
os.chdir('static')

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
  print("Start serving at port %i" % PORT)
  app.run('', PORT)
{% endhighlight %}

To implement the CRUD operations (create, read, update, delete), we will need to connect
to our Cloudant db.  Add the below code and you will be able to connect and create the db.

{% highlight python %}
app = Flask(__name__)

if 'VCAP_SERVICES' in os.environ:
    cloudantInfo = json.loads(os.environ['VCAP_SERVICES'])['cloudantNoSQLDB'][0]
#we are local
else:
    with open('.env.vcap_services.json') as configFile:
        cloudantInfo = json.load(configFile)['VCAP_SERVICES']['cloudantNoSQLDB'][0]

username = cloudantInfo["credentials"]["username"]
password = cloudantInfo["credentials"]["password"]

account = cloudant.Account(username)
login = account.login(username, password)
assert login.status_code == 200

# create a database object
db = account.database('calls')
# now, create the database on the server
response = db.put() 
print response.json()
{% endhighlight %}

Delete the boring HelloWorld code, and put this more exciting GET call in its place

{% highlight python %}
@app.route('/api/v1/calls', methods=['GET'])
def getCalls():
    response = db.all_docs().get(params={'include_docs': True})

    if response.status_code != 200:
        abort(response.status_code)

    rows = json.loads(response.content)['rows']

    docs = map(lambda row: row['doc'], rows)

    return jsonify({'calls': docs}), response.status_code
{% endhighlight %}

Run the server and navigate to [http://localhost:8000/api/v1/calls](http://localhost:8000/api/v1/calls).
Ok, pretty cool that you can data from the server, but its not very interesting yet.

Using your browser is fine for GET calls, but we need something more powerful to test
out the rest of our api. [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)
is an awesome tool for this.  Add the tool to Chrome and lets use it for our request.

![](/assets/pythoncrud/postman1.png)

Click the down arrow next to the floppy disk icon so that you can save this request to a collection (name the collection whatever you want).

Create the POST endpoint so that we can add data:

{% highlight python %}
@app.route('/api/v1/calls', methods=['POST'])
def createCall():
    if not request.json:
        abort(400)

    call = {
        'name': request.json['name'],
        'eventhost': request.json['eventhost'],
        'time': request.json['time'],
        'address': request.json['address'],
    }

    response = db.document('', params=call).post()

    return response.content, response.status_code
{% endhighlight %}

Change your Postman request to POST, and update the request like below.  Make sure to save this
request as well. You can run this POST a few times, and then try your GET request again.  Look at that, it works.

![](/assets/pythoncrud/postman2.png)

We do not always, or rarely even, want to GET every document in our database.  Lets
make a GET call that will only return the specified doc by id.

{% highlight python %}
@app.route('/api/v1/calls/<string:id>', methods=['GET'])
def getCall(id):
    response = db.get(id)

    if response.status_code != 200:
        abort(response.status_code)

    doc = json.loads(response.content)

    return jsonify(doc), response.status_code
{% endhighlight %}

At this point you should have the gist of how to create a REST api.  Lets go ahead
and knock out the last two CRUD endpoints; update and delete


{% highlight python %}
@app.route('/api/v1/calls/<string:id>', methods=['PUT'])
def updateCall(id):
    if not request.json:
        abort(400)

    document = db.document(id)

    response = document.merge(request.json)

    return '', response.status_code
{% endhighlight %}

{% highlight python %}
@app.route('/api/v1/calls/<string:id>', methods=['DELETE'])
def deleteCall(id):
    response = db.get(id)

    if response.status_code != 200:
        abort(response.status_code)

    doc = json.loads(response.content)

    response = db.delete(id, params={'rev': doc['_rev']})

    return '', response.status_code
{% endhighlight %}
