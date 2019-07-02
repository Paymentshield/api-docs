---
title: Paymentshield REST API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - http
  - javascript
  - cs

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/paymentshield/api-docs'>Contribute</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to our API! We have built our new REST API with you in mind, to make it easier for integrators to quote, sell, and deliver Paymentshield insurance policies in a secure and accessible way.

Our API is composed of a number of services separated by function and hosted in isolation, but you can access them all through one easy URL scheme.

Our APIs to date are:

 * Security Service
 * Quote Service
 * Documents Service
 * Catalogue Service


# Register to use the API

To integrate with us and start exploring and selling PSL policies, please visit our [Developer portal][developerportal] and complete the signup form to receive your `SystemId` and Test User credentials.



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


# Authentication

We have a custom authentication flow which is enforced by our backend system, rather than using OAuth or similar.

The general flow of authentication is like this:

 + Authenticate with our `SecurityService` by sending either `email`+`password` or `userid`+`passkey` in a JSON body
 + We'll send/echo back your `UserId` (a constant, unique integer ID) and a session `Token` (GUID) which is valid for 40 minutes. The Token's validity is reset to the full time whenever it is used for an authenticated API call. You can store the `Token` in local storage within your app.
 + You should know your `SystemId`, a GUID identifying the piece of software calling the API. This is somewhat like an API key (with the `UserId` and `Token` together forming a kind of API secret).
 + In all subsequent requests to any of our API services, send the `UserId`, `Token`, and `SystemId` in **HTTP headers**.

```http
POST https://api.paymentshield.co.uk/v1/security/login HTTP/1.1
Content-Type: application/json
{
    "Email": "broker@example.com", 
    "Password": "BrokerPassword"
}
```

```javascript
// TODO: JavaScript sample with Fetch API
```

```cs
// TODO: C# Sample with WebClient
```

> Replace the `Email` and `Password` fields with your credentials

# Error Handling

This is how we use HTTP status codes:

HTTP status | Meaning
----------- | -------
400         | Something was incorrect or missing in your request object. More information will be given in the `StatusCode` and `Messages` elements of the response.
401         | Unauthenticated - Your request didn't appear to be from a valid user. Maybe the credentials were missing or wrong, or don't work with the SystemId provided, or perhaps your Session has expired.
403         | Unauthorized - Your credentials are valid but the specified user isn't allowed to do the task you tried to do. This might be because of a product restriction or because you are trying to look at another users' data.
500         | Server Error - Generally a failure on our part or an impossible-to-fulfil request.

In many cases, we return additional information in the JSON response body in the `StatusCode` and `Messages` parameters.

~~~json
{
    "StatusCode": "OK",
    "Messages":
	[
		{
			// Message structure...
		}
	]
}
~~~

# Services and Resources

## Quote Service

You can use the Quote Service to get Quotes and QuickQuotes, retrieve those you created earlier, and continue an in-progress Quote. The Quote service relies upon some external services to PSL to get prices, which can mean it takes a while to get prices, particularly for complex quotes featuring multiple tiers of cover.

### Create Quote

Create a new full or partial, complete or incomplete Quote Request. A *Quote Request* is our term for a well-specified request for insurance quotes, for a given customer, based on the customer information you provide. As long as your request provides a non-trivial amount of information, a Quote Request will be created in our database and we will pass back a `QuotesResponse` with a `QuoteRequestId`. From here, you can add more information later using **Update Quote** or you can continue the user journey in our web frontend using the `ContinuationUri` in the response.

Please see the code pane for an example of the HTTP headers and JSON request you should send.

<aside class="notice">
To find out what values you should send in the <code>Answers</code> array, you can use the Catalogue API Questionset endpoint, or our <a href="#">Questionset browser</a>
</aside>

#### ProductId

Not all integrators can sell the full range of products. If you want to be authorised to sell a different range of products, please [Get in touch][contact]

ProductId | Product
--------- | -------
1008      | Home - Buildings and Contents
1009      | Landlords - Buildings and Contents
1013      | Tenants Contents

```http
POST https://api.paymentshield.co.uk/v1/Quote/ HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

```json
{
  "ProductId": 1008,
  "BranchNumber": "PS000000",
  "UseDefaults": true,
  "IsIndicativeQuote": false,
  "HasAssumedAnswers": false,
  "CommissionSacrifice": 0,
  "Answers": [
    {
      "Value": "Mrs",
      "InterfaceKey": "Applicant1Title"
    },
    {
      "Value": "Kayleigh",
      "InterfaceKey": "Applicant1Forename"
    },
    {
      "Value": "Porter",
      "InterfaceKey": "Applicant1Surname"
    },
    ...
    {
      "Value": "One",
      "InterfaceKey": "NumberOfBedrooms"
    },
    {
      "Value": "Flat",
      "InterfaceKey": "PropertyType"
    },
    ...
  ]
}
```

[contact]: https://paymentshield.co.uk
[developerportal]: https://paymentshield.co.uk