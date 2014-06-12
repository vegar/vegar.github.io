---
title: Introducing CouchDB
author: Vegar
layout: post
permalink: /2010/introducing-couchdb/
categories:
  - Development
tags:
  - CouchDB
---
<p><img src="http://blog.vi-kan.net/wp-content/uploads/2010/05/image2.png" alt="image" title="image" /> A while back, I became aware of something called CouchDB. Since then, I have spend a little time now and then trying to understand what it is, and if I should care. I still struggle a bit with the whole idea of &#8216;document datebases&#8217; and when it is more appropriate then relational databases, but want let a lack of understanding stop me from getting my hands wet.</p>

<h2 id="thebasics">The basics</h2>

<p>So here is the most basic ideas behind CouchDB and how it works.</p>

<h3 id="documents">Documents</h3>

<p>While a relational db stores rows in tables, CouchDB stores jason-formatted documents. While a table in a relational db has a defined set of columns that can have values, there is no definition on what values a document in CouchDB should contain. So when a table enforce every row to be in the same format, each and every document in a CouchDB database could contain completly different kind of values.</p>

<p>The book <a href="http://my.safaribooksonline.com/9780596158156">CouchDB: The Definitive Guide</a> uses business cards as an illustration on this. In a traditional database, you would have a table where each row represents one business card. You then have to choose what columns you would have. You would need a name, a telephone number, a url to a website and an email address. After a while, you realise that you would need a column for a company name and a new column for a second telephonenumber for the company. In the third iteration, you realise that you sill need  more phonenumbers, and maybe even a fax. Now, should we make more telephone columns, or should we have a one-to-many relationship from a business card to a list of phonenumbers? And what about all our trendy business contacts with only a single url on their card? There would be a whole lot of emtpy columns then.<img src="http://blog.vi-kan.net/wp-content/uploads/2010/05/image_thumb.png" alt="Table of business cards" title="Table of business cards" /></p>

<p>In CouchDB, you would have a business card database where each card would be represented by one document. The content of the document would be defined by the data on that given businesscard.<img src="http://blog.vi-kan.net/wp-content/uploads/2010/05/image1.png" alt="Business cards as documents" title="Business cards as documents" /></p>

<p>As you can see, this makes everything very flexible. No need for any migration of existing documents when you suddanly find your first business card with a twitter name on it.</p>

<h3 id="httpandrest">HTTP and REST</h3>

<p>CouchDB is basically a small webserver listening to a given port. When communicating with the server, you would make HTTP requests specifying what you want, and CouchDB would make an apropriate respons. Instead of inventing some new rpc format, it uses the basic http request methods for each and every method. So if you want to get a document, you send a GET request. Do create or update a document, you use PUT or POST, and to remove a document, you use DELETE.</p>

<p>A service designed in this way, is often refered to as RESTful.</p>

<h3 id="json">JSON</h3>

<p>Every response comming from CouchDB, and every document stored inside CouchDB, will be in the form of JSON. JSON, or <a href="http://www.json.org/">JavaScript Object Notation</a>,  is a standard made for representing objects in a way that should be easy both for human and machine to write and consume.</p>

<p>A JSON object consists of a set of name-value pairs, where the value can be either an array of values, a JSON object, a string, a number, true, fase or null. Even though the synax is simple, it is possible to represent list of complex, nested objects.</p>

<p>A JSON representation of one of our business cards could look something like this:</p>

<pre><code>{
   &quot;name&quot;: &quot;Joseph A. Gutierrez&quot;,
   &quot;website&quot;: &quot;http://wallpaperdealer.com/&quot;,
   &quot;phone&quot;: [
       &quot;41-397-0567&quot;,
       &quot;954-399-5693&quot;
   ],
   &quot;fax&quot;: &quot;518-923-6525&quot;
}
</code></pre>

<h2 id="installation">Installation</h2>

<p>Like many other open source applications, CouchDB is distributed as source code, some dependencies to other open source technologies, and a description on how to build for you spesific platform. This makes it easy to port everything to different platforms and operating systems, but a little more complex for windows users like me that are used to get everything wrapped up nice in a install executable.</p>

<p>Luckly, there are people out there that provieds us with binaries, and a binary for CouchDB for windows is made available through the <a href="http://wiki.apache.org/couchdb/Windows_binary_installer">CouchDB Wiki</a>.</p>

<p>The installer will suggest installing CouchDB as an service, which makes it easy enough for now.</p>

<h3 id="configuration">Configuration</h3>

<p>CouchDB stores configuration data in two ini files. You can find them where you installed CouchDB in the subfolder \etc</p>