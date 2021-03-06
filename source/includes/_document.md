# Document Service

You can use the Document Service to get a list of the Documents available for a customer's Quote (or QuoteRequest), and to retrieve the individual documents themselves. Documents are always PDFs but you can choose to receive them as either a Base64 stream or raw PDF bytes (with an `application/pdf` content-type).

Use `GET` when requesting lists and documents from the Document Service.

## Get Documents List

 > Get a list of Documents by QuoteRequestId
 
```http
GET https://apiuat.paymentshield.co.uk/Documents/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

 > Get a list of Documents by QuoteId
 
```http
GET https://apiuat.paymentshield.co.uk/Documents/?quoteId={QuoteId:GUID} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Returns a list of documents for the given `QuoteRequestId` or `QuoteId`

> Example response

```json
{
  "StatusCode": "OK",
  "Messages": [],
  "Documents": [
    {
      "QuoteRequestId": 633360,
      "Name": "Policy Booklet",
      "Code": "terms",
      "Type": "static",
      "Link": "https://apiuat.paymentshield.co.uk/Document/terms/633360.pdf"
    },
    {
      "QuoteRequestId": 633360,
      "Name": "Insurance Product Information Document",
      "Code": "ipid",
      "Type": "static",
      "Link": "https://apiuat.paymentshield.co.uk/Document/ipid/633360.pdf"
    },
    {
      "QuoteRequestId": 633360,
      "Name": "All Quotes Summary",
      "Code": "multiquote",
      "Type": "dynamic",
      "Criteria": {
        "BuildingsCoverLevel": 500000,
        "ContentsCoverLevel": 50000,
        "BuildingsAD": "Excluded",
        "ContentsAD": "Excluded"
      },
      "Link": "https://apiuat.paymentshield.co.uk/Document/multiquote/633360.pdf?adb=excluded&adc=excluded&bcl=500000&ccl=50000"
    },
    ...
    {
      "QuoteRequestId": 633360,
      "Name": "All Quotes Summary",
      "Code": "multiquote",
      "Type": "dynamic",
      "Criteria": {
        "BuildingsCoverLevel": 1000000,
        "ContentsCoverLevel": 75000,
        "BuildingsAD": "Included",
        "ContentsAD": "Included"
      },
      "Link": "https://apiuat.paymentshield.co.uk/Document/multiquote/633360.pdf?adb=included&adc=included&bcl=1000000&ccl=75000"
    },
    {
      "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
      "QuoteRequestId": 633360,
      "Name": "Quote and Application Summary",
      "Code": "qas",
      "Type": "dynamic",
      "Link": "https://apiuat.paymentshield.co.uk/Document/qas/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
    },
    {
      "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
      "QuoteRequestId": 633360,
      "Name": "Statement of Demands and Needs",
      "Code": "sdn",
      "Type": "dynamic",
      "Link": "https://apiuat.paymentshield.co.uk/Document/sdn/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
    },
    {
      "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
      "QuoteRequestId": 633360,
      "Name": "Quote Summary",
      "Code": "quote",
      "Type": "dynamic",
      "Criteria": {
        "PaymentFrequency": "monthly"
      },
      "Link": "https://apiuat.paymentshield.co.uk/Document/quote/monthly/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
    },
    {
      "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
      "QuoteRequestId": 633360,
      "Name": "Quote Summary",
      "Code": "quote",
      "Type": "dynamic",
      "Criteria": {
        "PaymentFrequency": "annual"
      },
      "Link": "https://apiuat.paymentshield.co.uk/Document/quote/annual/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
    },
    ...
    {
      "QuoteId": "b6cfed8a-b9ce-4035-98a8-f5d0da449c3e",
      "QuoteRequestId": 633360,
      "Name": "Quote Summary",
      "Code": "quote",
      "Type": "dynamic",
      "Criteria": {
        "PaymentFrequency": "annual"
      },
      "Link": "https://apiuat.paymentshield.co.uk/Document/quote/annual/b6cfed8a-b9ce-4035-98a8-f5d0da449c3e.pdf"
    }
  ]
}
```

To get the documents list by QuoteRequestId, make a GET request to `/Documents/{QuoteRequestId}`, like `/Documents/4812345`. 

A quote request has many quotes. To get the documents for an individual quote, make a GET request to `/Documents/?quoteId={QuoteId}` where the `QuoteId` is the long GUID of a quote, like `/Documents/?quoteId=ebff7102-0fb7-4856-b8ae-caf208834bdf`

### Response fields

Like all other response types from our API, there will be `StatusCode` and `Messages` array at the top of the response.

Within each entry in the documents list:

 * There is always a `QuoteRequestId` 
 * There may be a `QuoteId` if this document is specific to a quote
 * There is always a `Name` which is a user-facing description of the document
 * There is always a `Code` which is a predictable machine-readable code identifying the document type (see table below)
 * There is always a `Type` field which is a predictable machine-readable string to let you know if this is a `static` document (always the same), or `dynamic` (generated from user selections)
 * There may be a `Criteria` block which allows you to filter down a set of documents of the same Code based on their payment type or coverage
 * There is always a `Link` which is the hyperlink reference to download the document itself
 
