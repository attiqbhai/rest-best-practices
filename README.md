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

## Designing query URLs
Query URLs are to HTTP what queries are to a database management system.

We described earlier the popular pattern of defining query URLs that looks like

    https://dogtracker.com/persons/{personId}/dogs
rather than the following equivalent

    https://dogtracker.com/search?type=Dog&owner={personId}

