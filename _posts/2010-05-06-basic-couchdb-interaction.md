---
title: Basic CouchDB interaction
author: Vegar
layout: post
permalink: /2010/basic-couchdb-interaction/
categories:
  - Development
tags:
  - CouchDB
---
<img src="http://blog.vi-kan.net/wp-content/uploads/2010/05/image3.png" alt="" title="http://ditchnet.org/httpclient/" />So, did you read my little <a href="http://blog.vi-kan.net/2010/introducing-couchdb/">introduction</a>? Did you go nuts with the Futon application?   Well, then, I guess it's time to look a little closer on the CouchDB API.

As mentioned before, CouchDB provides a RESTful API. You will be sending requests over HTTP, and checking status codes for success. This makes it easy to test features, by using small utilities like <a href="http://ditchnet.org/httpclient/">HTTP Client</a> and <a href="http://curl.haxx.se/">Curl</a>. All GET operations should be easy enough to do through your browser, and some browsers have addons that lets you make other kind of requests too. So find a tool that suites you well, and start the expedition.

## Say Hello to CouchDB

Before we start making databases and documents, you should check that the server is running. Make a simple GET request to the server to see if you get any response.

```
GET http://127.0.0.1:5984/
```

If the server is running, it will respond with a welcome message and a version number.

```
{"couchdb":"Welcome","version":"0.11.0"}
```

You can also check to see what databases allready exists on the server, by doing a GET request to /_all_dbs.

```
GET http://127.0.0.1:5984/_all_dbs
```

The resonse will be formatted as an JSON array with database names.

```
["business-cards","newdb","testdb"]
```

## Databases

CouchDB exposes most everything as resources accessable by urls. This is one part of what makes it RESTful. So when you want to access one of the existing databases, you will make a GET request for the name of the database.

```
GET http://127.0.0.1:5984/business-cards
```

This request want return the complete database with all its documents. It will instead return some information about the database.

```
{"db_name":"business-cards", "doc_count":2, "doc_del_count":1, "update_seq":4, "purge_seq":0, "compact_running":false, "disk_size":16473, "instance_start_time":"1273129328746881", "disk_format_version":5}
```

As you can see, our business card database from our introduction contains two documents. It also contains one deleted document which is not purged yet.

Since the name of the database is used as an url, there has to be some constraints on what characters can be used. To make it easy to support different platforms, and avoid issues with casing, CouchDB will only allow lowercase letters in the range 'a' to 'z'. It will also allow digits and the following characters: _ $ ( ) – /

By allowing the /-character, it is possible to group different databases into namespaces, like /blog/posts and /blog/comments. It can be a little tricy to test, though. You will need to encode the / into /, but HTTP Client, which I use, doesn't like that. It will encode the % into % and send %2F instead of /. I guess there is some way of escapting this, but I don't know for sure.

So lets create our first database. I will make myself a database of geocaches. So what would be more natural then just to PUT it right there at the server?

```
PUT http://127.0.0.1:5984/geocaches
```

If the database was created as expected, we will get a positive response.

```
{"ok":true}
```

If the database name is taken, we will get an error response. The response will contain a standard HTTP status code, 412 Precondition Failed, and the body of the response will give us even more information.

```
{"error":"file_exists","reason":"The database could not be created, the file already exists."}
```

Similar, if we try to use one of the forbidden characters, we will get a 400 Bad Request with additional information on how to behave in the future.

```
{"error":"illegal_database_name", "reason":"Only lowercase characters (a-z), digits (0-9), and any of the characters _, $, (, ),  , -, and / are allowed"}
```

Nice.

If you need to remove the database again, you make a…..?  Yes, a DELETE request.

```
DELETE http://127.0.0.1:5984/geocaches
```

If everything goes well, the server will respond with a happy 'ok', and the database with all its documents is gone. No 'please, confirm' or 'are you sure', so now you're warned.

## Documents

In the world of CouchDB, a document consists of key-value pairs. Every document needs an unique id, which is stored in the '_id'-key. When you create a new documet, CouchDB will provide one for you if you want it too. The id will then be in the form of a guid, a string of 32 hexadecimal values.

The id of a document will be used as part of the uri when you later want to retrieve or change your document. If the data you want to store already have a unique identification, you could use that and get more domain specific urls. Be careful though. Not every character is suitable to be part of an uri, and you would need to escape them appropriately .

## Create

I want to store geocaches, and every geocache has an unique name consisting of letters and digits. I will therefore use this name as the id for my document. Heres my first geocache:

```
{
"url": "http://www.geocaching.com/seek/cache_details.aspx?guid=ab5cc0fb-4f0f-4809-98a1-60697daa5e08",
"sym": "Geocache Found",
"lon": 18.973967,
"lat": 69.696533,
"urlname": "Nesten hjemme",
"type": "Geocache|Traditional Cache",
"time": "2009-06-07T07:00:00Z",
"name": "GC1TCJR",
"desc": "Nesten hjemme by Geo_Raiders, Traditional Cache (2/1.5)"
}
```

