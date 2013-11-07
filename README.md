# What is this thing? #
This document defines the REST API specification implemented by public Go Daddy&reg; APIs.

Our goal is to make APIs that are as easy to use as possible.  Each service has documentation available, but this should be a supplement, not required reading.  Armed with just the URL and credentials, a user should be able to navigate their way through the API in a web browser to learn about what resources and operations it has.  In other words, the API should be _discoverable_.  Once you're familar with what's available, requests can be made with simple tools like cURL or any basic HTTP request library.

# Why are you publishing it? #

Maybe you'd like to build your own tools that work with our services.  Or maybe you want to build your own service that follows the same style as ours.  We'd love it if you did either one and want to give you all the information you need.  This is also the same documentation our internal teams use to build their APIs.

# What if I just want to use one of your APIs? #
See the documentation for the specific API you are interested in.  There are clients for various languages available:
  - [PHP](https://github.com/godaddy/gdapi-php)
  - [C#](https://github.com/godaddy/gdapi-csharp)
  - Python &mdash; coming soon
  - Command Line &mdash; coming soon
  - Node.js &mdash; coming soon
  - Java &mdash; coming soon

# Contact #
For questions, comments, corrections, suggestions, etc, you can use the usual tools in GitHub or email us at [apispec@godaddy.com](mailto:apispec@godaddy.com).  Please use our normal [support page](http://support.godaddy.com/) for questions or problems about a specific API, product, or account.

# License #
Copyright &copy; 2012-2013 Go Daddy Operating Company, LLC 

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Examples #
Examples demonstrate a hypothetical file storage API.  Only the relevant HTTP request and response information is shown, additional standard HTTP headers will be present in a real request.  Examples generally show JSON request and responses, but several [request](#format) types are available and other [response](#format) types may be defined in the future.

----------------------------------------

# Table of Contents #

- [Terminology](#terminology)
- [Status Codes](#status-codes)
- [Representations](#representations)
  - [Format](#format)
  - [Special Headers](#special-headers)
  - [Special Characters](#special-characters)
  - [Dates](#dates)
  - [HTML UI](#html-ui)
- [Errors](#errors)
- [Resources](#resources)
- [Collections](#collections)
  - [Inheritance](#inheritance)
- [Links](#links)
  - [Construction](#construction)
- [Schemas](#schemas)
- [Actions](#actions)
- [Operations](#operations)
  - [Query](#query-operation)
      - [Pagination](#pagination)
      - [Sorting](#sorting)
      - [Filtering](#filtering)
      - [Client Request](#client-request)
      - [Collection Response](#collection-response)
  - [Read](#read-operation)
      - [Root Level](#root-level)
      - [Version Root](#version-root)
      - [Individual Resource](#individual-resource)
      - [Headers Only](#headers-only)
  - [Create](#create-operation)
      - [Individual Resource](#single-resource)
        - [Form Post](#single-resource---form-post)
      - [Multiple Resources](#multiple-resources)
  - [Update](#update-operation)
      - [Individual Resource](#single-resource-1)
      - [Multiple Resources](#multiple-resources-1)
  - [Delete](#delete-operation)
      - [Individual Resource](#single-resource-2)
      - [Multiple Resources](#multiple-resources-2)
      - [Deleted Resources](#deleted-resources)
  - [Replace](#replace-operation)
  - [Action](#action-operation)
- [Nesting](#nesting)
- [Resource Versioning](#resource-versioning)
- [Base URL and Versioning](#base-url-and-versioning)
- [Authentication](#authentication)
- [Design Considerations](#design-considerations)
  - [Identifiers](#identifiers)
  - [Naming Conventions](#naming-conventions)
  - [What to Link](#what-to-link)
  - [Canonical Links](#canonical-links)
  - [Regions](#regions)
  - [Caching](#caching)
  - [ETag](#etag)
  - [Last-Modified](#last-modified)
  - [Cache-Control](#cache-control)
  - [Compression](#compression)
  - [Query Strings](#query-strings)

----------------------------------------

# Terminology #
**REST:**
A **RE**presentational **S**tate **T**ransfer service is a style of API where a client makes HTTP requests to manipulate resources identified by the request URL.  RESTful services are stateless, so no session state is stored on the server between requests.  Each requests contains all the information needed to service that request.

**Resource:**
A **resource** is an object or concept that can be manipulated by the API.  In our file storage example, they will be things like a 'Folder' and a 'File'.  Resources are the building blocks of an API, the nouns in the language describing the system.  All operations in the API manipulate the resources or their state.  Resources should be organized in a way that is useful to the client.  This is the most important part of the design of an API; if your resources just match your database schema one-to-one, chances are your API is not going to be very easy to use.

**Representation:**
REST APIs transfer **representations** of resources back and forth between the client and the service.  These are documnents that describe resources.  Representations are most commonly JSON-formatted, but can be of any type.

**Attribute:**
Representations of resources consist of a series of **attributes** names and values as key-value pairs.

**Collection:**
A **collection** is a special kind of resource that represents a group of related resources.  It contains a representation of each resource, and some additional information for things like pagination and filtering.

**Identifier:**
Every resource in a collection MUST have an **identifier** attribute with a value that is unique within that collection.

**Side Effects:**
Requests that change the state of something in the service are said to have **side effects**.  It is very important that certain types of requests not have side-effects.

**Idempotent:**
An operation is *idempotent* if it can be applied multiple times without changing the result after the first application.

**Opaque:**
Some pieces of information returned from a service are considered to be **opaque**.  Opaque values are a black box that has meaning only to the service; clients should not rely on any particular format, length, or content in an opaque value.  They should not change the value or try to parse any meaning out of it.  Opaque values MAY change between API revisions or even individual requests, so they should never be stored or compared against.  For example, pagination URLs contain a "marker" attribute that identifies the appropriate page to load.

**Action:**
An **action** is an operation that can be performed on a resource but does not fit into one the standard create/read/update/delete operations.  For example "sharing" or "encrypting" a file resource.

**Discoverability:**
**Discoverability** is a method of self-documenting.  Each API has a collection of schemas that define what resource types are available and what attributes they have.  Each response contains information about how to find related resources and what actions are available.

Since this documentation is in a computer-readable form, it is easy to make a generic client library that will speak to any API that implements this specification, with no code related to a particular service.  We provide several [client libraries](#what-if-i-just-want-to-use-one-of-your-apis) for different languages, and if you load the API in a browser we respond with a built-in JavaScript client that provides a friendly UI for experimentation.

----------------------------------------

# Status Codes #
Every HTTP response contains a **status code**.  This is the primary mechanism that a client uses to determine the result of their request, so it is very important that they be clearly defined and used consistently.  Requests that were successfully handled MUST return a response with a 2xx or 3xx status code.  Requests that cannot be processed MUST return a 4xx or 5xx code.

Code | Meaning
--------------------------|----------------------------
**2xx** | **Success**
200 OK | The request was successful.
201 Created | Success, and a new resource has been created.<br/>&bull; A Location header to the resource SHOULD be included.
202 Accepted | The request has been received but has not completed.  It MAY be completed in the future.<br/>&bull; This is typically used for requests that produce a long-running process that will be performed asynchronously and could fail later.
204 No Content | The request was successful, and the response will have no body portion.
| 
**3xx** | **Redirect**
301 Moved Permanently | Permanent redirect.  The requested resource is somewhere else now and will never be coming back.<br/>&bull; The new location MUST be specified in the Location header.<br/>&bull; The operative word here is *permanent*.  If in doubt, use 302.<br/>&bull; Clients may remember this and never request the old URL again.
302 Found | Temporary redirect.  The requested resource isn't here right now.<br/>&bull; The new location MUST be specified in the Location header.<br/>&bull; The client will ask for the original URL if it wants this resource again in the future.
304 Not Modified | The client made a conditional GET request for a resource, and that resource matches the condition so the response body does not need to be sent.
| 
**4xx** | **Client Error** &mdash; The client did something wrong.<br/>&bull; Sending the same request again **will** result in the same error.
400 Bad Request | The request was malformed in some way; the client should feel bad.
401 Unauthorized | Authentication information was not sent or is invalid.
403 Forbidden | Authentication information was validated, but does not give the client access to the requested resource.
404 Not Found | These are not the droids you are looking for.
405 Method Not Allowed | The requested HTTP method is not allowed for this URL.
406 Not Acceptable | The service does not support responding the representation type that the client **requested**.
409 Conflict | [Resource versioning](#resource-versioning) is enabled, and the requested operation conflicts with the current state.
413 Entity Too Large | The request body is bigger than is accepted by the service.
414 Request-URI Too Long | The requested URL is longer than is accepted by the service.
415 Unsuported Media Type | The service does not support handling the representation type that the client **sent**.
418 I'm a teapot | The request attempted to brew coffee with a teapot service that is not compliant with [RFC2324](http://tools.ietf.org/html/rfc2324).
422 Unprocessable Entity | The request was well-formed (400) and in a supported format (415), but cannot be processed.<br/>&bull; This is typically used for application-specific validation errors.
| 
**5xx** | **Service Error** &mdash; Something bad happened within the service.<br/>&bull; There is nothing the client can do about this.<br/>&bull; The same request may succeed (or produce a 4xx error) if re-submitted in the future.
500&nbsp;Internal&nbsp;Server&nbsp;Error | Generic something is broken in the service; the service should feel bad.
503 Service Unavailable | The request couldn't be handled due to maintenance or overload.

----------------------------------------

# Representations #
A representation is the way a resource is described/serialized in a HTTP request or response.  All services MUST support:

- JSON (application/json) for both requests and responses.
  - This is the primary representation used by all APIs.
- HTML (text/html; charset=utf8) for responses
  - This is a small HTML wrapper around the JSON format which presents the [HTML UI](#html-ui).
  - The HTML response format is only intended for use in a web browser, not for clients to parse.
- Forms ([multipart/form-data](http://tools.ietf.org/html/rfc2388) and [application/x-www-form-urlencoded](http://www.w3.org/TR/html401/interact/forms.html#form-content-type)) for requests.
  - These are needed for the [HTML UI](#html-ui) and are easy to create with browsers and command-line tools like cURL.

Other representation Content-Types that make sense for the particular application MAY also be supported.

## Format ##
Clients MAY specify the representation they would prefer to receive by including an appropriate [Accept](http://tools.ietf.org/html/rfc2616#section-14.1) header.
  - Service SHOULD be lenient in mapping this to one of their acceptable formats.
  - For example <code>application/json</code>, <code>text/json;charset=utf-8</code>, and <code>text/json</code> can all be interpreted as JSON.

If the request is from a web browser, the response SHOULD be HTML.
  - The suggested definiton of "a web browser" is that the Accept header contains "*/*" and the User-Agent header contains (case-insensitive) "mozilla".
  - This matches all the common graphical web browsers but not HTTP request libraries or command-line browsers.

When clients are sending a representation to the service (e.g. creating a resource), they MUST specify the format using the [Content-Type](http://tools.ietf.org/html/rfc2616#section-14.17 Content-Type) header.
  - If the service does not support the specified Content-Type, it SHOULD return a 406 error.

## Special Headers ##
A <code>X-API-Schemas:</code> MUST be present in all responses.  The value tells the client the URI where they can find the collection of schemas to understand your responses.
  - The URI SHOULD be for the particular [API version](#api-versioning) the request is for.
  - If the request is for the [root level](#root-level), it SHOULD be for the latest version.

## Special Characters ##
Escaping the forward slash ("/") character is not required by the [JSON specification](http://json.org/), but requires consideration.
  - When responding as JSON, slashes SHOULD NOT be escaped.
  - When responding as HTML, strings in the JSON data portion MUST have forward slashes escaped, typically as "\/".
    - Failure to do this exposes the page to an XSS attack (using strings that contain "&lt;/script&gt;").

## Dates ##
Dates MUST be represented as an [ISO-8601](http://www.w3.org/TR/NOTE-datetime) formatted string.
  - If a time component is included, a time zone designator MUST BE included and SHOULD always be UTC ("Z").
  - If you have a very good reason to use something other than UTC, the attribute name SHOULD indicate that it is "local", "user", or similar to avoid confusion.
  - e.g. "birthday": "1982-02-24", "created": "2012-09-27T18:39:53Z", "yourLocalTime": "2013-09-27T11:30:42-07:00"

## HTML UI ##
Services provide a HTML version of their API by wrapping the JSON response with the snippet below.  It includes JavaScript and CSS that pretty-prints the response and displays a bar that provides buttons for operations, actions, sorting, filtering, pagination, etc.

```html
<!DOCTYPE html>
<!-- If you are reading this, there is a good chance you would prefer sending an
"Accept: application/json" header and receiving actual JSON responses. -->
<link rel="stylesheet" type="text/css" href="http://img1.wsimg.com/ucp/api/1.2.10/ui.css" />
<script src="http://img1.wsimg.com/ucp/api/1.2.10/ui.js"></script>
<script>
var docs = "http://url-to-your-docs/site";
var schemas = "http://url-to-your-api/v1/schemas";
var data = { 
  /* ... JSON response ... */
};
</script>
```

----------------------------------------

# Resources #

Resource representations MUST have these attributes:
  - <code>type:</code> A string that uniquely identifies the kind of resource this is.
    - The value MUST be the id of a [schema](#schemas).

SHOULD have, in almost all cases:
  - <code>id:</code> A string which uniquely identifies this resource.  (See [identifier](#identifiers) design considerations).
  - <code>links:</code> A map with [links](#links) to:
    - <code>self:</code> The location of the resource itself.
  - Other application-specific information attributes
 
And MAY have, when appropriate:
  - Additional <code>links:</code> entries for related resources and collections.
  - <code>actions:</code> A map of [action](#actions) names &rarr; the URL that is used to invoke them.

```javascript
{
  "id":      "b1b2e7006be", 
  "type":    "file",
  "links":   { /* see links */ },
  "actions": { /* see actions */ },
  "name":    "ultimate_answer.txt",
  "size":    2,
  /* ...  more application-specific attributes ... */
}
```

----------------------------------------

# Collections #

Collections are a special kind of resource.  They have a "data" array that contains an array of resources.

Collection representations MUST have:
  - <code>type:</code> "collection"
  - <code>resourceType:</code> The id of the [schema](#schemas) for the type of resource this collection holds.
  - <code>data:</code> An array containing resource representations
    - This MUST always be present and be an array, even if there are 0 or 1 entries in it.

SHOULD have, in almost all cases:
  - <code>links:</code> A map with [links](#links) to:
    - <code>self:</code> The location of the resource itself.

And MAY have, when appropriate:
  - Additional <code>links:</code> entries for related resources and collections.
  - Additional application-specific attributes (on the collection, as it is itself a resource).
  - <code>actions:</code> A map of action names &rarr; the URL that can be requested to perform them (on the collection).
  - <code>pagination:</code> See [pagination](#pagination).
  - <code>sort:</code> See [sorting](#sorting).
  - <code>filters:</code> See [filtering](#filtering).
  - <code>createDefaults:</code> A map of field names &rarr; their values.
    - This values provide a context-specific default value for fields which a client MAY use when creating a new resource in the collection.
  - <code>createTypes:</code> See inhertance, below.

```javascript
{
  "type": "collection",
  "resourceType": "file",
  "links": { 
    "self":     "https://base/v1/files",
    /* ... more links ... */
  },
  "actions":    { /* see actions */ },
  "pagination": { /* see pagination */ },
  "sort":       { /* see sorting */ },
  "filters":    { /* see filtering */ },
  "data": [
    {/* file resource */},
    {/* file resource */},
    /* ... more resources ... */
  ],	
  /* ... application-specific attributes ... */
}
```

### Inheritance ###
Collections MAY support multiple resource types through simple inheritance:
  - The <code>resourceType:</code> SHOULD refer to the 'base' resource type.
  - Define additional sub-types that have all the same fields and filters as the base type, plus additional info.
  - Return a <code>createTypes:</code> map in the collection, with a mapping of type ids that can be created &rarr; the URL where they SHOULD be POSTed.
    - If the 'base' type can also be created, it SHOULD be present in the map.
    - The URL for each entry MAY be the same as the collection's self link, or it can be different to provide additional flexibility in the service.

```javascript
  "createTypes": {
    "file":      "https://base/v1/files",
    "textFile":  "https://base/v1/files",
    "imageFile": "https://base/v1/files?image=1",
  },
```

----------------------------------------

# Errors #
Error responses MUST be a vaild [resource](#resources) which gives more information about the error.  At a minimum, they MUST include:
  - <code>type:</code> "error"
  - <code>status:</code> the HTTP status code of the response
  - <code>code:</code> A short identifier for the error, suitable for identifying what specific kind of error this is within client code.

Errors SHOULD also include:
  - <code>message:</code> A short description of the error suitable for a human to read
  - <code>detail:</code> Additional more detailed information, if available.
  - Any other application-specific information that is available.

Error responses MUST be returned in the representation format that the client asked for.
  - It is rude to send an HTML error page back to a client that made a request for JSON.
  - Consider responses that might be returned from application servers and web proxies within your stack.
  - Respond with no body if it is not possible to respond in the correct format, or if an error occurs within [content negotiation](#format).

```javascript
{
  "type":    "error",
  "status":  404,
  "code":    "FolderNotFound",
  "message": "The specified folder does not exist",
  "folderName": "Documents",
  /* ... more application-specific info ... */
}
```

----------------------------------------


# Links #
Links provide a trail for the client to follow to get to related information that is not included in the response.

Clients
  - SHOULD use the links provided to navigate resources.
  - SHOULD NOT hard-code or string-concatenate URLs (other than the version root).

Services
  - SHOULD provide all appropriate links to that a client can get to anything exclusively by navigating resource links.
  - SHOULD strive to keep URL layout stable, as clients may save link values and expect to be able to retrieve them later.
  - MAY alter URL layout and link values, if necessary.
    - This is not considered a [breaking change](#api-versioning), but the Service SHOULD redirect old URLs to the new ones.

```javascript
{
  "links": {
    "self":     "https://base/v1/files/b1b2e7006be",
    "content":  "https://base/v1/files/b1b2e7006be/content",
    "folder":   "https://base/v1/folders/19c3932wef",
    "public":   "http://documents.your-files.com/ultimate_answer.txt"
    /* ... more links ... */
  },
  /* ...other attributes... */ 
}
```

Link entries SHOULD lead to a valid destination.  For example, if you know a resource has no "public" URL because it is not shared, do not include the "public" attribute.

See also: [What to link](#what-to-link)

### Construction ###
Link values are strings and MUST be an absolute URL, including the scheme, host, port, and [API Version](#api-versioning).
  - This prevents the client needing resolve relative URLs into absolute ones.
  - [HTTP compression](#compression) makes the overhead of transmitting absolute vs relative URLs negligible.
  - Port SHOULD be omitted if it is the default for the scheme (e.g. 443 for https).

Handling forward slashes:
  - Trailing slashes SHOULD NOT be included on URLs produced by a service.
  - SHOULD have no effect on the response if a client sends one.
  - Multiple slashes in a URL SHOULD be treated as a single slash.

----------------------------------------

# Schemas #

Each service MUST have a top-level collection called "schemas" which contains "schema" resource for each resource type.  The information contained in the schemas provides enough information to build a smart client that knows what actions are available, what fields make up a resource, etc.

The "schemas" collection MUST also have a link to the base URL of the API version.
  - This makes the schema collection (https://base//v1/schemas) a single URL that provides a client everything they need to know about the service, similar to a WSDL in a SOAP service.

Each schema resource MUST describe:
  - <code>id:</code>: The name of the resource type (e.g. "folder").
  - <code>type:</code> "schema"
  - <code>links:</code>: 
    - <code>self:</code>: The URL for this schema (e.g. "https://base/v1/schemas/folder")
    - <code>collection:</code> If this type can be queried/listed, the URL for doing so (e.g. "http://base/v1/folders").
      - This SHOULD correspond to the collections that are linked at the [version root](#version-root).
      - This link is used by API clients to generate object stubs as the URL to fetch data from.
      - This link is used by the [HTML UI](#html-ui) to provide a dropdown of choices for fields that are of type reference.
  - <code>resourceFields:</code>: A map of attribute names available in **resources** of this type &rarr; descriptions of that field (see [fields](#schema-fields)).

And SHOULD describe if applicable:
  - <code>resourceMethods:</code>: An array of HTTP methods that are available to some (but not necessarily all) **resources** of this type.
  - <code>resourceActions:</code>: A map detailing the actions available to **resources** of this type (see [actions](#actions)).
  - <code>collectionMethods:</code>: An array of HTTP methods that are available to a **collection** of this type.
  - <code>collectionActions:</code>: A map detailing the actions available to **collections** of this type (see [actions](#actions)).
  - <code>collectionFields:</code>: A map detailing the non-standard attribute fields that the **collection** has. (see [fields](#schema-fields)).
  - <code>collectionFilters:</code>: A map detailing the filters that are available to **collections** of this type (see [filters](#filtering)).

### Schema Fields ###
Each field is defined by a map of properties.  Fields MUST have a <code>type:</code>, which may be a "simple" type:

type        | description
------------|------------
"string"    | UTF-8 string
"password"  | String that should not be displayed
"float"     | JavaScript uses double-precision [IEEE 754](http://en.wikipedia.org/wiki/IEEE_floating_point)
"int"       | "int"s are just floats in JavaScript, so the min/max safe values are &#177; 2<sup>53</sup>.  Other clients may have signed 32-bit integers, so be careful with any values that may approach 2<sup>31</sup>.
"date"      | As a string, see [dates](#dates)
"blob"      | Binary data, encoded as a string
"boolean"   | Boolean

Or a non-simple type:

type                                  | description
--------------------------------------|-------------
"enum"                                | One choice out of an enumeration of possible values
"reference[<code>schema_name</code>]" | The value is the id of a resource of type <code>schema_name</code>
"type[<code>schema_name</code>]"      | The value is the representation of a resource of type <code>schema_name</code>
"array[<code>another_type</code>]"    | The value is an array of entries of type <code>another_type</code>
"map[<code>another_type</code>]"      | Key/value pairs.  Keys are strings, Values are of type <code>another_type</code>

Additional metadata SHOULD be provided for each field when applicable:

name                              | type                              | description
----------------------------------|-----------------------------------|--------------
<code>default:</code>             | <code>same as field</code>        | The default value that will be used if none is specified
<code>unique:</code>              | boolean                           | If true, this field's value must be unique with its resource type
<code>nullable:</code>            | boolean                           | If true, null is a valid value for this field
<code>create:</code>              | boolean                           | If true, this the field can be set when creating a resource
<code>required:</code>            | boolean                           | If true, this field must be set when creating a resource
<code>update:</code>              | boolean                           | If true, this field's value can be changed when updating a resource
<code>minLength:</code>           | int                               | For strings and arrays, the minimum length (inclusive)
<code>maxLength:</code>           | int                               | For strings and arrays, the maximum length (inclusive)
<code>min:</code>                 | int                               | For ints and floats, the minimum allowed value (inclusive)
<code>max:</code>                 | int                               | For ints and floats, the maximum allowed value (inclusive)
<code>options:</code>             | array[<code>same as field</code>] | For enums, the list of possible values
<code>validChars:</code>          | string                            | For strings, only these characters are allowed (see [character ranges](#character-ranges))
<code>invalidChars:</code>        | string                            | For strings, these characters are not allowed (see [character ranges](#character-ranges))
<code>referenceCollection:</code> | string                            | For references, a query URL that can be used to find valid resources of the reference type

### Validation ###
The additional fields above provide enough info for a client to do basic validation of values before submitting them to a service.  They are not intended to be completly comprehensive; A service will often have additional restrictions on values that cannot be represented here.  If the service is given a value that does not match all of it's conditions, it should return a 400 error with enough detail for the client to fix the problem and re-submit the reuqest.

#### Character Ranges ####
The <code>validChars:</code> and <code>invalidChars:</code> are case-sensitive and may contain ranges as in a simple regular expression.  For example alphanumerics may be represented as "a-zA-Z0-9".  Unicode characters should be represented as <code>\uXXXX</code> or <code>\uXXXXXX</code>, where X are the hexadecimal representation of the codepoint.

```javascript
{
  "id": "file", 
  "type": "schema",
  "resourceMethods": ["GET","DELETE"],
  "resourceFields": {
    "name":   {"type": "string", "required": true, "create": true, "update": true},
    "size":   {"type": "int"},
    "owner":  {"type": "reference[user]"},
    "access": {
      "type":    "enum",
      "options": ["public","private","requirepassword"],
      "default": "private",
      "create":  true,
      "update":  true
    },
    /* ... more fields ... */
  },
  "resourceActions": {
    "encrypt": {
      "input":  "cryptInput",
      "output": "file",
    },
    /* ...more actions... */
  },
  "collectionMethods": ["GET","POST"],
  "collectionActions": {
    "archive" {
      "input":  "archiveInput",
      "output": "file",
    }
    /* ...more actions... */
  },
  "collectionFields": {
    "size": {"type": "int"}
  },
  "collectionFilters": {
    "name":   {"modifiers": ["ne","prefix","suffix"]},
    "date":   {"modifiers": ["lt","gt","lte","gte","prefix"]}
    "size":   {"modifiers": ["lt","gt","lte","gte"] }
    "access": {"modifiers": ["eq","ne"], "options": ["public","private","requirepassword"]}
  }
}
```

----------------------------------------

# Actions #
Actions perform an operation on a resource and (optionally) return a result.  They may take an input resource and typically do something that requires application logic beyond what could be done with a simple [create](#create-operation), [update](#update-operation), or [delete](#delete-operation) operation.

Types that have actions available have an <code>actions:</code> and/or <code>collectionActions:</code> map in their schema detailing all the actions that are possible and their input/output schema.  For example a file resource may have "share", "encrypt", and "decrypt" actions.  See [schemas](#schemas) for more information.

### In Resources ###
Resources have a <code>actions:</code> attribute that details what actions are available for this particular resource, and what URL is to be accessed to perform them.  For example if only one of "encrypt" and "decrypt" are possible for a given file, only the one currently available would appear in the resource:

```javascript
{
  "id": "b1b2e7006be",
  "type": "file",
  "actions": {
    "share":   "https://base/v1/files/b1b2e7006be?share",
    "encrypt": "https://base/v1/files/b1b2e7006be?encrypt"
  },
  /* ... other resource attributes ... */
}
```

Action URLs MUST be considered opaque strings to the client.  A simple query parameter is suggested, but services may construct URLs in any format they desire.  Examples:
  - "https://base/v1/files/b1b2e7006be?encrypt"
  - "https://base/v1/files/b1b2e7006be?action=encrypt"
  - "https://base/v1/files/b1b2e7006be/encrypt"
  - "https://base/v1/actions/encryptFile/b1b2e7006be"

### In Schema ###
Actions are defined by a map with 2 attributes:
  - <code>input:</code> The ID of the schema that the inputs will conform to
  - <code>output:</code> The ID of the schema that the outputs will conform to

Inputs and outputs are resources and MUST have an associated schema.  If the action takes no input or has no output, the appropriate attribute SHOULD be omitted.

```javascript
{
  "id":      "file",
  "type":    "schema",
  "resourceActions": {
    "share": {
      "input": "shareInput",
    },
    "decrypt": {
      "input": "cryptInput",
      "output": "file",
    },
    "encrypt": {
      "input": "cryptInput",
      "output": "file",
    },
    /* ... other actions ... */
  },
  /* ... other schema attributes ... */
}
```

----------------------------------------

# Operations #

Operation | HTTP Method | Request URL | Description
----------|-----------------|-----------------------------------|--------------------
[Query](#query-operation) | GET or HEAD | https://base/v1/{collection_name} | Retrieve a collection of resources
[Read](#read-operation) | GET or HEAD | https://base/v1/{collection_name}/{resource id} | Retrieve a single resource
[Create](#create-operation) | POST | https://base/v1/{collection_name} | Creates one or more resources
[Update](#update-operation) | PUT | https://base/v1/{collection_name}/{resource_id}<br>https://base/v1/{collection_name} | Change one or more existing resources
[Delete](#delete-operation) | DELETE | https://base/v1/{collection_name}/{resource_id}<br>https://base/v1/{collection_name} | Delete one or more existing resources
[Action](#action-operation) | POST | https://base/v1/{collection_name}/{resource_id}?{action_name}<br>https://base/v1/{collection_name}?{action_name} | Perform an action on a resource or collection

----------------------------------------

# Query Operation #
The query operation allows a client to list the resources that are in a collection.  Query operations MUST NOT incur side effects.

```http
GET /v1/folders HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "type": "collection",
  "resourceType": "folder",
  "links": {
    "self":    "https://base/v1/folders",
    /* ... more links ... */
  },
  "data": [
    {
      "type":    "folder",
      "id":      "b1b2e7006be",
      "name":    "Documents",
      "links":   { /* see links */ },
      "actions": { /* see actions */ }
    },
    /* ... more folder resources ... */
  ],
  "actions": { 
    "deleteAll": "https://base/v1/folders?deleteAll", 
  },
  "pagination": { /* see pagination */ },
  "sort":       { /* see sorting */ },
  "filters":    { /* see filters */ }
}
```

### Pagination ###
Collections that may return a large number of results SHOULD paginate the response.

Pagination SHOULD be controlled by adding a <code>marker={opaque_string}</code> query string parameter.
  - The implementation and meaning of the opaque string is left to the service.
  - A MySQL-style numerical offset is simple, but can return the same entry on 2 subsequent pages if the result set changes between requests.
  - A marker that specifically identifies the last row returned prevents this.

Services SHOULD allow the client to specify how many results they would like per page with a <code>limit={integer}</code> query string parameter.
  - The service SHOULD enforce (and document) an upper limit on how high this is allowed to go.  At least 1000 is suggested.
  - Services SHOULD support <code>limit=0</code> so that clients can get just the metadata for a collection (sort, filter, etc).

If the collection supports pagination, the response MUST contain a "pagination" object containing:
  - <code>limit:</code> The number of items that are returned in each result set.
  - <code>partial:</code> Whether this result is the complete set (true) or if it's been truncated (false).

If the result has been truncated, it SHOULD also contain:
  - <code>first:</code> The URL of a page containing the first resource in the result set (this SHOULD be the same URL, without a marker).
  - <code>previous:</code> The URL of the page immediately preceding the current result set.
  - <code>next:</code> The URL of the page immediately following the current result set.

And MAY additionally contain:
  - <code>last:</code> The page containing the last resource in the result set.
    - This may be impractical to implement, depending on your data source.
  - <code>total:</code> The total number of items in the result set, if available.
    - This may also be impractical, but is very useful information to a client if possible.

When the current result set contains the first resource, 'first' and 'previous' SHOULD NOT be returned. 
When the current result set contains the last resource, 'next' and 'last' SHOULD NOT be returned. 

URLs included in this section SHOULD include all query parameters needed to maintain the current [sort](#sorting) and [filter](#filtering).

```javascript
{
  /* ... other collection attributes ... */
  "pagination": {
    "first":    "https://base/v1/files",
    "previous": "https://base/v1/files?marker=opaque_string1",
    "next":     "https://base/v1/files?marker=opaque_string2",
    "last":     "https://base/v1/files?marker=opaque_string3",
    "limit":    100,
    "total":    342,
    "partial":  true
  },
  /* ... other collection attributes ... */
}
```

### Sorting ###
Services MAY allow clients to request a sorted list of resources in a collection by adding <code>sort={sort name}</code> and <code>order={asc or desc}</code> query string parameters to the request.  Clients MAY use these directly when constructing their query, but SHOULD use the links contained in results.

If a collection can be sorted, the response SHOULD contain a <code>sortLinks:</code> object, a mapping of attributes that can be sorted &rarr; the URL to produce that sort.

If the response was sorted by something, it SHOULD contain a <code>sort:</code> object containing:
  - <code>name:</code> The name of the current sort, so that the client knows which column to show as sorted.
  - <code>order:</code> The direction of the current sort.  This SHOULD be either <code>"asc"</code> or <code>"desc"</code>.
  - <code>reverse:</code> The URL for the same sort in the opposite order.

If no sort is specified by the client, services SHOULD either:
  - Choose a default, and include the <code>sort:</code> attribute as if it were requested
  - Or return results in an undefined/unstable order, with no <code>sort:</code> attribute).

Sorts MUST be "stable".  This means they must produce a unique and repeatable ordering such that the same query will always return results in the same order.
  - This is necessary for [pagination](#pagination), and is generally what a client would expect.
  - For example the "size" of a file is not unique, so the results should be sorted by something else to make it stable.
  - A simple way to do this is to always sort by what the user wants, then by ID or unique name.

URLs included in this section SHOULD include all parameters needed to maintain the current [filter](#filtering), but SHOULD NOT include information about pagination.  When following a link to a different sort order, the client SHOULD end up back on the first page of results again.

Multi-level sorting is not defined.

```javascript
{
  /* ... other collection attributes ... */
  "sort": {
    "name":    "size",
    "order":   "asc",
    "reverse": "https://base/v1/files?sort=size&order=desc",
  },
  "sortLinks": {
    "name":       "https://base/v1/files?sort=name",
    "size":       "https://base/v1/files?sort=size",
    "upload_date: "https://base/v1/files?sort=upload_date",
    /* ... more sort options ... */
  },
  /* ... other collection attributes ... */
}
```

### Filtering ###
Services MAY support searching/filtering of the collection.  Support for this is indicated by <code>collectionFilters:</code> in the schema, and the presence of a <code>filters:</code> map in the collection response.

#### Client request ####
The client performs a search by starting with the URL for a standard query request.  If they've already done a query and want to filter the results, the <code>self:</code> link of the result can be used instead.  The client then adds on query string parameters for <code>{filter name}{"_"+modifier}={value}</code>.
  - The underscore and modifier are optional if the desired modifier is <code>eq</code>.
    - <code>name_eq=a</code> and <code>name=a</code> are equivalent.
  - For example to search for filenames starting with "a" and greater than 1kb, the client would append:
    - <code>name_prefix=a&size_gt=1024</code>
  - To search for non-image files, they could append:
    - <code>name_notlike=%.jpg&name_notlike=%.png</code>

All special characers in filter values MUST be [URL-encoded](http://tools.ietf.org/html/rfc3986#section-2.4).
Note: "%" characters are shown here for clarity, but would be "%25" in an actual request.  All special characers MUST be [URL-encoded](http://tools.ietf.org/html/rfc3986#section-2.4).

There is no mechanism defined for OR-ing conditions.  All filters are AND-ed together.

#### Collection response ####
Collections that support filtering SHOULD have a <code>filter:</code> map in the response, with an attribute for each field that can be filtered on.
  - If any filter was applied for that field, the value should be an array describing the filters.
  - If no filters were applied for that field, the value should be null.

```javascript
{
  /* ... other collection attributes ... */
  filters: {
    name:   [ {"modifier": "prefix", "value":"a"  } ],
    size:   [ {"modifier": "gt",     "value": 1024} ],
    date:   null,
    access: null
    /* ... more filters ... */
  }
  /* ... other collection attributes ... */
}
```

```javascript
{
  /* ... other collection attributes ... */
  filters: {
    name: [
      {"modifier": "notlike", "value": "%.jpg"},
      {"modifier": "notlike", "value": "%.png"}
    ],
    size:   null,
    date:   null,
    access: null
    /* ... more filters ... */
  }
  /* ... other collection attributes ... */
}
```
#### Schema ####
In the schema, the value of each filter attribute SHOULD be an object describing:
  - <code>modifiers:</code> What modifiers are supported, if any
  - <code>options:</code> What specific values are allowed, if the attribute is enumerable

Modifiers allow things like "greater than" or "not equal to".  The standard modifiers are:

modifier               | meaning
-----------------------|---------------------
<code>"eq"</code>      | equal to<br/>(always available, the default if no modifier is specified)
<code>"ne"</code>      | not equal to
<code>"lt"</code>      | less than
<code>"lte"</code>     | less than or equal to
<code>"gt"</code>      | greater than
<code>"gte"</code>     | greater than or equal to
<code>"prefix"</code>  | starts with
<code>"like"</code>    | matches, as in SQL:<br/>&bull; underscore ("_") for exactly one character<br/>&bull; percent ("%") for 0 or more characters<br/>&bull; "\_" for one underscore character<br/>&bull;"\%" for one percent character.
<code>"notlike"</code> | does not match (as above)
<code>"null"</code>    | value is NULL
<code>"notnull"</code> | value is not NULL

Services MAY define (and document) their own modifiers.

```javascript
{
  /* ... other collection attributes ... */
  "filters": {
    "name":   {"modifiers": ["eq","ne","prefix","suffix"]},
    "date":   {"modifiers": ["eq","lt","gt","lte","gte","prefix"]}
    "size":   {"modifiers": ["eq","lt","gt","lte","gte"] }
    "access": {"modifiers": ["eq","ne"], "options": ["public","private","requirepassword"]}
    /* ... more filters ... */
  },
  /* ... other collection attributes ... */
}
```


----------------------------------------
# Read Operation #
The read operation allows a client to get a single resource.  Read operations MUST NOT incur side effects.

### Root Level ###
GETting the base URL of a service SHOULD return a collection of API version resources.  It SHOULD include a link to the <code>latest:</code> stable version of the API.

Versions that are not the latest MAY have a <code>deprecated: true</code> flag to indicate support for this version will be removed in the near future.

```javascript
{
  "type": "collection",
  "links": {
    "self":   "https://base/",
    "latest": "https://base/v3",
  },
  "data": [ 
    {
      "id":    "v1",
      "type":  "apiversion",
      "deprecated": true,
      "links": {
        "self": "https://base/v1",
        "file":    "https://base/v1/files",
        "folder":  "https://base/v1/folders"
        /* ... other collections and resources ... */
      }
    },
    {
      "id":    "v2",
      "type":  "apiversion",
      "deprecated": false,
      "links": {
        "self": "https://base/v2"
        "file":    "https://base/v2/files",
        "folder":  "https://base/v2/folders"
        /* ... other collections and resources ... */
      }
    },
    {
      "id":    "v3",
      "type":  "apiversion",
      "deprecated": false,
      "links": {
        "self": "https://base/v3",
        "file":    "https://base/v3/files",
        "folder":  "https://base/v3/folders"
        /* ... other collections and resources ... */
      }
    }
  ]
}
```

### Version Root ###
GETting the root URL of a particular API version SHOULD return a resource containing links to all the collections and resources that are available.  See [API Versioning](#api-versioning) for more info.

```http
GET /v1 HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "id": "v1",
  "type":  "apiversion",
  "deprecated": true,
  "links": {
    "self":    "https://base/v1",
    "file":    "https://base/v1/files",
    "folder":  "https://base/v1/folders"
    /* ... other collections and resources ... */
  },
}
```

### Individual Resource ###
GETting the URL of a resource SHOULD return the representation of that resource.

```http
GET /v1/files/b1b2e7006be HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "id": "b1b2e7006be",
  "type": "file",
  /* ... other resource attributes ... */
}
```

### Headers only ###
Making a HEAD request to a URL SHOULD return the same response code and header information as a GET of the same URL would, except that it MUST NOT return any message body.

```http
HEAD /v1/files/b1b2e7006be HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas
Content-Length: 234

```


```http
HEAD /v1/files/non-existent-id HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```
HTTP/1.1 404 Not Found
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas
Content-Length: 123

```

----------------------------------------
# Create Operation #
The create operation allows a client to create a new resource in a collection.  The URL of the request SHOULD NOT contain a query component, to differentiate a create request from an [action](#action-operation) request.

The body of the request MUST contain a representation of the resource to be created.  If any of the fields that are required by the schema for this resource are not included or are invalid, a 400 error SHOULD be returned, with information about what specifically is wrong.

If the created resource is immediately available, a 201 Created status code SHOULD be returned, and the body SHOULD be the representation of the new resource.  If the resource cannot be created and read immediately, a 202 Accepted status SHOULD be returned instead.  At least the resource id should be returned if at all possible.

### Single resource ###
For a single resource, the response SHOULD include a [Location](http://tools.ietf.org/html/rfc2616#section-14.30) header with the URL for the new resource. This is likely the same as the <code>self:</code> link in the resource.  Note that Location headers MUST always be an absolute URL.

```http
POST /v1/folders HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

{
  "name": "Documents"
}
```
```http
HTTP/1.1 201 Created
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas
Location: https://base/v1/folders/ab391827x31

{
  "id":   "ab391827x31",
  "type": "folder",
  /* ... more resource attributes ... */
}
```

### Single resource - form post ###
Services SHOULD also support create calls which are HTML Form encoded, <code>Content-Type: multipart/form-data</code> or <code>Content-Type: application/x-www-form-urlencoded</code>.  These are easy for a client to construct with a HTML form or cURL, and also allow for simple uploading of binary data/files.  A file management API might allow creating a file resource like:

```bash
curl -u access_key:secret_key \
  -F name="ultimate_answer.txt" \
  -F file="@mydatafile.txt" \
  "http://base/v1/folders/ab391827x31"
```

Which would make a request like:

```http
POST /v1/folders/ab391827x31 HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Length: 307
Content-Type: multipart/form-data; boundary=----------------------------9a41916dd20a

------------------------------9a41916dd20a
Content-Disposition: form-data; name="name"

ultimate_answer.txt
------------------------------9a41916dd20a
Content-Disposition: form-data; name="file"; filename="mydatafile.txt"
Content-Type: text/plain

42
------------------------------9a41916dd20a--
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas
Location: https://base/v1/files/b1b2e7006be

{
  "id":   "b1b2e7006be",
  "type": "file",
  /* ... more resource attributes ... */
}
```

### Multiple resources ###
Services MAY allow multiple resources to be created in one request by accepting an array of representations instead of a single one. 
  - A Location header SHOULD NOT be returned.
  - The response SHOULD be a collection containing the created resources.
  - The service MUST ensure that this operation is atomic:
    - Either ALL the creates succeed and a 2xx response code is returned, 
    - or NONE of the changes are committed and a 4xx or 5xx code is returned.
    - If this is not possible then the service SHOULD reject requests for multi-resource create with a 406 error.

```http
POST /v1/folders HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

[
  {"name": "Documents", /* ... more resource attributes ... */},
  {"name": "Pictures",  /*... more resource attributes ... */},
  /* ... more resource representations ... */
]
```
```http
HTTP/1.1 201 Created
X-API-Schemas: https://base/v1/schemas
Content-Type: application/json

{
  "type": "collection",
  "resourceType": "folder",
  "data": [
    {"name": "Documents", /* ... more resource attributes ... */},
    {"name": "Pictures",  /* ... more resource attributes ... */},
    /* ... more resource representations ... */
  ]
  /* ... more collection attributes ... */
}
```

----------------------------------------
# Update Operation #

The update operation allows a client to modify resources.  Update operations MUST be idempotent; making the same update request again should have no effect.
  - Any change which does not have this property is an [action](#action-operation), not an update.

Clients may submit just the attributes that they wish to change, or the entire representation of a resource.  The <code>id:</code>, and <code>rev:</code>(if [resource versioning](#resource-versioning) is enabled) attributes MUST always be included.

The response SHOULD include the entire updated resource representation(s), even if the request did not.  If the resource is a potentially large binary file instead of a JSON document, a 204 status and no body SHOULD be returned.

### Single resource ###
```http
PUT /v1/folders/b1b2e7006be HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

{
  "id":     "b1b2e7006be",
  "rev":    "8d2e54afd",
  "access": "private"
}
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "id":     "b1b2e7006be",
  "rev":    "8d2e54afd",
  "name":   "Documents",
  "access": "private"
}
```

### Multiple resources ###
When updating multiple resources, the service MUST ensure that the operation is atomic.
  - Either ALL the changes succeed and a 2xx response code is returned,
  - or NONE of the changes are committed and a 4xx or 5xx code is returned.
  - If this is not possible, do not allow multiple updates.
  - The service SHOULD document and limit the number of resources that may be updated in one request and return a 400 error if the limit is exceeded.

```http
PUT /v1/folders HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

[
  {
    "id":     "b1b2e7006be",
    "rev":    "8d2e54afd",
    "access": "private"
  },
  {
    "id":     "d5a80ee7",
    "type":   "folder", 
    "rev":    "9586f36z",
    "name":   "Pictures",
    "access": "public"
  }
]
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "type": "collection",
  "resourceType": "folder",
  "data": [
    {
      "id":     "b1b2e7006be",
      "type":   "folder",
      "rev":    "8d2e54afd",
      "name":   "Documents",
      "access": "private"
    },
    {
      "id":     "d5a80ee7",
      "type":   "folder", 
      "rev":    "9586f36z",
      "name":   "Pictures",
      "access": "public"
    }
  ]
  /* ... other collection attributes ... */
}
```

----------------------------------------
# Delete Operation #
The delete operation deletes the item given by the request URL.

### Single resource ###
```http
DELETE /v1/folders/b1b2e7006be HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 204 No Content

```

### Multiple Resources
When deleting multiple resources, the service MUST ensure that the operation is atomic.
  - Either ALL the deletes succeed and a 2xx response code is returned,
  - or NONE of the deletes are committed and a 4xx or 5xx code is returned.
  - If this is not possible, do not allow multiple deletes.
  - The service SHOULD document and limit the number of resources that may be deletes in one request and return a 400 error if the limit is exceeded.

```http
DELETE /v1/folders/ HTTP/1.1
Content-Type: text/json

[
  "b1b2e7006be",
  "another_folder_id",
  /* ... more resource IDs ... */
]
```
```http
HTTP/1.1 204 No Content

```

### All resources in a collection ###
If appropriate for your service, collections MAY support:
  - A "truncate" action to delete all the resources in the collection (but not the collection itself).
  - A "recursiveDelete" action to truncate and then delete the collection itself.

### Deleted Resources ###
Services MAY continue to return resources that have been recently deleted in [query](#query-operation) and [read](#read-operation) operations.  This is a [suggestion from RightScale](http://blog.rightscale.com/2010/02/13/top-cloud-api-sins/); It is easier for some use cases for the client to explicitly see that a resource has been removed than it is to infer that it is gone because it no longer comes back in query responses.

Deleted resources:
  - MUST have a <code>removed:</code> or similar attribute that the client can use to differentiate them from resources that still exist.
  - SHOULD support filtering on that attribute so clients that don't want them can get results that omit removed results.
  - SHOULD stop being returned in query results after a documented period of time on the order of a few hours to days.


----------------------------------------
# Replace Operation #
The replace operation is an atomic combination of the [query](#query-operation), [delete](#delete-operation), and [create](#create-operation) operations.
  - The request URL is treated as a query operation.
  - Any matching resources are deleted.
  - New resources described in the request body are created. 

Services MAY implement replace for any collections where they see fit.  It is most useful when updating a many-to-many relationship.  For example, associating a list of IP addresses to a firewall rule.  Updating this list would require a client UI to:
  - Perform a query to get the current associations.
  - Diff the result against what the new entries should be.
  - Issue delete operation(s) for the ones that need to be removed.
  - Issue create operation(s) for the ones that need to be added.
  - Somehow resolve being in an inconsistent state if an error occurs partway through this process.

Replace performs all this in one convenient and atomic action for the client.
  - The service MUST ensure that this entire operation is atomic.
  - Either ALL the changes succeed and a 2xx response code is returned,
  - or NONE of the changes are committed and a 4xx or 5xx code is returned.

The response SHOULD return the list of newly created resources, just as if the request were a normal create.

Note: this operation uses a non-standard HTTP method.  There's nothing wrong with that, but some clients may not support abitrary methods.

```
REPLACE /v1/firewalls/42/allow HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

[
  {"ip": "172.16.234.12"},
  {"ip": "10.1.2.3"}
]
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "type": "collection",
  "data": [
    {"id": "1234", "ip": "172.16.234.12"},
    {"id": "1235", "ip": "10.1.2.3"},
  }
  /* ... more collection attributes ... */
}
```


----------------------------------------
# Action Operation #
The action operation allows a client to manipulate a resource or collection in a way that cannot be done with any of the other standard operations.  The request MAY include a body with additional information (arguments), and the response MAY contain a response body with result info.

Actions MAY (and generally SHOULD) incur side effects.  The request URL for an operation MUST contain a query component, to distinguish it from a [create](#create-operation) operation.  The actions that may be possible for given a resource type are defined in its [schema](#schemas).  The <code>actions:</code> attribute in a resource specifies which actions are actually available for this particular resource and the URL to request to perform them.

```http
POST /v1/folders/b1b2e7006be?encrypt HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: application/json

{
  "password": "purple monkey dishwasher"
}
```
```http
HTTP/1.1 202 Accepted
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  /* ... updated representation of the resource ... */
}
```


----------------------------------------
# Nesting #
Resources and collections MAY be nested.  For example, folders might be associated to files, and files to a collection of tags:

```http
GET /v1/folders/d5a80ee7/files/b1b2e7006be/tags HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "type": "collection",
  "resourceType": "tag",
  "data": [
    {"type: "tag", /*... more tag attributes ... */},
    /* ... more tag resources ... */
  ]
  /* ... more collection attributes ... */
}
```

Or the content of the files stored might be exposed as a sub-resource:
```http
PUT /v1/files/b1b2e7006be/content HTTP/1.1
Accept: application/json
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5
Content-Type: text/plain
Content-Length: 2

42
```
```http
HTTP/1.1 204 No Content
Content-Type: application/json

```

```http
GET /v1/files/b1b2e7006be/content HTTP/1.1
Authorization: Basic YWNjZXNzX2tleTpzZWNyZXRfa2V5

```
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Conetent-Length: 2

42
```

----------------------------------------
# Resource Versioning #
Some (or all) resources MAY implement resource versioning.  Versioning allows for the client to perform multiple concurrent updates on a resource and resolve conflicts when they occur.

For resources that implement versioning:
  - The service MUST return resource version in responses as the <code>rev:</code> attribute in query and read operations.
  - The client MUST include the <code>rev:</code> attribute in update, patch, and action operations.
    - If the revision given does not match the current revision when the request is processed, the service MUST return a 409 error.
  - The service MUST change the <code>rev:</code> attribute whenever a field in the resource changes.

----------------------------------------
# Base URL and Versioning #
The base URL that users access your API with SHOULD be of the form:
<code>https://api{-optional-region-code}.{your product domain}/</code>

APIs SHOULD have a separate DNS record from the your web/app, so that the API can be scaled separately from the rest of your stack.  The name can always point to the web/app servers if this scale is not needed yet

All APIs that require authentication MUST be accessible to the public **only** over HTTPS.  Unauthenticated public APIs MAY be provided over HTTP, but SHOULD also offer HTTPS.

## API Versioning ##
APIs MUST support more than one version of their implementation.  Clients MUST specify the particular version they want, and the application MUST NOT make changes that are not backwards compatible to that version.

Breaking changes should be avoided when possible, but you are eventually going to have to make a breaking change so it is far better to have a way to handle this built in from the start.  The suggested format for the version string is the letter "v" followed by a single integer which increases by one for each revision.  The version SHOULD be treated as an opaque string to the client, so any other format MAY be used.

Versions resources  MAY also have a "revision" string attribute that changes on each release.  This can be useful to help debug issues caused by changes made within a version that are supposed to be compatible with each other.

Outdated versions SHOULD be deprecated and eventually removed from the service.  A request for a version that has been removed SHOULD return a 404 error.

### Listing ###
Clients MUST be able to make a GET request to the base URL (without a version fragment) to find out what versions are available.  This request SHOULD NOT require authentication.  Clients SHOULD check the "latest" link periodically and take action if they are no longer using the latest version.

See the [root level](#root-level) and [individual version](#individual-version) read operation for more info.

----------------------------------------
# Authentication #
Most APIs that do something useful will need some form of authentication to determine what user is making the request and validate that they should be allowed to make it.  A user is identified by a set of credentials called an API Key pair.

### API Keys ###
An API Key consists of a pair of strings called an **access key** and **secret key**.  These are randomly generated opaque strings assigned by the service to each API user.  The access key is analogous to a username and the secret key to a password.

### HTTP Basic ###
Services MUST support [HTTP Basic](http://tools.ietf.org/html/rfc2617#section-2) authentication.  In Basic auth, the client sends their access key and secret key in the Authorization header.  The service then reads these and validates the keys.

----------------------------------------
# Design Considerations #

### Identifiers ###
Resource Identifiers
  - MUST be unique among other resources of the same type.
  - MUST consist only of URL-safe characters.
  - SHOULD be globally unique, if practical.
  - SHOULD be a string.  Numbers require care to avoid eventual overflow.
  - MAY be represented as [base32](http://www.crockford.com/wrmg/base32.html) for enhanced readability and error resistance.
  - SHOULD NOT be an auto-incrementing ID from a database.
    - Exposing this value reveals information about the size and growth of the service, and easily allows attackers to guess other valid IDs.

In general something along the lines of a [GUID](http://www.ietf.org/rfc/rfc4122.txt) is best if there will be many of the resource or they are dynamically created.  A short human-readable string may be appropriate if there is a very limited set of possible resources.

### Naming Conventions ###
Several keywords are reserved by this standard and have specific meanings.  These words MUST ONLY be used for their documented purpose:
  - In resources: 
    - <code>id:</code>, <code>type:</code>, <code>rev:</code>
    - <code>links:</code>, <code>actions:</code>
  - In collections: 
    - (Everything reserved for resources)
    - <code>filters:</code>, <code>pagination:</code>,
    - <code>sort:</code>, <code>sortLinks:</code>,
    - <code>createTypes:</code>, <code>createDefaults:</code>
    - <code>data:</code>
  - In resource and collection <code>links:</code>:
    - <code>self:</code>
  - In query strings:
    - For pagination: <code>marker</code>, <code>limit</code>
    - For sorting: <code>sort</code>, <code>order</code>

Some additional guidelines:
  - Names and attribute keys should be a single word when practical.
  - Single-word collection, resource, attribute names SHOULD be all lowercase.
  - Multiple words SHOULD be interCaps (also known as camelCase), not dash-separated, under_scored, or TitleCase.
  - Resource names SHOULD be singular, not plural.
    - These appear in a [resources's](#resources) <code>type:</code> and [schema's](#schemas) <code>id:</code>.

### What to Link ###
Guidelines for creating links:
  - Every reference to the <code>id:</code> of another resource SHOULD have a corresponding link.
    - For example if a file resource has a <code>folderId:</code> field, there should be a <code>folder:</code> link.
  - Links to a single resource SHOULD be singular.
    - e.g. <code>"content": "https://base/v1/files/b1b2e7006be/content"</code>
  - Links to a collection SHOULD be plural.
    - e.g. <code>"files": "https://base/v1/folders/d5a80ee7/files"</code>
  - Limit your URL namespace as much as possible.  The less surface area you have exposed the less there is that might need to change later.
  - Path components and query parameter names SHOULD be short, meaningful words in all lowercase, easy for a human to read.
  - Services SHOULD NOT change the format or construction of URLs within an API version
    - In theory, everyone uses the discoverability features and follows links, so services may change URL formats at any time.
    - But some clients will inevitably ignore disoverability and hardcode paths into their code.
    - So if a URL needs to be changed, provide a 301/302 redirect or release a new [API version](#api-versioning).

### Canonical Links ###
Each resource SHOULD have a single canonical URL and the representation of that resource SHOULD NOT change depending on how the client got to it.
  
If a file resource is accessible at <code>https://base/v1/files/b1b2e7006be</code> and as a data item in a collection <code>https://base/v1/folders/d5a80ee7/files</code>, both representations SHOULD be exactly the same.
  - The links inside the resource SHOULD reflect the canonical URL.
  - This prevents the service from generating arbitrarily long link URLs that contain unnecessary history of how the client got to where they are.
  - It also makes it possible to compare 2 reperesentations and determine if they are in fact the same resource.

```http
GET /v1/files/b1b2e7006be HTTP/1.1
Accept: application/json

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "id":      "b1b2e7006be", 
  "type":    "file",
  "links":   { 
    "self":     "https://base/v1/files/b1b2e7006be",
    "schemas":  "https://base/v1/schemas",
    "content":  "https://base/v1/files/b1b2e7006be/content",
    "folder":   "https://base/v1/folders/19c3932wef",
    "public":   "http://documents.your-files.com/ultimate_answer.txt"
  },
  "actions": {
      "encrypt": "htps://base/v1/files/b1b2e7006be?encrypt"
    },
  "name":    "ultimate_answer.txt",
  /* ...  more attributes ... */
}
```

```http
GET /v1/folders/d5a80ee7/files HTTP/1.1
Accept: application/json

```
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-API-Schemas: https://base/v1/schemas

{
  "type": "collection",
  "links": {
    "self": "https://base/v1/folders/d5a80ee7/files",
    "schemas": "https://base/v1/schemas"
  }
  /* ... more attributes ... */
  "data": [
    { 
      "id":      "b1b2e7006be", 
      "type":    "file",
      /* the same JSON for file b1b2e7006be, as above */
    }
    /* ... more resources ... */
  ]
}
```

### Regions ###
Many services have resources that exist in multiple geographic regions.

To provide the easiest management for the user, APIs SHOULD provide a single unified endpoint that can manage resources in all regions.  This endpoint MAY have servers in each region and use geographic load balancing to direct the requests to the closest region.

In applications where centralized management is not possible, each region MAY have a separate endpoint with a region code identified in the base URL.

### Caching ###
Services SHOULD do everything they can to enable the clients to take advantage of caching as much as possible.

#### ETag ####
Read operations SHOULD return a HTTP [ETag](http://tools.ietf.org/html/rfc2616#section-14.19) header with an opaque value.  The opaque value MUST uniquely describe the entire resource, and change if any part of the resource is changed.  This can often be done as a hash of the serialized resource representation, e.g. convert the resource to a JSON string and return a sha-sum of the string.

Query operations SHOULD also return an ETag.  The opaque value MUST uniquely describe the entire result set and change if any of the resources or metadata are changed/added/removed.

Clients MAY send an [If-None-Match](http://tools.ietf.org/html/rfc2616#section-14.26) header along with their query or read operation.  The service SHOULD compute its response, and if that response has the same ETag value, return a 304 status instead of transmitting the entire response again.

#### Last-Modified ####
Whenver possible, services SHOULD keep a last modified date and return a [Last-Modified](http://tools.ietf.org/html/rfc2616#section-13.3) header on Read and Query operations.

Clients MAY send an [If-Modified-Since](http://tools.ietf.org/html/rfc2616#section-14.25) header.  The service SHOULD return a 304 status instead of transmitting the entire response again if it hasn't changed since the time given.

#### Cache-Control ####
Read and Query operations SHOULD return a [Cache-Control](http://tools.ietf.org/html/rfc2616#section-14.9) header with values that are appropriate for the information being returned.  If the data changes infrequently, allow clients to cache it for a reasonable period of time.

### Compression ###
API Services SHOULD support gzip and deflate compression and deliver responses with an appropriate [Content-Encoding](http://tools.ietf.org/html/rfc2616#section-14.11) header according to the [Accept-Encoding](http://tools.ietf.org/html/rfc2616#section-14.3) requested by the client.

### Query Strings ###
Many clients, frameworks, and web servers limit the length of query strings and the request URL as a whole.  Services SHOULD be designed so that clients never have any good reason to make a request for a URL longer than 2048 bytes.  Services MAY return a 413 error if a request URL is longer than they wish to handle.
