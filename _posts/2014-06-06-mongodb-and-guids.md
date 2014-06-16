---
layout: post
title: MongoDB and Guids
---

> TLDR: use `MongoClientSettings.GuidRepresentation` to control how guids are encoded through the .net mongo driver. 

When first starting with MongoDb, we generated our document Ids our self in code with [`Guid.NewGuid()`](http://msdn.microsoft.com/en-us/library/system.guid.newguid.aspx). What we then experienced, was that the actually stored guid, the guid displayed to us by the mongo shell, was not the same as the guid that we saw in Visual Studio during debugging.

At first we where afraid that there was something completly wrong, but testing showed that everything worked as expected - except for the difference in the displayed values.

Today I had the oppertunity to ask [Craig Wilson](http://github.com/craiggwilson), the maintianer of the [.net mongo driver](http://docs.mongodb.org/ecosystem/drivers/csharp), about this issue. What he told was that each mongo driver has its own way of encoding guids when sending it to mongodb. I'm not sure how big an issue this really is. As long as we stick to one language, it shouldn't be any problem at all, but could mixing two languages with different encoding schemas create trouble? I don't know.

What I do know, though, is that the the .net driver includes a setting to smooth this out. When crating the `MongoClient`-object, you pass a `MongoClientSettings`-object that again has a [`GuidRepresentation`](http://api.mongodb.org/csharp/current/?topic=html/d73d605b-11a8-222c-6937-347ffdab2c63.htm)-property. 

With that problem sorted out, there are something else that should be considered. Rob Conery have written a piece on [id generation](http://www.wekeroad.com/2014/05/29/a-better-id-generator-for-postgresql/). Basically its much better to have sequential keys than random keys. There are ways of generating sequential guids, but [this project](https://github.com/Restuta/mongo.Guid-vs-ObjectId-performance) shows that using the native `ObjectId` may be a better solution anyway. 