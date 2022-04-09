# REST Best Practices

## Post Operation
If your API uses `POST` to create a resource, be sure to include a Location header in the response that includes the URL of the newly-created resource, along with a `201 status code`—that is part of the HTTP standard
## Two update same resource simultaneously
If you need to check that two people don’t try to update the same web resource simultaneously, use the ETag and If-Match headers— that again is the HTTP standard
## Response in desired format
If your API allows users to request data in different formats, use the `HTTP Accept` header.
## Use the fewest number of concepts
Perfection is achieved not when there is nothing more to add, but when there is nothing more to take away
## In URLs, nouns are good; verbs are bad
A consequence of REST’s data-oriented model is that every URL identifies a _thing_. This means that—for well-known URLs and query URLs that are designed to be read and written by programmers of API clients—the URLs should be formed from nouns, which are the natural language part of speech used to designate things.
## Permalinks (Permanent Links)
Any link that a server puts in the representation of a resource can be bookmarked, stored by the client, and used later. This means that you need to try to make it stable.

URLs used in linking tend to be rather unfriendly to humans. Because they must avoid including information that may change, they tend instead to have long strings of random characters in them, like this one from Google:

> https://docs.google.com/document/d/1YADQAXN2sqyC2OAkblp75GUccnAScVSVI7GZk5M-TRo

From the discussion above, you might think that the ideal format for a URL used in linking would be this:

> https://dogtracker.com/{type}/{uuid}

Permalink URLs are designed entirely for the benefit of the server that issues them, not the client that uses them.

## The web is flat
Hierarchies are everywhere, and hierarchies are an important part of how we think about the world. For example, employees are in departments, which are in business units which are in companies. It is thus very tempting to encode those hierarchies in URLs. 

For query URLs, this can be a good idea because queries that represent navigation paths through hierarchies (and other relationships) can be very useful for locating things. 

However, for link URLs that are stored by servers and can be bookmarked by clients, encoding a hierarchy in a URL is a bad idea. Hierarchies are not as stable as they might seem, and encoding them in your URLs will either prevent you from reorganizing your hierarchies or cause your persistent links to break when you do.

## Solutions to the renaming dilemma
If you put the human-friendly identifier in the URL (or otherwise use the human-friendly identifier to persistently reference the resource), then you get human-friendly URLs, but you can never rename the resource because that would break all references. 
If you put the machine-generated identifier in the URL (or otherwise use the machine-generated identifier to persistently reference the resource), then you can freely rename the resource, but you get URLs that are not friendly to humans.

A solution to this problem is to use one URL in links and offer a different one in URI templates for queries.

As a result of this, we can address Joe with the following URL:

    https://dogtracker.com/person/JoeMCarraclough

This URL is very useful for finding Joe, and it is an attractive URL for inclusion in our API. However, we don’t want to use this URL for linking because, while names can be made unique, we need to allow them to change, which is not desirable for persistent links. If we wanted to express a persistent link to Joe, it might look like this:

    { 
	    “id”: “12345678”,
	    “kind”: “Dog”
	    “name”: “Lassie”,
	    “furColor”: “brown”,
	    “Owner”: “https://dogtracker.com/person/e9cdcf7a-25b3-11e5-34363bd0ac10”
	}

This isn’t a beautiful URL, but that doesn’t matter too much because being able to parse URLs in links is not important for users. The URL whose format is more visible to the API user is the query URL derived from the URI template.

## Stability of types
In the last example, we used 

    https://dogtracker.com/person/e9cdcf7a-25b3-11e5- 34363bd0ac10 

as the stable, permanent URL used for linking for Joe. This is based on the assumption that being of type person is a stable property. 

In this example, this is probably OK, but you do need to be thoughtful about types because some types are more stable than others. A person does not change into a dog or a mouse (except perhaps figuratively), but hunters turn into prey, politicians turn into lobbyists, clients are also servers and so on (note that the last example shows that a single resource can have more than one type). 

The only motivation for having person appear in a permalink URL at all is for the server to be able to route incoming requests to the right part of the implementation.

## Designing Query URLs
Query URLs are to HTTP what queries are to a database management system.

We described earlier the popular pattern of defining query URLs that looks like

    https://dogtracker.com/persons/{personId}/dogs
rather than the following equivalent

    https://dogtracker.com/search?type=Dog&owner={personId}

This style of query URL can easily express graph traversal queries that are difficult to express in the query parameter style.

An example that illustrates the last point is this:

    https://dogtracker.com/dogs/123456/owner/spouse

The resource identified by this query URL is a resource whose canonical URL might be 

    https:// dogtracker.com/ person/98765

