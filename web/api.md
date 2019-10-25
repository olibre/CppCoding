* [HTTP](#http)
* [WebSocket](#websocket)


HTTP
====

Different API types:

* Single entrypoint with messages within body request (similar as for WebSocket)
* CRUD = Reuse of HTTP methods (`POST`, `GET`, `PUT`, `DELETE`...)
* REST = CRUD applied to ressources (entrypoint = plural form of a ressource name)
* RESTful = REST with standardized API documentation (OpenAPI = Swagger, reDoc)
* GraphQL = single entrypoint with query language

Features
--------

* Authentication can be provided within the header request
* Pagination (TDOD)

REST URI
--------

URI can be:

* `https://www.example.com/api/v1/books`
* `https://api.example.com/v1/books`


HTTP persistent connection
--------------------------

* HTTP/1.0
    * Header `Connection: keep-alive`
* HTTP/1.1
    *  Chunked transfer encoding https://en.wikipedia.org/wiki/Chunked_transfer_encoding
    *  HTTP pipelining https://en.wikipedia.org/wiki/HTTP_pipelining
* HTTP/2.0
    * Multiplexing https://en.wikipedia.org/wiki/HTTP/2#TCP_head-of-line_blocking




CRUD methods (HTTP requests)
----------------------------

Methode | Ressource   | Parameters            | Meaning           | Response
--------|-------------|-----------------------|-------------------|--------------
`POST`  |`/books`     | JSON in body          | Create a book 	  | `201` (sync) or `202` (asynchrone)
`GET`   |`/books`     | Query string          | Retrieve all books| `200`, `204`, `206`
`GET`   |`/books/{id}`| `{id}` + Query string | Read one book 	  | `200`
`PUT`   |`/books/{id}`| `{id}` + JSON         | Update a book     | `200`
`DELETE`|`/books/{id}`| Path `{id}`           | Delete a book     | `204`
`DELETE`|`/books`     |                       | Delete all books  | `204`

The `PUT` and `DELETE` mathods may allow accept query string parameters (often common with the `GET` method). The `GET /books` may accept optional parameters (query string) to provide a response with pagination, sorting, ordering and filtering. The `DELETE` method may accept extra parameter within the request body (JSON). Request body usually conveys the parameters within the JSON format, but other formats can be used such as XML or even binary formats such as SBE.

Other HTTP methods can also be used such as `PATCH /books/{isbn}`. The `PATHC` method has a subtile difference with the `PUT` method. The `PUT` method means a full replacemet, and `PATCH` is a partial modification. However, to simplify the API, the `PUT` method is often used for a partial update, without the presence of the `PATCH` method.


HTTP response status codes
--------------------------

Inspired from:
* Kay Plößer (Ploesser) has posted in May 2019 the article https://www.moesif.com/blog/technical/api%20design/Which-HTTP-Status-Code-To-Use-For-Every-CRUD-App
* https://gist.github.com/subfuzion/669dfae1d1a27de83e69
* https://metamug.com/article/status-codes-for-rest-api.html
* (CC BY-SA 4.0) https://www.restapitutorial.com/lessons/httpmethods.html#get


### Status Classes

* **`100..199` Information** - The request header has been received and the server will try to comply with a transmission demand of the client like using a different protocol (WebSocket) or telling the client that its request will fail before they start sending the body.
* **`200..299` Success** - The client request was accepted. In case of `202` asynchronous processing of a request, this doesn’t mean the request was successfully processed only that the request met all validation requirements when received by the server.
* **`300..399` Redirection** - The requested resource is not available at the expected location. This can have multiple reasons, temporary or permanent. The client should issue a request to the new location.
* **`400..499` Error in client request** such as timeouts, wrong URI, missing authentication… The client should change the request parameters before retrying the request.
* **`500..599` Error on server side** indicating problems with overwhelmed servers or unreachable servers behind proxies, but sometimes they can be directly related to client requests that trigger error exceptions on the server side. These errors can be temporary or permanent. Usually it’s best for the client to retry the same request.
* **`600..xxx` Custom codes** above 599 aren’t permitted but used by some services anyway. For API designers, they are relevant as bad examples. For API client developpers, however, they have to deal with them.

### Create `POST /books`

To create a new resource or to access a token.

* **`200` OK** - When returning an access token (no ressource endpoint creation).
* **`201` Created** - Synchronous ressource creation. The response should contain a `Location` header for the newly created resource URL. The response body should contain the created resource.
* **`202` Accepted** - Asynchronous processing of a valid request but completion will finish sometime in the future. The response body should include an URL to the finished resource or to some monitoring endpoint that tells the client when the resource is available. The response may also provide an estimation of how long the creation will take.
* **`303` See Other** - Like the `202` code but using a `Location` header. This is nice because some clients automatically follow the `Location`.
* **`409` Conflict** - If the resource to create already exists.

### Read `GET /books/{id}`

To retrieve a resource representation.

* **`200` OK**
* **`404` Not Found** - If `{id}` does not exist.

### Read `GET /books`

To fetch multiple resource representations.

* **`200` OK** - When response contains all requested ressources.
* **`206` Partial Content** - When the response content is too big to be delivered in one response. The client request may contain a [`Range` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range). The server response must provide this `Range` header.
* **`300` Multiple Choices** - This redirect is used if there are multiple representations for one resource and the client has to choose one of them.
* **`308` Permanent Redirect** - This tells the client to use another URL to access the resource and not use the current URL anymore. It’s helpful when we have multiple endpoints for one resource, but do not want to implement reading from all of them.
* **`304` Not Modified** - Like `200` but without a body, so the client will be redirected to its locally cached representation from a previous read.
* **`307` Temporary Redirect** - When the URL to a resource could change in the future, and the client should always ask the current URL before accessing the real one.
* **`416` Range Not Satisfiable** - When the [`Range` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) is ot valid.


### Update `PUT /books/{id}`

To update a ressource.

* **`200` OK** - Synchronous update. The response should contain the updated ressource.
* **`202` Accepted** - Asynchronous update. The response should provide an URL to the resource or to some monitoring endpoint that tells the client the update progress. The response may also provide an estimation of how long the update will take.
* **`204` No Content** - Like `200` but without a respnse body. This is fine, for example, when the client has just saved a currently edited ressource.
* **`404` Not Found** - If `{id}` does not exist.


### Delete `DELETE /books/{id}`

To delete (or disable/cancel) a ressource.

* **`200` OK** - Synchronous deletion. The response should contain the deleted ressource.
* **`202` Accepted** - Asynchronous deletion. The response should include an URL to follow the deletion progress. The response may also provide an estimation of how long the deletion will take.
* **`204` No Content** - Synchronous deletion without response body (as the resource has been deleted).
* **`404` Not Found** - If `{id}` does not exist.

### Delete `DELETE /books`

To delete, disable or cancel all ressources.

* The previous status codes, or
* **`405` Method Not Allowed** - If the deletion of the whole collection is not possible.


API versionning
---------------

The URL contains the majour version number `{v1}` such as:

    https://api.example.com/{v1}/books

To allow a client to test the API in a simulation anvironment, the URL can be:

    https://api-sandbox.example.com/{v1}/books

To allow a client to test a minor API change (e.g. a new feature backward compatible), the URL can be:

    https://api-sandbox-next.example.com/{v1}/books

or:

    https://api-next.example.com/{v1}/books`

Developers may use:

    https://api-sandbox-dev.example.com/{v1}/books

When an API lives long enough, sooner or later, its structure will change. To avoid breaking changes, the new API version should status codes of the redirect class. This will help, especially for the clients automatically following the `Location` header.

* **`307` Temporary Redirect** - This is the right code if the resource could be available on a different URL in the future, but we want the current endpoint to control where the client is redirected to. This status code will let the client come back to the current URL for every request.
* **`308` Permanent Redirect** - This is the right code if the resource will now be available at a new URL and the client should directly access it via the new URL in the future. The current endpoint can’t control the clients behavior after the request and a subsequent redirect if the resource URL changes again have to be issued from the new URL.

Multiple endpoints for one resource
-----------------------------------

The API using nested or sub-resources can only deliver representations from root (non-nested) resources. For example, the two endpoints `/books/{id}/comments/1234` and `/users/{u}/comments/1234` represent the same ressource (a user's comment of a book) that can also be access using `/comments/1234`.

* **`308` Permanent Redirect** - To redirect at the nested resources with the `Location` header pointed to the root resource so not every endpoint needs a delivery implementation. This should only be done for GET requests.


Permission errors
-----------------


* **`401` Unauthorized** - The client has not authorized itself to the backend yet and the server may give it access after that has happened.
* **`403` Forbidden** - The client has authorized or does not need to authorize itself, but still has no permissions to access the resource.
* **`404` Not Found** - If `401` or `403` is the case, but the backend doesn’t want to tell the client that the resource exists for security reasons.



Other errors
------------


* **`404` Not Found** - This is the most natural of responses and should be used in the case that the client URL was wrong.
* **`405` Method Not Allowed** - In many frameworks we define the URL alongside the HTTP method, so removing one of these definitions could leave all but one methods to one URL intact. The response has to include an Allow header field that lists what is allowed to do.
* **`501` Not Implemented** - Like 405, but the method is missing for all resources on the backend.
* **`406` Not Acceptable** - The URL exists, but the backend can’t send a representation the client will accept. So for that specific client, the URL behaves like a 404, but they now know that they need to update the client.
* **`410` Gone** - This is like 404 but we know that the resource existed in the past, but it got deleted or somehow moved, and we don’t know where.
* **`414` Request-URI Too Long** - This is sometimes the case when the endpoint is right, and the resource exists, but the query makes the URL too long to be interpreted.
* **`308` Permanent Redirect** - This would be the right code if we know that a resource has moved to a different URL and we can tell the client about it.
* **`307` Temporary Redirect** - Same as 308, but we don’t know if the resource will be back on the original URL or another different URL in the future.
