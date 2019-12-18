---
layout: post
title: "REST good practices"
categories: REST
---

"Representational State Transfer (REST) is a software architectural style that defines a set of constraints to be used for creating Web services." [Wiki](https://en.wikipedia.org/wiki/Representational_state_transfer).

# Disclaimer
REST is not a specification. You can do one thing in many ways and sometimes it is difficult to say which method is more correct.

# General rules
* Treat the resource like a noun, and HTTP method as a verb, e.g.
 ```DELETE /companies/3/employees/45``` - delete employee #45 in company #3

* Use plural resource nouns, e.g. ```/companies```

* If a relation can only exist within another resource, RESTful principles provide relations, e.g.
```/companies/3/employees/45``` - employee #45 in company #3

* Do not use trailing forward slash (/) in URIs, e,g. ```http://api.example.com/companies/```
* Use hyphens ```-``` to improve the readability of URIs:
http://api.example.com/inventory-management/managed-entities/{id}/install-script-location  //More readable

* Do not use underscores ( _ )
http://api.example.com/inventory_management/managed_entities/{id}/install_script_location  //More error prone

* Use standard terminology, e.g. ```bookId```, ```dogId```, but not ```bId```, ```dId```.

* use lowercase letters in URIs. RFC 3986 defines URIs as case-sensitive except for the scheme and host components. e.g.
```
http://api.example.org/my-folder/my-doc
HTTP://API.EXAMPLE.ORG/my-folder/my-doc - the same URIs
```
``` 
http://api.example.org/My-Folder/my-doc - different one
```
 
* Do not use file extensions in URI
```
http://api.example.com/device-management/managed-devices.xml  - This is not correct URI
http://api.example.com/device-management/managed-devices 	  - This is correct URI
```

Use query component to filter URI collection
```
/devices
/devices?region=USA
/devices?region=USA&brand=XYZ
/devices?region=USA&brand=XYZ&sort=installation-date
```

# Querystring & pagination 
* use of the querystring for filtering and pagination
```
/customers?page-number=2&page-size=100&sort=name,-id
```

Use:
```
GET: /articles/?published=true&page=2&page_size=20
NOT
GET: /articles/published/
```

SKIPLIMIT - offset 












* If the actions doesn't fit into the world of CRUD operations, restructure the action to appear like a field of a resource.
  * For example an `activate` action could be mapped to a `boolean activated field` and updated via a PATCH to the resource:
  ```
  PATCH 
  {
    "activated": true
  }
  ```
  
  
  * Threat it like a sub-resource. For example, GitHub's API lets you star a gist with PUT /gists/:id/star and unstar with DELETE /gists/:id/star.
  
  * Sometimes you really have no way to map the action to a sensible RESTful structure. 
  For example, a multi-resource search doesn't really make sense to be applied to a specific resource's endpoint.
  In this case, /search would make the most sense even though it isn't a resource. 
  This is OK - just do what's right from the perspective of the API consumer and make sure it's documented clearly to avoid confusion.







# METOHODS

    GET retrieves a representation of the resource at the specified URI. The body of the response message contains the details of the requested resource.
    POST creates a new resource at the specified URI. The body of the request message provides the details of the new resource. Note that POST can also be used to trigger operations that don't actually create resources.
    PUT either creates or replaces the resource at the specified URI. The body of the request message specifies the resource to be created or updated.
    PATCH performs a partial update of a resource. The request body specifies the set of changes to apply to the resource.
    DELETE removes the resource at the specified URI.




# GET

który zapis ?
GET: /authors/12/articles
GET: /articles/?author_id=12 <- na blogu ten preferowany, bo plaska struktura




# Limiting which fields are returned by the API
Use a fields query parameter that takes a comma separated list of fields to include. 
For example, the following request would retrieve just enough information to display a sorted listing of open tickets:
```
GET /tickets?fields=id,subject,customer_name,updated_at&state=open&sort=-updated-at
```








# Error 
{
    "error": "Invalid payoad.",
    "detail": {
        "surname": "This field is required."
    }
}





Learn the difference between 401 Unauthorized and 403 Forbidden

When handling security errors in an API, it's very easy to be confused about whether the error relates to authentication or authorization (a.k.a. permissions) — happens to me all the time.

Here's my cheat-sheet for knowing what I'm dealing with depending on the situation:

    Has the user not provided authentication credentials? Were they invalid? 👉 401 Unauthorized.
    Was the user correctly authenticated, but they don’t have the required permissions to access the resource? 👉 403 Forbidden.

-----
There are two cases which I find 202 Accepted to be especially suitable for:

    If the resource will be created as a result of future processing — e.g. after a job has finished.
------

Updates & creation should return a resource representation

A PUT, POST or PATCH call may make modifications to fields of the underlying resource that weren't part of the provided parameters (for example: created_at or updated_at timestamps). To prevent an API consumer from having to hit the API again for an updated representation, have the API return the updated (or created) representation as part of the response.

In case of a POST that resulted in a creation, use a HTTP 201 status code and include a Location header that points to the URL of the new resource.

-----
jak zrobic, zeby zwracalo jsona lub xml?
a .json or .xml
-----
czy zwracac Liste elementow czy obiekt z lista?
----