The query URL encodes a graph query, because it requires first locating the dog whose ID is 123456, and then navigating the owner relationship (a graph edge) to get to the owner, followed by the spouse relationship. These sorts of queries are very natural to express in a URL as a sequence of path segments, but much more difficult to express in query parameters.

##  Filtering collections
It has been traditional to put more complex query clauses that filter collections after the ? in the query string portion of the URL. For example, to get all red dogs running in the park looks like this:

    /dogs?color=red&state=running&location=park
The use of query URLs based on paths and query string parameters can be combined, as shown in this example:

    /persons/5678/dogs?color=red

## What about responses that don’t involve persistent resources?
API calls that return a response that is not the representation of a persistent resource are common.

Words like the following are your clue that you might not be dealing with a persistent resource response:
 - Calculate 
 - Translate 
 - Convert

For example, say you want to make a simple algorithmic calculation like how much tax someone should pay, or do a natural language translation (one language in the request; another in the response), or convert one currency to another. None involve persistent resources returned from a database.

In these cases, it is not necessary to depart from our noun-based, document-based model. The key insight is that the URL should identify the resource in the response, not the processing _algorithm_ that calculates the response.

Since the resource whose representation is in the response is a thing, it is easy to invent a URL for it that fits the noun-based entity model of the web. From the client’s perspective, it is not relevant whether the result is calculated or retrieved from a database, and indeed it is common for calculated resources to be subsequently stored in a persistent cache for performance.

For example, an API to convert 100 euros to Chinese Yuan:

    /monetary-amount?quantity=100&unit=EUR&in=CNY
or simply

    /monetary-amount/100/EUR/CNY

The resource in the response is an amount of money expressed in a currency, which is a thing you can identify with a noun or noun phrase. You could also implement this example using content negotiation:

Request:

    GET /monetary-amount/100/EUR HTTP/1.1
    Host: currency-converter.com
    Accept-Currency:CNY
Response:

    HTTP/1.1 200 OK
    Content-Length: 9
    683.92CNY

A third solution for this sort of situation is to POST to some noun-based URL. For example:

Request:

    POST /currency-converter HTTP/1.1
    Host: currency-converter.com
    Content-Length: 69
    {
    	:amount”: 100,
    	“inputCurrency”: “EUR”,
    	“outputCurrency”: “CNY”
    }

Response:

    HTTP/1.1 200 OK
    Content-Length: 5
    
    683.92

This approach feels very natural to many people, and if this is the best solution for your situation, you can certainly use it. However, it has two downsides you should think about:

 - The response is not cacheable using standard HTTP mechanisms. You can
   make up your own system for caching the response in your application,
   but to do that you will have to assign a key—an identity—to the
   response.
 - When you use the POST pattern discussed here, you are beginning to
   stray from the uniform interface model, because each entity you POST
   to typically has its own unique inputs and outputs that are not
   deducible from a broader data model that underpins a whole set of
   operations. This can quickly begin to look more like the wider, more
   complex interfaces you see in RPC as described in the section Why is
   a data-oriented approach useful?

You should feel free to use POST in this way if it is the best solution to your problem, but you should not do so if there is a straightforward way to use GET instead, especially if the GET can be designed to address a resource from a data model that underpins other operations of the API.

##  Filtering collections
It has been traditional to put more complex query clauses that filter collections after the ? in the query string portion of the URL. For example, to get all red dogs running in the park looks like this:

    /dogs?color=red&state=running&location=park
The use of query URLs based on paths and query string parameters can be combined, as shown in this example:

    /persons/5678/dogs?color=red

## What about responses that don’t involve persistent resources?
API calls that return a response that is not the representation of a persistent resource are common.

Words like the following are your clue that you might not be dealing with a persistent resource response:
 - Calculate 
 - Translate 
 - Convert

For example, say you want to make a simple algorithmic calculation like how much tax someone should pay, or do a natural language translation (one language in the request; another in the response), or convert one currency to another. None involve persistent resources returned from a database.

In these cases, it is not necessary to depart from our noun-based, document-based model. The key insight is that the URL should identify the resource in the response, not the processing _algorithm_ that calculates the response.

Since the resource whose representation is in the response is a thing, it is easy to invent a URL for it that fits the noun-based entity model of the web. From the client’s perspective, it is not relevant whether the result is calculated or retrieved from a database, and indeed it is common for calculated resources to be subsequently stored in a persistent cache for performance.

For example, an API to convert 100 euros to Chinese Yuan:

    /monetary-amount?quantity=100&unit=EUR&in=CNY
or simply

    /monetary-amount/100/EUR/CNY

The resource in the response is an amount of money expressed in a currency, which is a thing you can identify with a noun or noun phrase. You could also implement this example using content negotiation:

