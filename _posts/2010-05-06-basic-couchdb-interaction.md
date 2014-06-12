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
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2010/05/image3.png" alt="" title="http://ditchnet.org/httpclient/" />So, did you read my little <a href="http://blog.vi-kan.net/2010/introducing-couchdb/">introduction</a>? Did you go nuts with the Futon application?   Well, then, I guess it&#8217;s time to look a little closer on the CouchDB API.</p>

<p>As mentioned before, CouchDB provides a RESTful API. You will be sending requests over HTTP, and checking status codes for success. This makes it easy to test features, by using small utilities like <a href="http://ditchnet.org/httpclient/">HTTP Client</a> and <a href="http://curl.haxx.se/">Curl</a>. All GET operations should be easy enough to do through your browser, and some browsers have addons that lets you make other kind of requests too. So find a tool that suites you well, and start the expedition.</p>

<p>Say Hello to CouchDB</p>

<p>Before we start making databases and documents, you should check that the server is running. Make a simple GET request to the server to see if you get any response.</p>

<p>GET http://127.0.0.1:5984/</p>

<p>If the server is running, it will respond with a welcome message and a version number.</p>

<pre><code>{&quot;couchdb&quot;:&quot;Welcome&quot;,&quot;version&quot;:&quot;0.11.0&quot;}
</code></pre>

<p>You can also check to see what databases allready exists on the server, by doing a GET request to /_all_dbs.</p>

<pre><code>GET http://127.0.0.1:5984/_all_dbs
</code></pre>

<p>The resonse will be formatted as an JSON array with database names.</p>

<pre><code>[&quot;business-cards&quot;,&quot;newdb&quot;,&quot;testdb&quot;]
</code></pre>

<p>Databases</p>

<p>CouchDB exposes most everything as resources accessable by urls. This is one part of what makes it RESTful. So when you want to access one of the existing databases, you will make a GET request for the name of the database.</p>

<pre><code>GET http://127.0.0.1:5984/business-cards
</code></pre>

<p>This request want return the complete database with all its documents. It will instead return some information about the database.</p>

<pre><code>{&quot;db_name&quot;:&quot;business-cards&quot;, &quot;doc_count&quot;:2, &quot;doc_del_count&quot;:1, &quot;update_seq&quot;:4, &quot;purge_seq&quot;:0, &quot;compact_running&quot;:false, &quot;disk_size&quot;:16473, &quot;instance_start_time&quot;:&quot;1273129328746881&quot;, &quot;disk_format_version&quot;:5}
</code></pre>

<p>As you can see, our business card database from our introduction contains two documents. It also contains one deleted document which is not purged yet.</p>

<p>Since the name of the database is used as an url, there has to be some constraints on what characters can be used. To make it easy to support different platforms, and avoid issues with casing, CouchDB will only allow lowercase letters in the range &#8216;a&#8217; to &#8216;z&#8217;. It will also allow digits and the following characters: _ $ ( ) – /</p>

<p>By allowing the /-character, it is possible to group different databases into namespaces, like /blog/posts and /blog/comments. It can be a little tricy to test, though. You will need to encode the / into /, but HTTP Client, which I use, doesn&#8217;t like that. It will encode the % into % and send %2F instead of /. I guess there is some way of escapting this, but I don&#8217;t know for sure.</p>

<p>So lets create our first database. I will make myself a database of geocaches. So what would be more natural then just to PUT it right there at the server?</p>

<pre><code>PUT http://127.0.0.1:5984/geocaches
</code></pre>

<p>If the database was created as expected, we will get a positive response.</p>

<pre><code>{&quot;ok&quot;:true}
</code></pre>

<p>If the database name is taken, we will get an error response. The response will contain a standard HTTP status code, 412 Precondition Failed, and the body of the response will give us even more information.</p>

<pre><code>{&quot;error&quot;:&quot;file_exists&quot;,&quot;reason&quot;:&quot;The database could not be created, the file already exists.&quot;}
</code></pre>

<p>Similar, if we try to use one of the forbidden characters, we will get a 400 Bad Request with additional information on how to behave in the future.</p>

<pre><code>{&quot;error&quot;:&quot;illegal_database_name&quot;, &quot;reason&quot;:&quot;Only lowercase characters (a-z), digits (0-9), and any of the characters _, $, (, ),  , -, and / are allowed&quot;}
</code></pre>

<p>Nice.</p>