I want to store this as an document with the key GC1TCJR in the database geocaches, so lets PUT it up there.

```
PUT http://127.0.0.1:5984/geocaches/GC1TCJR

{"url":"http://www.geocaching.com/seek/cache_details.aspx?guid=……."}
```

If everything goes by the plan, CouchDB will responde with a 201 Created status, together with something like this:

```
{"ok":true,"id":"GC1TCJR","rev":"1-f70033bafd9d72e174cecb67f58b2e4f"}[/js]
```

If a document with the same id already exists, you will get a 409 Conflict status and an error.

You may wonder what the "rev" value returned is for. CouchDB will never change a document. Instead, it will make a new document stored with the same id. The reason for this is to handle concurrency, and you should not rely on this for any revision control in your application. You can compact a database to get rid of wasted space.

If we want CouchDB to generate a id for us, we could use a POST instead. We would also omit the id from the url, since the id yet remains to be decided.

```
POST http://127.0.0.1:5984/geocaches

{"url":"http://www.geocaching.com/seek/cache_details.aspx?guid=……"}
```

Now that we have successfully stored our document, we can retreive it by doing a GET.

```
GET http://127.0.0.1:5984/geocaches/GC1TCJR
```

As you can see from the response, CouchDB has added the id and revision fields to our document.

```
"_id":"GC1TCJR","_rev":"1-f70033bafd9d72e174cecb67f58b2e4f","url":"http://www.geocaching.com/……"}
```

## Update

Storing updates is nearly as simple as creating new documents. The only difference is that the document must contain both the id and the rev values, and the rev value needs to point to the latest revision. So if you get a document first, make your changes, and then put it up on the server again, things should go nicely. If someone else has stored an update between your get and put request, you will get an conflict error that you need to resolve. How you resolve the conflict is up to the you. You could try to merge your changes with the new revision in the database, or you could simply update your rev value with the latest from the databse and do a new put, effectively overwriting what changes that may have been done.

```
PUT http://127.0.0.1:5984/geocaches/GC1TCJR

{"_id":"GC1TCJR","_rev":"1-f70033bafd9d72e174cecb67f58b2e4f","url":"http://www.geoinfo.com/……
```

CouchDB will respond with a 200 OK status, and inform you about the new revision number.

```
{"ok":true,"id":"GC1TCJR","rev":"2-6aa44736b02f96844fdfbf4eece00b83"}
```

## Delete

Can you guess how to delete a document? Close. You make a DELETE request, but you have to include the latest revision in the url as well.

```
DELETE http://127.0.0.1:5984/geocaches/GC1TCJR?rev=1-f70033….
```

As for updating, CouchDB keep your old document around for a while. It will add a new revision, and mark it as a stub for a deleted document.

The response will tell you the new revision number.

```
{"ok":true,"rev":"3-6b8ecfd8bd5d1cb8ae3fd49aea9dfe8d"}
```

If you try to do a get on the deleted document, you will get a 404 Object not found response with additional information stating that the document was deleted.

```
{"error":"not_found","reason":"deleted"}
```

## Listing

Now that you know the basic CRUD operations, it would be nice to get a list of all our documents. CouchDB provides a special url we can use for this.

```
GET http://127.0.0.1:5984/geocaches/_all_docs
```

The response will contain information of total number of rows, and an array of ids and revision numbers.

```
{"total_rows":4,"offset":0,"rows":[

{"id":"GC13H9E","key":"GC13H9E", "value":{"rev":"1-8e3c239c5da38d0fbe91ffdd24582b02"}},

{"id":"GC144MQ","key":"GC144MQ", "value":{"rev":"1-865f40ed3047aefdf8c45520c84c4321"}},

{"id":"GC145KB","key":"GC145KB", "value":{"rev":"1-83ba674d10ce404be7fc90460b1a9a13"}},

{"id":"GCYXEG","key":"GCYXEG", "value":{"rev":"1-880c0d1767c8f6db0ce815fe0b35afdb"}}

]}
```

The _all_docs uri can take some parameters that makes it possible to paginate the result. You can use a combination of 'startkey', 'endkey', 'limit' and 'skip'. Here is a couple of examples that you can try:

```
GET http://127.0.0.1:5984/geocaches/_all_docs?limit=2

=> First two documents

GET http://127.0.0.1:5984/geocaches/_all_docs?limit=2&amp;skip=2

=> Third and fourth document

GET http://127.0.0.1:5984/geocaches/_all_docs?startkey=GC144MQ&amp;limit=2&amp;skip=1

=> Two next documents after GC144MQ
```

## What's next?

Well, then. You should now have enough information to start making basic use of CouchDB. There are still a lot to cover, though. Did you know that you can upload binary attachments to documents? You can. What you can't, though, is to query your data using SQL. Instead you will be making views, but that has to be waiting for now.