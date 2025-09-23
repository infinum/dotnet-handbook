# API Standardization

# Introduction

In this chapter, we will discuss the API standardization - why do we have it, what are its benefits, and how do we do it.

When developing an API, we are constantly solving numerous challenges. From picking the right technology, designing the infrastructure to support all our requirements, organizing our solution to enable all developers to work with the least friction possible, developing workflows to support the business requirements, to coming up with entity & property names, and even determining the indent sizes in our projects. A lot of these challenges are not unique to a specific project or a domain, and can reuse the same solutions. We have standardized architectures, design patterns and code standards, but when it comes to designing the interface for communication between systems, we didn't agree on a default way to solve our common issues. To mitigate that, we’re introducing API Standardization.

The goal of API standardization is to remove the need to have the same discussions for each project, and make onboarding easier of new developers, leaving room for focusing on resolving real business challenges. This document describes the key concepts and proposes solutions to common challenges when it comes to API design. All future projects that are in our full control must conform to this standard. Obviously, we can make exceptions if a project has specific requirements that cannot be met using the rules mentioned in this document. In that case, the project **must** contain a document describing those exceptions and use cases.

# Key terms & concepts

## Resources and resource representations

In his dissertation, [Architectural Styles and the Design of Network-based Software Architectures](https://ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf), Roy Fielding talks about his experiences and lessons learned from applying REST while authoring the standards for HTTP and URI. What’s particularly interesting for this topic are the changes affected by applying REST to URI:

> *The early Web architecture defined URI as document identifiers. Authors were instructed to define identifiers in terms of a document’s location on the network. Web protocols could then be used to retrieve that document. However, this definition proved to be unsatisfactory for a number of reasons. First, it suggests that the author is identifying the content transferred, which would imply that the identifier should change whenever the content changes. Second, there exist many addresses that corresponded to a service rather than a document — authors may be intending to direct readers to that service, rather than to any specific result from a prior access of that service. Finally, there exist addresses that do not correspond to a document at some periods of time, such as when the document does not yet exist or when the address is being used solely for naming, rather than locating information.*
> 

This led to the redefinition of the term resource in the scope of HTTP and URI:

> *Defining* resource *such that a URI identifies a concept rather than a document leaves us with another question: how does a user access, manipulate, or transfer a concept such that they can get something useful when a hypertext link is selected? REST answers that question by defining the things that are manipulated to be representations of the identified resource, rather than the resource itself. An origin server maintains a mapping from resource identifiers to the set of representations corresponding to each resource. A resource is therefore manipulated by transferring representations through the generic interface defined by the resource identifier.*
> 

This redefinition demonstrates the importance of the distinction between what is served and accepted by the API (a resource representation) and what the API really works with in the background (a resource). When building an API, designing resource representations should focus on what is useful to the clients that will consume the API. Regardless if we’re building something as specific as building a backend for frontend (BFF) or as generic as a public-facing API, the outermost layer’s design must be led with the question “How will this API be used?”. In contrast, the resource representation should not greatly influence the resources themselves - we don’t want to impose unreasonable restrictions on our DB schema, resource relations and processes for the convenience of the API consumers. Same logic can be applied the other way round: the way we manipulate the resources on the backend shouldn’t make consuming the API more complex than the process we’re implementing requires it to be. The backend owns the mapping of resource representations to the resources, which gives it the liberty to be creative and enable both sides to work as simple and efficiently as possible.

# URL standard

## Casing

All parts of an URL are written in `camelCase`.

✅ DO:

```
https://exampleWebsite.com/api/requestAnotherItem?pageSize=10
```

❌ DON’T:

```
// Not in camel case, and even worse, mixing multiple casings:
https://exampleWebsite.com/api/request-another-item?page_size=20
```

## Plural vs. singular resource names

Every project should determine what standard to use and stick to it. Note that the decision itself (plural or singular names) is not as important as being **consistent throughout the project**.

That being said, we recommend using plural where more than one resource exists (e.g., when we need to specify an ID to get the specific resource, otherwise we will get a list of resources), and singular if there is only one possible resource that can be fetched.

✅ DO:

```
https://exampleWebsite.com/api/users/1/devices/10
```

❌ DON’T:

```
// Contains both plural & singular:
https://exampleWebsite.com/api/users/1/device/10
// The same API as above, now contains singular, but it should be plural:
https://exampleWebsite.com/api/payment?userId=1
```

## Route values vs. query parameters

When determining which variable should be a route value or a query parameter, we must think about how the variable is related to the resource the API is working with. As a rule of thumb, if the variable uniquely defines the resource, it should be used as a route value. The most obvious example is the resource ID. On the other hand (pun not intended), query parameters describe some aspects of the resource, which are not necessarily related to just one resource, and are often optional. One exception to this rule is when we’re nesting the resources in the URL. In those cases, setting an identifiable variable for the parent resource can be used to filter out child resources related to that parent.

✅ DO:

```
// User ID defined as a route parameter:
https://exampleWebsite.com/users/1
// User ID defined as a query parameter, which is not uniquely defining a payment:
https://exampleWebsite.com/payments?userId=1
// User ID defined as a route parameter that filters profiles, which is a nested resource,
// while public=true defines a part of the profile resource that we're using to filter the response:
https://exampleWebsite.com/users/1/profiles?public=true
```

❌ DON’T:

```
// User ID defined as a query parameter, but it uniquely defines the returning resource:
https://exampleWebsite.com/users?id=1
// User ID defined as a route parameter, but used to filter out payments made by the user:
https://exampleWebsite.com/payments/1 // 1 is UserId
// It is not obvious which parameter is described by "true", nor is it a resource identifier
https://exampleWebsite.com/payments/true
```

## Pagination parameters

For offset pagination, we use the following parameters:

- `pageNumber` - number of the page, starting with 1.
- `pageSize` - number of items returned per page, must have a **max value defined**, which is usually 20.
- (Possible exception) - `pageIndex` - index of the page, starting with 0 - this can be used in special cases where it’s beneficial for the project to do so.

All parameters **must have a default value**.

Pagination should have a standardized response throughout the API - this is defined in the responses section of the document.

✅ DO:

```
// All parameters are defined and are within the limits:
https://exampleWebsite.com/payments?pageNumber=10&pageSize=15
// No parameter is defined, using default values:
https://exampleWebsite.com/payments
```

❌ DON’T:

```
// Page size is too big:
https://exampleWebsite.com/payments?pageNumber=10&pageSize=12582583215
// Using page index instead of page number:
https://exampleWebsite.com/payments?pageIndex=0&pageSize=20
```

# HTTP behavior

We follow the [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) standard as much as possible. This chapter contains a summary of the HTTP verb behaviors described in that standard.

## Key concepts

- **Safe**: A method is considered safe if it’s intended for information retrieval only. It should not have any side-effects, meaning that the state of the resource on the server should not be changed as a result of this method. Safe methods can be pre-fetched and cached by clients without any risk.
- **Idempotent**: A method is considered idempotent if making multiple identical requests has the same effect as making a single request. This doesn’t mean the response will be identical each time, but the server’s state will be the same after the first request as it is after the hundredth. This is important for designing resilient APIs that can safely retry requests after a network failure.

## GET

```jsx
Request:
GET www.example.com/users/1

Response:
200 OK
{
   "id": 1,
   "name": "S. Quelle",
}
```

- **Purpose:** To retrieve a representation of a specific resource. This is a read-only operation.
- **Safe:** Yes.
- **Idempotent**: Yes.
- **Request body**: Should not have a request body.
- **Possible responses:**
    - **200 OK** - if the request returns the requested resource. This response **must** have a body content.
    - **404 Not Found** - if the requested resource doesn’t exist for the user authorized in the request. Content body is optional, can be used only to describe the issue.
    - **400 Bad Request** - if the request is malformed, mostly if a parameter is not formatted correctly (e.g. request expects a GUID, but the received value is “1”). Response body is recommended, should be used to describe the issue
    - **401 Unauthorized** - if the client is not authorized to fetch the resource.

## POST

```jsx
Request:
POST www.example.com/users
{
   "name": "G. Olang",
}

Response:
201 Created
{
   "id": 2,
   "name": "G. Olang",
}
```

- **Purpose**: To submit data to be processed by a specific resource. It’s most often used to create a new resource that is a subordinate of the target URI.
    - In specific scenarios, the POST method can also be used for executing complex actions that don’t neatly fit other verbs. A common example is rejecting a request with a reason, where we have an endpoint `POST [example.com/requests/reject](http://example.com/requests/reject)` with a request body specifying the reason.
- **Safe**: No. It modifies the server state through processing the data.
- **Idempotent:** No, each request will create a new separate resource.
    - POST endpoints can be made idempotent by using idempotency headers (e.g., `X-Request-Id`), but that’s not our default behavior and should be only used if the project specifically requires it.
- **Request body:** Contains the data for the new resource or action.
- **Possible responses**:
    - **200 OK** - if the target resource was updated, but not created. Must include the resource in the response body.
    - **201 Created** - if a new resource was created as a result of this action. Must include the resource in the response body.
    - **204 No Content** - if the action was completed successfully, but nothing is included in the response body
    - **400 Bad Request** - if the action could not be completed due to an invalid request made by the client. Must contain the problem details in the response body
    - **401 Unauthorized** - if the user is not authenticated.
    - **403 Forbidden** - if the user does not have the permission to execute this action

## PUT

```jsx
Request:
PUT www.example.com/users/3
{
   "name": "J. Avah",
}

Response:
201 Created
{
   "id": 3,
   "name": "J. Avah",
}
```

- **Purpose**: To completely replace a resource at a specific URI.
    - Note that the PUT method can also be used to create a new resource on a specified URI, e.g., `PUT /user/123` will create a new user with ID `123`. **We do not support this by default**, it should only be used if the project has specific requirements for it.
- **Safe**: No.
- **Idempotent**: Yes.
- **Request body**: Contains the full representation of the resource to be replaced (or created).
- **Possible responses**:
    - **200 OK** - if an existing resource was replaced
    - **201 Created** - if a new resource was created
    - **204 No Content** - if an existing resource was replaced
    - **400 Bad Request** - if the action could not be completed due to an invalid request made by the client. Must contain the problem details in the response body.
    - **401 Unauthorized**
    - **403 Forbidden**

## PATCH

- By default, we **do not support** the PATCH method. This is because the behavior of this method is not strictly defined in the RFC standard, nor is there a commonly used standard for it.
- We have previously used this method in the projects that used JSON:API standard, which explicitly does not support PUT, only PATCH.
- If the project requires usage of a PATCH method, we recommend using the JSON PATCH standard which [.NET supports](https://learn.microsoft.com/en-us/aspnet/core/web-api/jsonpatch?view=aspnetcore-9.0). However, there are security implications with this standard the developers must address when using it.

## DELETE

```jsx
Request:
PUT www.example.com/users/4

Response:
204 No Content
```

- **Purpose:** To remove a specific resource.
- **Safe**: No.
- **Idempotent**: Yes - the response might not always be the same (subsequent requests might get a `404 Not Found` response), but the outcome on the server will be the same.
- **Request body**: Should not have a request body.
- **Possible responses:**
    - **200 OK**
    - **204 No Content**
    - **400 Bad Request**
    - **401 Unauthorized**
    - **403 Forbidden**
    - **404 Not Found**

## Other common verbs

### `HEAD`

- **Purpose**: Identical to `GET`, but the server **must not** send a response body. It's used to check a resource's metadata (like `Content-Type` or `Last-Modified` headers) without downloading the entire resource.
- **Safe**: **Yes**.
- **Idempotent**: **Yes**.

### `OPTIONS`

- **Purpose**: To describe the communication options for the target resource. It's often used by browsers for CORS (Cross-Origin Resource Sharing) preflight requests to see which methods and headers are allowed by the server.
- **Safe**: **Yes**.
- **Idempotent**: **Yes**.

## Returning the target resource in the response body

The RFC standard does not specify if a created, updated, or deleted resource needs to be included in the response body. This decision is left to the developers to decide, depending on the project needs. Note that whatever decision is made on the project-level, **it must be consistent for all endpoints** - if the decision is made to return the resource, that must be done throughout the API, not just specific endpoints.

We recommend that, by default, the API **does not** return the content it created or updated, although we do recommend returning the ID of the new resource if it was created as a result of a request. That being said, there are a lot of cases where the clients depend on the data returned to update their state, so be mindful of the API consumers when making that decision.

# Responses

## Media type

All endpoints within a single API should support the same media types. The exceptions can be endpoints like web-hook URLs that must conform to another system’s requirements, and endpoints that return specific files, like images or documents. This means that if an API supports a specific standard, like JSON HAL, it must do so on all endpoints used by a single client.

## Pagination

All paginated endpoints within a single API must return the pagination metadata for a specific pagination type in the same format. Every pagination response **must** include a `page` object that contains the following properties:

- `size` - size of the page, can be defined by the client in the request or can be default (e.g. 20)
- `totalElements` - total number of elements that would be returned if the result was not paginated
- `totalPages` - total number of available pages, calculated using the `size` and `totalElements`
- `number` - current page number, can be defined by the client in the request or can be default (1)

All returned items must be contained inside the `items` list.

By default, we use the following format (shown here in JSON, but can be converted to specific API standard if needed):

```json
{
	"page": {
		"size": 10,
		"totalElements": 112,
		"totalPages": 12,
		"number": 2
	},
	"items": [
		{
			"id": 20,
			"name": "some entity",
		},
		...
	]
}
```

## Error response

In case the API needs to return an error response (4xx or 5xx), the response body **must** contain a standardized error response. The response model we use is **Problem Details for HTTP APIs**, defined in [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html).