Request:

    GET /monetary-amount/100/EUR HTTP/1.1
    Host: currency-converter.com
    Accept-Currency:CNY
Response:

    HTTP/1.1 200 OK
    Content-Length: 9
    683.92CNY

A third solution for this sort of situation is to POST to some noun-based URL. For example:

Request:

    POST /currency-converter HTTP/1.1
    Host: currency-converter.com
    Content-Length: 69
    {
    	:amount”: 100,
    	“inputCurrency”: “EUR”,
    	“outputCurrency”: “CNY”
    }

Response:

    HTTP/1.1 200 OK
    Content-Length: 5
    
    683.92

This approach feels very natural to many people, and if this is the best solution for your situation, you can certainly use it. However, it has two downsides you should think about:

 - The response is not cacheable using standard HTTP mechanisms. You can
   make up your own system for caching the response in your application,
   but to do that you will have to assign a key—an identity—to the
   response.
 - When you use the POST pattern discussed here, you are beginning to
   stray from the uniform interface model, because each entity you POST
   to typically has its own unique inputs and outputs that are not
   deducible from a broader data model that underpins a whole set of
   operations. This can quickly begin to look more like the wider, more
   complex interfaces you see in RPC as described in the section Why is
   a data-oriented approach useful?

You should feel free to use POST in this way if it is the best solution to your problem, but you should not do so if there is a straightforward way to use GET instead, especially if the GET can be designed to address a resource from a data model that underpins other operations of the API.

##  Include self-reference and kind properties
The Google and GitHub examples illustrate two additional design choices that we like. The first is including a `kind` property (sometimes spelled type, or isA). The second is including a `selfLink` property, which is called `url` in the GitHub example. 

Many different spellings of this property are in use— we like the name `self` which is the relationship name standardized in the ATOM specification, registered in IANA, and copied by many others.

Notice particularly that these properties apply to every JSON object, not just the top-level one.

## Why are the self and the kind properties good ideas?
Including a self property makes it explicit what web resource’s properties we are talking about without requiring contextual knowledge—like what URL did you perform a GET on to retrieve this representation—or application-specific knowledge. This is especially important for nested JSON objects where the outer context doesn’t help establish what resource we’re talking about, but it is good practice everywhere.

Including a kind property helps clients recognize whether or not this is an object they know how to process. This helps you add new concepts to your API without having to change the API version.

## How should I represent collections?
Collections are resources too, so the recommendations we have established thus far for all resources also apply to collections.

For example, use the simplest form of JSON, and give them a self property and a kind property. Here is an example of the collection of all dogs:

    {
    	“self”: “https://dogtracker.com/dogs”,
    	“kind”: “Collection”,
    	“contents”: [
    		{
    			“self”: “https://dogtracker.com/dogs/12344”,
    			“kind”: “Dog”,
    			“name”: “Fido”,
    			“furColor”: “white”
    		},
    		{
    			“self”: “https://dogtracker.com/dogs/12345”,
    			“kind”: “Dog”,
    			“name”: “Rover”,
    			“furColor”: “brown”
    		}
    	]
    }

An even simpler option would have been to omit the outer JSON object that defines the collection as a resource, like this:

    [
      		{
      			“self”: “https://dogtracker.com/dogs/12344”,
      			“kind”: “Dog”,
      			“name”: “Fido”,
      			“furColor”: “white”
      		},
      		{
      			“self”: “https://dogtracker.com/dogs/12345”,
      			“kind”: “Dog”,
      			“name”: “Rover”,
      			“furColor”: “brown”
      		}
    ]

While we’re all for simplicity, we think the extra structure in the first version is worth it.

Others have proposed specialized media types for JSON collections, like `Collection+JSON`. For our own APIs, we think this media type and its protocols introduce too much complexity for the benefit they bring. If you do believe that this media type will work well for your clients and your scenarios, we see no fundamental objection.

## Paginated collections
Here is an HTTP message exchange that illustrates one way we have handled pagination in the past:

Request:

    GET /dogs HTTP/1.1
    Host: dogtracker.com
    Accept: application/json

Response:

    HTTP/1.1 303 See Other
    Location: https://dogtracker.com/dogs?limit=25,offset=0

Request:

    GET /dogs?limit=25,offset=0 HTTP/1.1
    Host: dogtracker.com
    Accept: application/json

