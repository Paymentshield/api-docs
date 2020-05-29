---
title: Paymentshield REST API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - http
  - php

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/paymentshield/api-docs'>Contribute</a>

includes:
  - history
  - security
  - quote
  - document
  - catalogue

search: true
---

# Paymentshield REST API Reference

Welcome to our API! We have built our new REST API with you in mind, to make it easier for integrators to quote, sell, and deliver Paymentshield insurance policies in a secure and accessible way.

Our API is composed of a number of services separated by function and hosted in isolation, but you can access them all through one easy URL scheme.

Our APIs to date are:

 * Security Service
 * Quote Service
 * Documents Service
 * Catalogue Service


# Environments

The code samples on this page use the UAT environment, which you should use for integration.
When you go to production, use the Live environment, which will create real quotes and policies in our live system.

Environment | Base URL
----------- | -------
UAT         | https://apiuat.paymentshield.co.uk
Live        | https://api.paymentshield.co.uk


# General API Principles

 + All message bodies (requests and responses) are raw JSON: set `content-type` header to `application/json`
 + We currently use verbs `GET` and `POST`:
     - `GET` is idempotent, with no side-effects.
	 - `POST` is used for functions which have an effect on system and data state.
 + Responses have an informative HTTP Status:
     - 200 OK
     - 201 Created
     - 400 Bad Request
     - 401 Unauthenticated
     - 403 Forbidden
     - 500 Server Error
 + Response bodies have an additional `StatusCode` field to give fine-grained error information
 + Response bodies will have an array of `Messages` which is populated if there are problems with the data you send us
 + Swagger (OpenAPI) definitions are available for each service


# Error Handling

~~~json
{
    "StatusCode": "ClientError",
    "Messages":
	[
		{
            "MessageType": "Authentication",
            "Summary": "Sorry, we couldn’t log you in with those details. Please check and try again"
		}
	]
}
~~~

This is how we use HTTP status codes:

HTTP  | Meaning
----- | -------
400   | Something was incorrect or missing in your request object. More information will be given in the `StatusCode` and `Messages` elements of the response.
401   | Unauthenticated - Your request didn't appear to be from a valid user. Maybe the credentials were missing or wrong, or don't work with the SystemId provided, or perhaps your Session has expired.
403   | Unauthorized - Your credentials are valid but the specified user isn't allowed to do the task you tried to do. This might be because of a product restriction or because you are trying to look at another users' data.
500   | Server Error - Generally a failure on our part or an impossible-to-fulfil request.

In many cases, we return additional information in the JSON response body in the `StatusCode` and `Messages` parameters.

### StatusCode

StatusCode            | Meaning
--------------------- | -------
EmptyRequest          | We couldn't see or understand your JSON Body data
ServerError           | A problem with our service
ClientError           | A problem with your request
AuthenticationRefused | `UserId` and `Token` (or `SystemId`) are wrong, or the session expired
UserLockedOut         | The supplied `UserId` has been locked out by an administrator
AuthorisationRefused  | The supplied user isn't allowed to complete the specified action
NotAllowed            | The supplied user isn't allowed to complete the specified action

### MessageType

MessageType    | Meaning
-------------- | -------
Authentication | Problem with the authentication details (headers) provided
Mandatory      | Problem with an Answer which is mandatory and for which you didn't provide a value
Invalid        | Problem with an Answer with validation criteria which weren't met
Quote          | Problem with quote retrieval or insurability of the submitted data
Error          | Problem with our service or a third party service dependency
ResultsLimited | Warning that there were too many results to show (for [Search Quotes](#search-quotes))
Generic        | Fallback error designation


[contact]: https://paymentshield.co.uk
[developerportal]: https://paymentshield.co.uk
