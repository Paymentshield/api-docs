# Quote Service

You can use the Quote Service to get Quotes and QuickQuotes, retrieve those you created earlier, and continue an in-progress Quote. The Quote service relies upon some external services to PSL to get prices, which can mean it takes a while to get prices, particularly for complex quotes featuring multiple tiers of cover.

### Full vs Partial integration

We use the term 'full integration' to mean integrations which complete a quote and buy journey using only API endpoints. In contrast, a 'partial integration' gets a quote or quick quote from the API and then completes the journey in our web frontend (sometimes called *IOL* or *Adviser Hub*).

### QuoteRequests and Quotes

A **QuoteRequest** is a well-defined request for cover which we submit to one or more insurers. QuoteRequests have sequential integer IDs. If insurers can provide prices for the specified cover, we return these in the response and refer to each insurer offer as a **Quote**, which has a GUID (i.e. `12345678-1234-1234-1234-123456789abc`). A customer can apply for a Quote. 

## Create Quote

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
  "BranchNumber": "AB000000",
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

Create a new full or partial, complete or incomplete Quote Request. A *Quote Request* is our term for a well-specified request for insurance quotes, for a given customer, based on the customer information you provide. As long as your request provides a non-trivial amount of information, a Quote Request will be created in our database and we will pass back a `QuotesResponse` with a `QuoteRequestId`. From here, you can add more information later using **Update Quote** or you can continue the user journey in our web frontend using the `ContinuationUri` in the response.

Please see the code pane for an example of the HTTP headers and JSON request you should send.

You can find the types and examples of each request field in the Swagger definition of the API.

<aside class="notice">
To find out what values you should send in the <code>Answers</code> array, you can use the Catalogue API Questionset endpoint, or our <a href="#">Questionset browser</a>
</aside>

### ProductId

Not all integrators can sell the full range of products. If you want to be authorised to sell a different range of products, please [Get in touch][contact]

ProductId | Product
--------- | -------
1008      | Home - Buildings and Contents
1009      | Landlords - Buildings and Contents
1010      | Tenants Liability
1011      | Rent Protection
1012      | MPPI
1013      | Tenants Contents (2018)


### Other field notes

 + We recommend that `UseDefaults` is set to true. It sets some Answers, including calculated Answers (total coverage value for STIP/MPPI), to their sensible default or calculated value if you don't specify one.


## Retrieve Quote

 > Retrieve a set of Quotes by QuoteRequestId
 
```http
GET https://api.paymentshield.co.uk/v1/Quote/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Retrieve a fully-hydrated Quote Response from a previously created quote. This allows you to show your user up-to-date quote state and whatever values they filled in originally. You can use this feature to resume a journey. Because we store the full quote information associated with a `QuoteRequestId`, you don't have to store full quote information in your database; you can look it up with this operation.

Pass the integer `QuoteRequestId` from the original Quote Response.


## Create QuickQuote

 > Create a QuickQuote
 
```http
POST https://api.paymentshield.co.uk/v1/QuickQuote/ HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

```json
{
  "ProductId": 1008,
  "BranchNumber": "AB000000",
  "UseDefaults": true,
  "IsIndicativeQuote": false,
  "HasAssumedAnswers": false,
  "CommissionSacrifice": 0.0,
  "Answers": [
    {
      "Value": "Mr",
      "InterfaceKey": "Applicant1Title"
    },
    {
      "Value": "Carrie",
      "InterfaceKey": "Applicant1Forename"
    },
    {
      "Value": "Palmer",
      "InterfaceKey": "Applicant1Surname"
    },
    {
      "Value": "29/01/1993",
      "InterfaceKey": "Applicant1DoB"
    },
    ...
    {
      "Value": "93 Normal Street",
      "InterfaceKey": "InsuredAddressStreet"
    },
    {
      "Value": "Birkdale",
      "InterfaceKey": "InsuredAddressDistrict"
    },
    {
      "Value": "Southport",
      "InterfaceKey": "InsuredAddressTown"
    },
    {
      "Value": "Merseyside",
      "InterfaceKey": "InsuredAddressCounty"
    }
  ]
}
```

A QuickQuote is an indicative price which you can get by submitting less information about the customer. You can allow your customer to pick up their QuickQuote journey in our frontend using the `QuickQuoteContinuationUri`. In the next phase of our API, you will be able to promote it into a full quote.

Today, QuickQuote is only implemented for Home insurance (product 1008).

<aside class="warning">
You can't apply for QuickQuote quotes because they don't have enough information to give a firm price.
</aside>

You must supply the following answers as a minimum:

 * Applicant1Title
 * Applicant1Forename
 * Applicant1Surname
 * Applicant1DoB
 * NoClaimsBuildings
 * NoClaimsContents
 * PropertyType
 * NumberOfBedrooms
 * BuildYear
 * InsuredAddressPostCode
 * InsuredAddressStreet
 * InsuredAddressDistrict
 * InsuredAddressTown
 * InsuredAddressCounty

### QuickQuotesResponse

We deliver a `QuickQuotesResponse` in response to both the 'Create' and 'Get' QuickQuote operations. This object has some differences from a normal QuotesResponse:

 + It has a `QuickQuoteContinuationUri` instead of `QuoteSessionContinuationUri`
 + Only the cheapest Price is returned, instead of many prices


## Retrieve QuickQuote

> Retrieve a QuickQuote the same way you retrieve a Quote

```http
GET https://api.paymentshield.co.uk/v1/Quote/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

You can retrieve a QuickQuote the same way you would [Retrieve a quote](#retrieve-quote), passing the integer `QuoteRequestId` from the original QuickQuote Response.

If the user has progressed a QuickQuote by adding information, so that it now has many prices, then we return a full `QuotesResponse`.