Response:

    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Location: https://dogtracker.com/dogs?limit=25,offset=0
    Content-Length: 23456
    {
    	“self”: “https://dogtracker.com/dogs?limit=25,offset=0”,
    	“kind”: “Page”,
    	“pageOf”: “https://dogtracker.com/dogs”,
    	“next”: “https://dogtracker.com/dogs?limit=25,offset=25”,
    	“contents”: [
    		{
    			“self”: “https://dogtracker.com/dogs/12344”,
    			“kind”: “Dog”,
    			“name”: “Fido”,
    			“furColor”: “white”
    		},
    		{
    			“self”: “https://dogtracker.com/dogs/12345”,
    			“kind”: “Dog”,
    			“name”: “Rover”,
    			“furColor”: “brown”
    		},
    		… (23 more)
    	]
    }

As you can see, the server recognized that returning the whole collection wasn’t going to work, and redirected the client to a different resource that represents the first page of the collection. The client followed the redirect, which returned the first page.

The first page contains a subset of the collection contents, references the original collection in the pageOf property, and also references the subsequent page in the next property (unless there is no next page). There is also a previous property that is missing in this case because there is no page before the first one.

There is no standard for the representation of pages of collections shown here—we offer it in the spirit of a minimalist approach that uses JSON in a simple, direct way. There are many other approaches out there.

## Supporting multiple formats
If you support multiple output formats, you should do so by supporting the standard `HTTP Accept` header.
## Javascript Object Notation PATCH

    https://www.rfc-editor.org/rfc/pdfrfc/rfc6902.txt.pdf

## PATCH OR PUT
The reason you should use PATCH for update is that it helps with the evolution of your API. PUT requires the client to replace the entire content of the resource with every update. As your API evolves, you want to be able to add new properties without breaking existing clients. If clients replace the entire content on every update, you are relying on them to faithfully preserve all properties, even if they weren’t changed and even if they didn’t exist at the time the client was written.

## What about property names?
You have an object with data attributes on it. How should you name the attributes?

Here are API responses from a few leading APIs:

Twitter:

    “created_at”: “Thu Nov 03 05:19;38 +0000 2011”
Bing:

    “DateTime”: “2011-10-29T09:35:00Z”
Foursquare:

    “createdAt”: 1475795458

## Date and time formats
The format we see used most often is the one standardized by XML Schema, which is a subset of the complex ISO 8601 standard. Bing seems to be using this. 

Unix had a wonderfully simple and elegant time format that was based on the number of milliseconds elapsed since the beginning of the epoch—defined to be the beginning of 1970, GMT. Foursquare seems to be using this format. 

This terrifically simple representation—a monotonically-increasing integer that was independent of location and could be differenced to calculate time intervals—made an ideal time representation until it was undermined by a very poor design decision that was made when implementing leap seconds in Unix.

The sad story is told here and elsewhere. Even though it is now compromised, this format deserves consideration as a date and time representation.

## Partial response
Partial response allows you to give application developers just the information they need.

Let’s look at how several leading APIs handle giving application developers just what they need in responses.

LinkedIn:

    /people:(id,first-name,last-name,industry)

Facebook:

    /joe.smith/friends?fields=id,name,picture

Google:

    ?fields=title,media:group(media:thumbnail)

Google and Facebook have a similar approach, which works well.

They each have an optional parameter called fields after which you put the names of fields you want to be returned.

As you see in this example, you can also put subobjects in responses to pull in other information from additional resources.

## Make it easy for application developers to paginate objects in a database

In a previous section, we showed how you can use `next`, `previous`, `first`, and `last` links to allow API clients to scroll through large lists very simply without the need to construct URLs from complex URI templates. 
This is often sufficient for handling collections, but occasionally you may want to allow clients more explicit control over pagination.

Let’s look at how Facebook, Twitter, and LinkedIn handle pagination. Facebook uses `offset` and `limit`. Twitter uses `page` and `rpp` (records per page). LinkedIn uses `start` and `count`.

Semantically, Facebook and LinkedIn do the same thing. That is, the LinkedIn `start` and `count` is used in the same way as the Facebook `offset` and `limit`.

To get records 50 through 75 from each system, you would use:

 - Facebook - offset 50 and limit 25 
 - Twitter - page 3 and rpp 25 (records per page) 
 - LinkedIn - start 50 and count 25

We like `limit` and `offset`, although the data values that the implementation has to sort on, and the database technologies being used may dictate compromises.

## Response to a successful GET

    HTTP/1.1 200 OK
    Content-Location: https://dogtracker.com/dogs/1234567
    Content-Type: application/json
    ETag: 1437080173827
    Content-Length: nnnn
    
    // body goes here //

## Response to a method that the server does not support
The 405 error code indicating that the client tried to use a method that the server does not support for the given resource must be paired with an Allow header saying which methods are supported for the resource.

    HTTP/1.1 405 Method Not Allowed
    
    Allow: GET, DELETE, PATCH
