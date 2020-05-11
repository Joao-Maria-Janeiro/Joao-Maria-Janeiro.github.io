---
layout: post
title: "Handling Anonymous Users"
author: "João Maria Janeiro"
categories: sample
tags: [Web Dev]
image: extraction.png
---

Anonymous users is a big part of web development and e-commerce, more specifically. What makes it so important? It's the first experience users have on your website! 

## The issue
The issue with anonymous users (the one we will focus on in this post, of course there are others) is how to store their information. Let's say your anonymous user adds some product to it's cart, where do you save it? How can you access it in the future?

## Possible solutions
If you are developing a ***website made up of a front end*** only in the browser (no mobile app, only browser) ***plus a back end*** somewhere else you have quite a few options. You can save the data in the client side and use [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage), [cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies), [Web SQL](https://www.tutorialspoint.com/html5/html5_web_sql.htm), [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [Cache Storage](https://developer.mozilla.org/en-US/docs/Web/API/Cache) or you can save the data in the server side treating an anonymous user as a regular user or as a special data type. Let's go through our options. 

### Client-side storage

#### Cookies
Cookies are the oldest of the bunch, and are not very useful these days so you really shouldn't use them unless you need to support some legacy browsers for your project. Cookies have security issues and do not handle complex data very well. Cookies go with every request you make, so they can seriously degrade your website's performance and increase the size of every request. You might wonder why you still see a lot of websites using cookies, this is mostly because of the amount of resources still available on cookies and that some projects were developed originally with cookies and didn't migrate to the new storage options.

#### Local Storage
This is one of the most useful and the one I use the most. This is made of pairs of keys and values, both strings, it was developed to store simple information like a user id, a user token or user preference like background color or some other info. The data stored is saved across browser sessions, so the data persists. It's relevant to mention that Safari in private mode gives an empty localStorage with a quota of zero so it's unusable and you might also get a QuotaExceededError which means that we've used up all available storage space (but this is not just for localStorage, it's for all storage options in the browser). There's a separate data storage for each domanin so apple.com can't see google.com's information, you can imagine the security issues if it could! The great advantage with local storage is its ease of use, it's really simple to add, edit and delete data from local storage. The performance is good but since it's not indexed and it's a synchronous API (blocking requests) you don't get the performance you get with localStorage but you get a lot less code to do the same (possibly) as with indexedDb which of course leads to less errors and bugs!


You can add an item to localStorage like this:
```javascript
// You can just assign it like this
window.localStorage['user_id'] = 2;
// Or use the provided method
window.localStorage.setItem('user_id',2);
// Or even
window.localStorage.user_id = 2;
```

Retrieve items:
```javascript
// You can just assign it like this
var user_id = window.localStorage['user_id'];
// Or use the provided method
var user_id = window.localStorage.getItem('user_id');
```

Check if a key is present:
```javascript
window.localStorage.containsKey('user_id')
```

Remove items:
```javascript
window.localStorage.removeItem('user_id');
```
If you choose to store data in the client you will most likely use local storage to store some data! 

#### Session Storage
Session storage is the same as local storage but local storage persists the data after you close the browser and session storage does not.

#### Web SQL
"The W3C Web Applications Working Group ceased working on the specification in November 2010, citing a lack of independent implementations (i.e. the use of a database system other than SQLite as the backend) as the reason the specification could not move forward to become a W3C Recommendation.

Mozilla Corporation were one of the major voices behind the break-up of negotiations and deprecation of the standard, while at the same time being the main proponents behind an 'alternative storage' standard, IndexedDB" [Web SQL Database. In Wikipedia. Retrieved from [url](https://en.wikipedia.org/wiki/Web_SQL_Database)]

