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