<p>If you need to remove the database again, you make a…..?  Yes, a DELETE request.</p>

<pre><code>DELETE http://127.0.0.1:5984/geocaches
</code></pre>

<p>If everything goes well, the server will respond with a happy &#8216;ok&#8217;, and the database with all its documents is gone. No &#8216;please, confirm&#8217; or &#8216;are you sure&#8217;, so now you&#8217;re warned.</p>

<p>Documents</p>

<p>In the world of CouchDB, a document consists of key-value pairs. Every document needs an unique id, which is stored in the &#8216;_id&#8217;-key. When you create a new documet, CouchDB will provide one for you if you want it too. The id will then be in the form of a guid, a string of 32 hexadecimal values.</p>

<p>The id of a document will be used as part of the uri when you later want to retrieve or change your document. If the data you want to store already have a unique identification, you could use that and get more domain specific urls. Be careful though. Not every character is suitable to be part of an uri, and you would need to escape them appropriately .</p>

<p>Create</p>

<p>I want to store geocaches, and every geocache has an unique name consisting of letters and digits. I will therefore use this name as the id for my document. Heres my first geocache:</p>

<pre><code>{

&quot;url&quot;: &quot;http://www.geocaching.com/seek/cache_details.aspx?guid=ab5cc0fb-4f0f-4809-98a1-60697daa5e08&quot;,

&quot;sym&quot;: &quot;Geocache Found&quot;,

&quot;lon&quot;: 18.973967,

&quot;lat&quot;: 69.696533,

&quot;urlname&quot;: &quot;Nesten hjemme&quot;,

&quot;type&quot;: &quot;Geocache|Traditional Cache&quot;,

&quot;time&quot;: &quot;2009-06-07T07:00:00Z&quot;,

&quot;name&quot;: &quot;GC1TCJR&quot;,

&quot;desc&quot;: &quot;Nesten hjemme by Geo_Raiders, Traditional Cache (2/1.5)&quot;

}
</code></pre>

<p>I want to store this as an document with the key GC1TCJR in the database geocaches, so lets PUT it up there.</p>

<pre><code>PUT http://127.0.0.1:5984/geocaches/GC1TCJR

