# Document Service

You can use the Document Service to get a list of the Documents available for a customer's Quote (or QuoteRequest), and to retrieve the individual documents themselves. Documents are always PDFs but you can choose to receive them as either a Base64 stream or raw PDF bytes (with an `application/pdf` content-type).

## Get Documents List

 > Get a list of Documents by QuoteRequestId
 
```http
GET https://api.paymentshield.co.uk/Documents/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

 > Get a list of Documents by QuoteId
 
```http
GET https://api.paymentshield.co.uk/Documents/{QuoteId:GUID} HTTP/1.1
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
            "Link": "https://api.paymentshield.co.uk/Document/terms/633360.pdf"
        },
        {
            "QuoteRequestId": 633360,
            "Name": "Insurance Product Information Document",
            "Code": "ipid",
            "Type": "static",
            "Link": "https://api.paymentshield.co.uk/Document/ipid/633360.pdf"
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
            "Link": "https://api.paymentshield.co.uk/Document/multiquote/633360.pdf?adb=excluded&adc=excluded&bcl=500000&ccl=50000"
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
            "Link": "https://api.paymentshield.co.uk/Document/multiquote/633360.pdf?adb=included&adc=included&bcl=1000000&ccl=75000"
        },
        {
            "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
            "QuoteRequestId": 633360,
            "Name": "Quote and Application Summary",
            "Code": "qas",
            "Type": "dynamic",
            "Link": "https://api.paymentshield.co.uk/Document/qas/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
        },
        {
            "QuoteId": "37b322f5-8240-48a5-b362-000bddb544b3",
            "QuoteRequestId": 633360,
            "Name": "Statement of Demands and Needs",
            "Code": "sdn",
            "Type": "dynamic",
            "Link": "https://api.paymentshield.co.uk/Document/sdn/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
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
            "Link": "https://api.paymentshield.co.uk/Document/quote/monthly/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
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
            "Link": "https://api.paymentshield.co.uk/Document/quote/annual/37b322f5-8240-48a5-b362-000bddb544b3.pdf"
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
            "Link": "https://api.paymentshield.co.uk/Document/quote/annual/b6cfed8a-b9ce-4035-98a8-f5d0da449c3e.pdf"
        }
    ]
}
```

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



## Get Document

Your primary way to discover Get Document links is to follow those returned in a Documents list response.

```http
GET https://api.paymentshield.co.uk/Document/quote/monthly/37b322f5-8240-48a5-b362-000bddb544b3.pdf HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```
You can use the `Link` returned for each document to retrieve the document as pdf.  You can request the document as a base64 string by removing **.pdf** from the Link.

You can also construct a Get Document request string if the Quote Request or QuoteId is known and you know which document you want to retrieve:


Document     | Request string
-------- | ----------------------------
IPID       | https://api.paymentshield.co.uk/Document/ipid/{QuoteRequestId}
Terms      | https://api.paymentshield.co.uk/Document/terms/{QuoteRequestId}
Quote      | https://api.paymentshield.co.uk/Document/quote/{PaymentFrequency}/{QuoteId}
QAS        | https://api.paymentshield.co.uk/Document/qas/{QuoteId}
SDN        | https://api.paymentshield.co.uk/Document/sdn/{QuoteId}



