---
layout: post
comments: true
title:  "Discovery News for Bitcoin"
date:   2017-03-15 18:01:11
categories: bluemix watson discovery news bitcoin
---

For the past year, the biggest movements in Bitcoin have been caused by China.  Lets use [Discovery News](https://www.ibm.com/watson/developercloud/discovery-news.html) to track news stories about Bitcoin where China is involved and use sentiment analysis to see if the stories are trending positive or negative.

Full source for this demo is (available on Github)[https://github.com/pwcremin/discovery-news-bitcoin-watcher]
### Setup 

#### Create your Bluemix app and bind the Watson Discovery service

Log into Bluemix, go to Catalog, and, under Services, select the Watson services.  Here you see a list of all Watson services that you can use.  We want Discovery so click on it and then hit the Create Button

<div>
<img src="/assets/discovery-bitcoin/catalaog.png" width="200"/>
<img src="/assets/discovery-bitcoin/watson-service.png" width="200"/>
<img src="/assets/discovery-bitcoin/discovery_service.png" width="350"/>
</div>

Go to the menu, select Services, and then Dashboard.  Here you will see your newly created Discovery service.  Click on it.

<img src="/assets/discovery-bitcoin/view_service.png" width="350"/>

In order to access this service we are going to need the service credentials. Click on Service Credentials and then View the credentials.  You will need your username and password, so save this data.

Now lets get setup for Watson News.  Click on Manage and then Launch Tool.  

Using Discovery you can upload your own files, which can then be searched using the power of Discovery, or search the predefined dataset of Discovery News, which is what we are going to do.

<img src="/assets/discovery-bitcoin/your_data.png" width="350"/>

Click on Watson News and you will see the environment id and collection id.  Save this data as we will need it when telling the Discovery API what dataset we are querying.

#### Setup Node Express

We are going to create a Node Express server with routes that return the results of our queries.  I am going to assume that you know how to setup your environment for Node and get an Express server going.  In addition we are going to grap the watson-developer-cloud package which will allow us to easily make calls to the Discovery service.

```
npm install watson-developer-cloud --save
```

Now we have everything we need. I will assume your Express server has a routes/index.js, but use any route you want.  Lets add this setup code to index.js.  Replace username and password with your own service credentials that you gathered earlier.

```javascript
var express = require( 'express' );
var router = express.Router();
var DiscoveryV1 = require( 'watson-developer-cloud/discovery/v1' );

var discovery = new DiscoveryV1( {
    username: "xxx",
    password: "xxx",
    version_date: DiscoveryV1.VERSION_DATE_2016_12_15
} );
```

### Creating your first query

First do a simple query that just returns all articles that have Bitcoin in their text.

```javascript
var discovery = new DiscoveryV1( {
    username: "xxx",
    password: "xxx",
    version_date: DiscoveryV1.VERSION_DATE_2016_12_15
} );

discovery.query( {
    environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
    collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

    query: 'text:Bitcoin'
    
}, function ( err, response )
{
    if ( err )
    {
        console.error( err );
    }
    else
    {
        let queryResult = JSON.stringify( response, null, 2 )

        console.log( queryResult );
    }
} );
```

Run your server and you will see output like below:

```json
{
      "id": "585342f6989c856f20eb371252e1e70d",
      "score": 4.7683716,
      "totalTransactions": "17",
      "docSentiment": {
        "mixed": "1",
        "score": "0.390088",
        "type": "positive"
      },
      "taxonomy": [
        {
          "score": "0.905536",
          "label": "/technology and computing"
        },
        {
          "confident": "no",
          "score": "0.496139",
          "label": "/technology and computing/software/databases"
        },
        {
          "confident": "no",
          "score": "0.292148",
          "label": "/law, govt and politics/government/government agencies"
        }
      ],
      "enrichedTitle": {
        "status": "OK",
        "totalTransactions": "14",
        "docSentiment": {
          "type": "neutral"
        },
        "language": "english",
        "taxonomy": [
          {
            "score": "0.905536",
            "label": "/technology and computing"
          },
          {
            "confident": "no",
            "score": "0.156856",
            "label": "/law, govt and politics/legal issues/legislation"
          },
          {
            "confident": "no",
            "score": "0.119892",
            "label": "/law, govt and politics/politics/lobbying"
          }
        ],
        "relations": [],
        "entities": [
          {
            "count": "1",
            "sentiment": {
              "type": "neutral"
            },
            "knowledgeGraph": {
              "typeHierarchy": "/publications/travel weekly"
            },
            "text": "Travel Weekly",
            "type": "PrintMedia",
            "relevance": "0.33",
            "disambiguated": {
              "freebase": "http://rdf.freebase.com/ns/m.0h3v7h6",
              "dbpedia": "http://dbpedia.org/resource/Travel_Weekly",
              "name": "Travel Weekly"
            }
          }
        ],
        "text": "Lawmakers want state to explore bitcoin technology: Travel Weekly",
        "keywords": [	
```

The resulting dataset contains all news stories with Bitcoin in the text (default count of 10 results).  We are interested in stories that also involve China, so lets update our query.


```javascript
discovery.query( {
    environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
    collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

    query: 'text:Bitcoin,China'
    
}, 
```

The ',' is the boolean operater 'and'.  You can find a list of <a href="https://www.ibm.com/watson/developercloud/doc/discovery/query-reference.html#parameter-descriptions">all query operators here</a>.

Run the query again and you will see that the result now contains stories that include both Bitcoin and China. Going through the results you see that there are items you really do not care about.  If you look at the 'taxonomy' of a result, Discovery gives you a breakdown of the categories the story belongs to.  Some of these categories we do not care about, and do not want them in our results.  For instance, I was getting stories about Pets.  Definitely not the kind of results that I want, so lets exclude those.  I also see that 'finance' is an option.  I definitely want those.  Also, I am not interested in old stories so lets just get then items that are a few days old.  Note that 'yyyymmdd' is not some special search marker.  Its simple a value that was in our results that we can now use to create a better query.  You can do this with any value that you wish.

```javascript
discovery.query( {
    environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
    collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

    query: 'text:Bitcoin,China,' +
    'yyyymmdd:>20170315,' +
    'taxonomy.label:finance,' +
    'taxonomy.label:!pets',
}, 
```

Run your server again and you see that we are getting much more relevant information.  However, lots of the stories seem to be the same.  Not a big suprise here, these financial blogs seems to all copy the big story.  It is likely that if many people are writing about the same story, it is probably something we will care about.  Lets group together stories with the same title using 'aggregate.  Also, lets tell the query to only return the values that we want using 'return'

```javascript
discovery.query( {
    environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
    collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

    query: 'text:Bitcoin,China,' +
    'yyyymmdd:>20170315,' +
    'taxonomy.label:finance,' +
    'taxonomy.label:!pets',
    
    aggregation:'term(enrichedTitle.text,count:10)',
    
    return: 'enrichedTitle.text,url,text,taxonomy,docSentiment' 
}, 
```

We are now getting back a dataset that we can do something with. The sentiment score is especially handy. As expected, a positive or negative score expresses the general sentiment of the article.  The bigger the number, the better/worse the sentiment.  Lets add an aggregation that will tell of the average sentiment of the results.

```javascript
discovery.query( {
    environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
    collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

    query: 'text:Bitcoin,China,' +
    'yyyymmdd:>20170315,' +
    'taxonomy.label:finance,' +
    'taxonomy.label:!pets',
    
    aggregation:'[term(enrichedTitle.text,count:10),average(docSentiment.score)]',
    
    return: 'enrichedTitle.text,url,text,taxonomy,docSentiment' 
}, 
```

At the time of writing, I received the following results

```json
 "aggregations": [
    {
      "type": "term",
      "field": "enrichedTitle.text",
      "count": 10,
      "results": [
        {
          "key": "Bitcoin could be on the edge of a cliff",
          "matching_results": 13
        },
        {
          "key": "Judge approves $27 million settlement between Lyft, drivers",
          "matching_results": 5
        },
        {
          "key": "AmTrust cites errors in financials since 2014",
          "matching_results": 4
        },
        {
          "key": "Brazil judge suspends $50B dam disaster lawsuit",
          "matching_results": 4
        },
        {
          "key": "Yen stuck in narrow range ahead of G-20 meeting",
          "matching_results": 4
        },
        {
          "key": "News Highlights : Top Global Markets News of the Day",
          "matching_results": 3
        },
        {
          "key": "Bitcoin could be on the edge of a cliff - Business Insider",
          "matching_results": 2
        },
        {
          "key": "News Highlights : Top Financial Services News of the Day",
          "matching_results": 2
        },
        {
          "key": "Subscribe to read",
          "matching_results": 2
        },
        {
          "key": "6 things Australian traders will be talking about this morning",
          "matching_results": 1
        }
      ]
    },
    {
      "type": "average",
      "field": "docSentiment.score",
      "value": -0.45740894323045556
    }
  ]
```

A negative score of -0.45.  The sentiment score ranges from -1 to 1, so that is pretty bad.  Of course, right now Bitcoin is getting hammered by the possibility of new regulations from China so this score is what I was expecting to get.  This result is also skewed by the large number of articles on the same title "Bitcoin could be on the edge of a cliff".  Something you may or may not want.

Finally, lets go ahead and return these results:

```javascript
router.get( '/news', function ( req, res, next )
{
    discovery.query( {
        environment_id: '9d72194c-bfa5-41ec-b47b-00b7a1943612',
        collection_id: '8854ecb2-6c2b-4bfd-acca-8333a9f4b64e',

        query: 'text:Bitcoin,China,' +
        'yyyymmdd:>20170315,' +
        'docSentiment.score:<-0.1,' +
        'taxonomy.label:finance,' +
        'taxonomy.label:!pets',
        aggregation:'[term(enrichedTitle.text,count:10),average(docSentiment.score)]',
        count: 10,
        return: 'enrichedTitle.text,url,text,taxonomy,docSentiment'

    }, function ( err, response )
    {
        if ( err )
        {
            console.error( err );
            res.status( 500 ).json( err )
        }
        else
        {
            let queryResult = JSON.stringify( response, null, 2 )

            console.log( queryResult );

            res.json( queryResult);
        }
    } );
} );
```
