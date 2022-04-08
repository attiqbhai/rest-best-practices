# REST Best Practices

## Post Operation
If your API uses `POST` to create a resource, be sure to include a Location header in the response that includes the URL of the newly-created resource, along with a `201 status code`—that is part of the HTTP standard
## Two update same resource simultaneously
If you need to check that two people don’t try to update the same web resource simultaneously, use the ETag and If-Match headers— that again is the HTTP standard
## Response in desired format
If your API allows users to request data in different formats, use the `HTTP Accept` header.