{&quot;url&quot;:&quot;http://www.geocaching.com/seek/cache_details.aspx?guid=…….&quot;}
</code></pre>

<p>If everything goes by the plan, CouchDB will responde with a 201 Created status, together with something like this:</p>

<pre><code>{&quot;ok&quot;:true,&quot;id&quot;:&quot;GC1TCJR&quot;,&quot;rev&quot;:&quot;1-f70033bafd9d72e174cecb67f58b2e4f&quot;}[/js]

&lt;p&gt;

&lt;br /&gt;If a document with the same id already exists, you will get a 409 Conflict status and the following error:

&lt;br /&gt;&lt;/p&gt;

1

&lt;p&gt;You may wonder what the &quot;rev&quot; value returned is for. CouchDB will never change a document. Instead, it will make a new document stored with the same id. The reason for this is to handle concurrency, and you should not rely on this for any revision control in your application. You can compact a database to get rid of wasted space.&lt;/p&gt;

&lt;p&gt;If we want CouchDB to generate a id for us, we could use a POST instead. We would also omit the id from the url, since the id yet remains to be decided.&lt;/p&gt;

1POST http://127.0.0.1:5984/geocaches

{&quot;url&quot;:&quot;http://www.geocaching.com/seek/cache_details.aspx?guid=……&quot;}
</code></pre>

<p>Now that we have successfully stored our document, we can retreive it by doing a GET.</p>

<pre><code>GET http://127.0.0.1:5984/geocaches/GC1TCJR
</code></pre>

<p>As you can see from the response, CouchDB has added the id and revision fields to our document.</p>

<pre><code>{&quot;_id&quot;:&quot;GC1TCJR&quot;,&quot;_rev&quot;:&quot;1-f70033bafd9d72e174cecb67f58b2e4f&quot;,&quot;url&quot;:&quot;http://www.geocaching.com/……&quot;}
</code></pre>

<p>Update</p>

<p>Storing updates is nearly as simple as creating new documents. The only difference is that the document must contain both the id and the rev values, and the rev value needs to point to the latest revision. So if you get a document first, make your changes, and then put it up on the server again, things should go nicely. If someone else has stored an update between your get and put request, you will get an conflict error that you need to resolve. How you resolve the conflict is up to the you. You could try to merge your changes with the new revision in the database, or you could simply update your rev value with the latest from the databse and do a new put, effectively overwriting what changes that may have been done.</p>

<pre><code>PUT http://127.0.0.1:5984/geocaches/GC1TCJR

{&quot;_id&quot;:&quot;GC1TCJR&quot;,&quot;_rev&quot;:&quot;1-f70033bafd9d72e174cecb67f58b2e4f&quot;,&quot;url&quot;:&quot;http://www.geoinfo.com/……
</code></pre>

<p>CouchDB will respond with a 200 OK status, and inform you about the new revision number.</p>

<pre><code>{&quot;ok&quot;:true,&quot;id&quot;:&quot;GC1TCJR&quot;,&quot;rev&quot;:&quot;2-6aa44736b02f96844fdfbf4eece00b83&quot;}
</code></pre>

<p>Delete</p>

<p>Can you guess how to delete a document? Close. You make a DELETE request, but you have to include the latest revision in the url as well.</p>

<pre><code>DELETE http://127.0.0.1:5984/geocaches/GC1TCJR?rev=1-f70033….
</code></pre>

<p>As for updating, CouchDB keep your old document around for a while. It will add a new revision, and mark it as a stub for a deleted document.</p>

<p>The response will tell you the new revision number.</p>

<pre><code>{&quot;ok&quot;:true,&quot;rev&quot;:&quot;3-6b8ecfd8bd5d1cb8ae3fd49aea9dfe8d&quot;}
</code></pre>

<p>If you try to do a get on the deleted document, you will get a 404 Object not found response with additional information stating that the document was deleted.</p>

<pre><code>{&quot;error&quot;:&quot;not_found&quot;,&quot;reason&quot;:&quot;deleted&quot;}
</code></pre>

<p>Listing</p>

<p>Now that you know the basic CRUD operations, it would be nice to get a list of all our documents. CouchDB provides a special url we can use for this.</p>

<pre><code>GET http://127.0.0.1:5984/geocaches/_all_docs
</code></pre>

<p>The response will contain information of total number of rows, and an array of ids and revision numbers.</p>

<pre><code>{&quot;total_rows&quot;:4,&quot;offset&quot;:0,&quot;rows&quot;:[

{&quot;id&quot;:&quot;GC13H9E&quot;,&quot;key&quot;:&quot;GC13H9E&quot;, &quot;value&quot;:{&quot;rev&quot;:&quot;1-8e3c239c5da38d0fbe91ffdd24582b02&quot;}},

{&quot;id&quot;:&quot;GC144MQ&quot;,&quot;key&quot;:&quot;GC144MQ&quot;, &quot;value&quot;:{&quot;rev&quot;:&quot;1-865f40ed3047aefdf8c45520c84c4321&quot;}},

{&quot;id&quot;:&quot;GC145KB&quot;,&quot;key&quot;:&quot;GC145KB&quot;, &quot;value&quot;:{&quot;rev&quot;:&quot;1-83ba674d10ce404be7fc90460b1a9a13&quot;}},

{&quot;id&quot;:&quot;GCYXEG&quot;,&quot;key&quot;:&quot;GCYXEG&quot;, &quot;value&quot;:{&quot;rev&quot;:&quot;1-880c0d1767c8f6db0ce815fe0b35afdb&quot;}}

]}
</code></pre>

<p>The _all_docs uri can take some parameters that makes it possible to paginate the result. You can use a combination of &#8216;startkey&#8217;, &#8216;endkey&#8217;, &#8216;limit&#8217; and &#8216;skip&#8217;. Here is a couple of examples that you can try:</p>

<pre><code>GET http://127.0.0.1:5984/geocaches/_all_docs?limit=2

=&gt;First two documents

GET http://127.0.0.1:5984/geocaches/_all_docs?limit=2&amp;skip=2

=&gt;Third and fourth document

GET http://127.0.0.1:5984/geocaches/_all_docs?startkey=GC144MQ&amp;limit=2&amp;skip=1

=&gt;Two next documents after GC144MQ
</code></pre>

<p>What&#8217;s next?</p>

<p>Well, then. You should now have enough information to start making basic use of CouchDB. There are still a lot to cover, though. Did you know that you can upload binary attachments to documents? You can. What you can&#8217;t, though, is to query your data using SQL. Instead you will be making views, but that has to be waiting for now.</p>