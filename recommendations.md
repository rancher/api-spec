# Situations #

Situations fall outside of the technical specification but are listed to provide guidance for edge cases while remaining within the specification.  This document is not meant to be an all-inclusive list but rather provide a way to use the specification to accommodate specific needs.

----------------------------------------

# Table of Contents #

- [Errors](#errors)
  - [Client Error Messages](#client-error-messages)
  - [Multipart Errors](#multipart-errors)
- [Processing](#processing)
  - [Batch Processing](#batch-processing)
- [Usability](#usability)
  - [Alternative Formats](#alternative-formats)
  - [Clients with Limitations](#clients-with-limitations)

----------------------------------------
# Errors #

## Client Error Messages ##
Error messages are for developer use only and are not intended to be displayed to an end user to prevent accidental disclosure.  However, in the event that an error message is the preferred delivery of localized strings to an end user (as is often the case in mobile applications) the server MAY choose to localize the error to be passed onto the user.

If a service chooses to localize the error it MUST accept a language identifier in the request as the <code>Accept-Language</code> header value is reserved for the service to use.  The RECOMMENDATION is to use the language identifier provided to the service in the original request to localize a <code>userMessage</code> attribute and store the value witin the <code>userLocale</code> attribute.  A service MUST provide a <code>userLocale</code> if it provides a <code>userMessage</code> to allow the client to verify the language of the request with the response before providing it to the user.  If the service is unwilling to perform this localization service these fields should be omitted in the response.

The client MUST verify the language identifier value stored in <code>userLocale</code> with the requested identifier before displaying the message.  The server MAY not support the requested language and negotiate to an alternative format that differs from the request.

## Multipart Errors ##
Often a service will receive a request containing multiple parts that prevent the request from succeeding; which is a common case with creating a user.  If a service decides to offer this support it is RECOMMENDED that a single error be returned identifying the failure including individual parameters within the <code>details</code> attribute as an array of <code>errorFields</code> attributes.  Any error field MAY contain a <code>userMessage</code> attribute that will be localized with the <code>userLocale</code> in the error.  For example:

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

# Processing #

## Batch Processing ##
A special type of asynchronous interaction, batch processing, involves a client sending multiple actions in a single request for the service to process and respond with results.  These interactions follow the asynchronous behavior outlined within the [specification](./specification.md#asynchronous-actions) but the service must decide how to respond in the event of an error.  

Each batch service is unique and a recommendation cannot be easily made.  Services MUST describe in documentation how batches are processed.

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
