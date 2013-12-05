# Situations #

Situations fall outside of the technical specification but are listed to provide guidance for edge cases while remaining within the specification.  This document is not meant to be an all-inclusive list but rather provide a way to use the specification to accommodate specific needs.

----------------------------------------

# Errors #

## Client Error Messages ##
Error messages are for developer use only and are not intended to be displayed to an end user to prevent accidental disclosure.  However, in the event that an error message is the preferred delivery of messages to an end user (as is often the case in mobile applications) the server MAY choose to localize the error.  Services MUST use the value provided by <code>Accept-Language</code> in the original request to localize the <code>userMessage</code> and store the language in <code>userLocale</code>.  A service MUST provide a <code>userLocale</code> if it provides a <code>userMessage</code>.

The client MUST verify the value of <code>userLocale</code> and the value of the <code>Accept-Language</code> header before displaying the message as the server MAY NOT support the requested language and negotiated to an alternative format.

## Multipart Errors ##
Often a service will receive a request that contains multiple parts that may prevent the request from succeeding which is commonly the case with creating a user.  If a service decides to offer this support it is RECOMMENDED that a single error be returned identifying the failure and that individual parameters are enumerated within the <code>details</code> field as an array of <code>errorFields</code>.  Any error field MAY contain <code>userMessage</code> that will be localized with the <code>userLocale</code> in the error.  For example:

```http
POST /v1/user HTTP/1.1
Accept: application/json
Accept-Language: es-mx

{
  "username": "jsmith",
  "password": "******"
}
```

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json
Content-Language: en-us
X-API-Schemas: https://base/v1/schemas

{
  "type":    "error",
  "status":  400,
  "code":    "UserCreationError",
  "message": "The user could not be created.",
  "userMessage": "El usuario no pudo ser creada.",
  "userLocale": "es-mx", 
  "details": {
    "errorFields": [
      { "field": "username", "message": "Username is already taken.", "userMessage": "Nombre de usuario ya está en uso." },
      { "field": "password", "message": "Password must contain at least one number.", "userMessage": "La contraseña debe contener al menos un número." },
    ]
  }
}
```
----------------------------------------

# Usability 

## Alternative Formats ##
Often a service will expose a piece of data that may have multiple representations such as retrieving a domain name in its ASCII (Punycode) or Unicode representation.  Alternative formats may be specified using the links section of the [Resources](./specification.md#resources).

It is RECOMMENDED to allow a client to add a query string identifying the behavior desired for the field in question.  For example, if a service returned a domain name in a <code>domain</code> field, the client should be allowed to specify <code>?domain_format=xxxxxx</code>.  Where xxxxx is the identifier of the representation requested.  

The service MUST default to a logical choice if no presentation is requested and MUST result in an error if a requested representation is invalid or not possible.  

## Clients with Limitations ##
In some cases, clients may be limited by HTTP verbs or status codes requiring special care to enable interoperability.  Clients may provide query string parameters to override standard behavior:

  - <code>suppress_response_code</code> is reserved for clients that override error-handling, such as Flash.  When this flag is present in the query string the service must always respond with HTTP status 200. Special care must be made that the HTTP Status Code remains in the response body.
  - The <code>_method</code> parameter is reserved to allow clients that are unable to specify the HTTP method.  Use of this override has unintentional consequences with automated services and is not recommended for use at this time.

For example, if a client requires that the HTTP verb and status code be overridden they would do so with <code>/v1/files?suppress_response_code=true&method=POST</code>.

This behavior is not included by default as it is not an expected use case.  The custom fields are reserved and clients will support this behavior when requested.