### Document Codes

The valid options for document `Code` are listed below:

Code     | Significance and other names
-------- | ----------------------------
quote       | Quote Summary (Price Summary)
ipid        | Insurance Product Information Document, or IPID (Policy Summary)
terms       | Policy Booklet (Terms and Conditions)
sdn         | Statement of Demands and Needs.  Only available when the `QuoteStatus` is `SUBMITTED`
qas         | Quote and Application Summary.  Only available when the `QuoteStatus` is `SUBMITTED`
cofc        | Confirmation of Cover. Tenants Contents only, available when the `QuoteStatus` is `SUBMITTED`
multiquote  | All Quotes Summary (Multiple Price Summary)


### Differences in retrieving docs by QuoteRequest or Quote

A QuoteRequest may have many quotes.  This means there could be more entries of certain document types when you get the list for an entire `QuoteRequest` compared to that for an individual `QuoteId`. 
This is summarised in the table below:

Document | For QuoteRequest         | For Quote
-------- | ------------------------ | --------
IPID       | Always one entry                                        | Always one entry
Terms      | Always one entry                                        | Always one entry
Quote      | One entry for each **QuoteId** in the QuotesResponse    | One entry for the specified quote
Multiquote | One entry, aggregating all the qualifying quotes        | None
QAS        | One, for the applied-for quote                          | One, for the applied-for quote
SDN        | One, after applying for a quote                         | One, after applying for a quote
COfC       | One, after applying for a quote                         | One, after applying for a quote


## Get Document

Your primary way to discover Get Document links is to follow those returned in a Documents list response.

```http
GET https://apiuat.paymentshield.co.uk/Document/quote/monthly/37b322f5-8240-48a5-b362-000bddb544b3.pdf HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

You can use the `Link` returned for each document to retrieve the document as a PDF, which will load with `content-disposition:attachment`.  You can request the document as a base64 string by removing **.pdf** from the Link.

You can also construct a Get Document request string if you know the `QuoteRequestId` or `QuoteId` and you know which document you want to retrieve:


Document  | Request string
--------- | ----------------------------
IPID      | https://apiuat.paymentshield.co.uk/Document/ipid/{QuoteRequestId}
Terms     | https://apiuat.paymentshield.co.uk/Document/terms/{QuoteRequestId}
Quote     | https://apiuat.paymentshield.co.uk/Document/quote/{PaymentFrequency}/{QuoteId}
QAS       | https://apiuat.paymentshield.co.uk/Document/qas/{QuoteId}
SDN       | https://apiuat.paymentshield.co.uk/Document/sdn/{QuoteId}
COfC      | https://apiuat.paymentshield.co.uk/Document/cofc/{QuoteId}

### PaymentFrequency

Please note that there are payment frequencies named `Monthly2MonthsFree` and `Monthly3MonthsDeferred`. One is a 10-month payment plan, and the other is a 9-month payment plan. "Deferred" is the correct language as the total amount paid is still the same as if it were paid in 12 months. The word 'Free' is retained due to internal implementation, but both should be considered as "deferred" payment plans.


## Build SDN

> Build an SDN

```http
POST https://apiuat.paymentshield.co.uk/SDN/4786184 HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8

{
  "Sections" : [
    {
      "InterfaceKey" : "YourDemandsAndNeeds",
      "Statements" : [
        "Allows you to utilise your no claims discount",
        "Covers your home from loss or damage"
        ]
    },
    {
      "InterfaceKey" : "SuitabilityReasons",
      "Statements" : [
        "Is competitively priced",
        "Includes Personal Possessions cover"
        ]
    }
  ]
}
```

A Statement of Demands and Needs (SDN) document contains a number of statements about the suitability of the product to the customer.

If your API login is an Advised user, you must build SDN documents by selecting statements appropriate for the customer before you can download the document.

When creating an SDN using our API, it's your choice as to what free text goes into these demands and needs, but for reference and feature parity with Adviser Hub, you can [use the `/Catalogue/SdnTemplate` endpoint](/#sdn-templates) to get the default list of statements used by Paymentshield.

POST to `/SDN/{QuoteRequestId}` with a list of `Sections` as shown in the code pane example. The `InterfaceKey` elements must match with those from the `SdnTemplate` request for the product. Looking at the SdnTemplate response will also help you as the developer to understand what sentences are being finished by the bullets you submit. I.E. "YourDemandsAndNeeds" statements complete the sentence beginning "Find insurance that:..."

The `Statements` element is a simple array of strings, and you can submit any strings here that accurately describe your customers needs.

<aside class="notice">
You may submit up to 20 statements per section, and up to 500 characters per statement. Please be aware that because of internal processing rules, your Statements <strong>should not</strong> include the characters '-', ';', carriage return or line feed (<code>\r</code> and <code>\n</code>) as these will be treated as delimiters and will create extra statements.
</aside>

If you POST more than once to the SDN for a QuoteRequestId, it will have act as a complete overwrite of all statements in all sections, as though the most recent POST were the only one you submitted.

After you have built the SDN with this function, you will be able to see it in the `GET /Documents/` list and will be able to download it from `GET /Document/SDN/{QuoteRequestId}`.