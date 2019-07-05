# Document Service

You can use the Document Service to get a list of the Documents available for a customer's Quote (or QuoteRequest), and to retrieve the individual documents themselves. Documents are always PDFs but you can choose to retrieve them as either a Base64 stream or raw PDF bytes (with an `application/pdf` content-type).

## Get Documents List

 > By QuoteRequestId
 
```http
GET https://api.paymentshield.co.uk/v1/Documents/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

 > By Quote Id
 
```http
GET https://api.paymentshield.co.uk/v1/Documents/{QuoteId:GUID} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Returns a list of documents for the given `QuoteRequestId` or `QuoteId`


## Get Document

Docs coming soon.