#### IndexedDB
IndexedDB is a low-level API of client side storage that is able to store complex and structured data types, including files and blobs (which means you can store data such as images, audio, video...). This API uses indexes so searching is very fast. While Local and Session storage are very useful for storing smaller and less complex data, this API excels at storing large amounts of structured, related and complex data! IndexedDB is a transactional database like a SQL based system. However it doesn't use fixed-column tables, it's instead an object oriented databse in JavaScript. You can store and retrieve objects that are indexed with a key, it can be any object suported by the [structered clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm). In terms of performance since it's indexed it's really fast but even better than that, this is an asynchronous API so as not to block operations even if they are really big! There also limits in browser space as well, the maximum size is dynamic - based on your hard drive size. MDN says the global limit is calculated as 50% of free disk space, check full info [here](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria). Also Firefox in private mode doesn't support IndexedDB. If you want to develop an offline website this makes it really easy! In order to do it offline you should take advantage of [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API). Much like Web storage (local and session) IndexedDB is also unique for each origin/domain.

Check [this example](https://developers.google.com/web/ilt/pwa/working-with-indexeddb) to see how to use the indexedDB.


#### Cache API
This one is fairly recent! The goal of this API is to store HTTP [responses](https://developer.mozilla.org/en-US/docs/Web/API/Response) to requests, this is very useful because you can use the website offline as the responses are saved on disk! "Cache is usually used in combination with the Service Worker API, although it doesn't have to be" said MDN. This is also an asynchronous system, and will not block the main thread. This is usually used for network resources to load the app. This is a really good API that you should totally check out but it's not of major relevance here.


#### Let's compare
Since our objective is to store data, the comparison really comes down to Web Storage (Local and Session) versus IndexedDB. Maybe you got the idea that IndexedDB is the ideal way to go every time and... If you do that it's ok but you might be walking into unnecessary bugs and development time. Web Storage is a much simpler API, you can just pick it up and use it without any database knowledge, IndexedDB is better to have some database background knowledge and is quite complex, you'll need to write more code to get the same thing (the more code you have, the more error prone you are!). Like I said before Local Storage only stores strings so to put an object in you'll usually Json serialize it (JSON.stringify). 
You can store an object like:

```javascript
window.localStorage.setItem('user_profile', JSON.stringify({'First Name': 'John', 'Last Name': 'Doe', 'Address': 'Random Street Address'}));
```

Finding the object with the key 'user_profile' is really easy but in order to get the object back you have to parse it (JSON.parse):

```javascript
JSON.parse(localStorage.getItem('user_profile'));
```

To store an image you'll have to base64 encode and decode as well, which is a heavy process if you are doing it with a lot of images. 

Using Web Storage is fine if you are justing storing some information to then upload to the server or you just store some information you got from the server, if you don't change or update it a lot, you just store it. Let's say you have a user profile you got from the server, this is just one entity and you are probably not going to update those values, if you are, you are probably updating it in the server first and just getting it back from the server again! Let's say you have the user cart, the user won't add multiple items at once and probably won't add many items, so if your cart is simple you could opt to store it this way, if it's a very complex model this is probably not the best way, you should opt for IndexedDB. If you only have a couple of objects in local storage, converting to and from JSON is okay but if you have thousands of objects, all of them with the property first name or like a cart with products all with a property price and you want to filter it by price above a certain threshold, with local storage you'll have to loop through all the objects parse and check the price on each one, this is a lot of processing time! Also IndexedDB knows about ranges, so you can use that as well! With IndexedDB, since you have indexes, you could just add each product object directly and create an index for the price and you can use that to retrieve, without having to scan every object and since IndexedDB is asynchronous it won't block the other processes while searching or doing other operations. Web Storage does block but unless you are working with lots and lots of data you won't notice it because in-memory operations are quite fast! 

So the take away really is, it depends on the data you have! If you are storing data that's not very complex and you don't have a lot of it, Web Storage is a great way to go since it's very easy to use (you use a framework like Angular Dart as I do, which is not JS based, having less complexity in the JS is a BIG PLUS!) and you'll probably face 0 errors using it! If you are storing files, videos or other types, or objects that have many fields that you update a lot, or big arrays of objects taht you do operations on, you should probably go with IndexedDB! 

Having that said they are not mutually exclusive, you can use both of them in the same project for different types of data! 