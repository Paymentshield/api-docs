# Catalogue Service

You can use the Catalogue Service to retrieve industry and occupation lists.  You can also retrieve lists of valid Branch Number and associated maximum commission sacrifice levels for a user.

## Industries


> Example industry request without filter

```http
GET https://apiuat.paymentshield.co.uk/industries HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

> Example industry request with filter

```http
GET https://apiuat.paymentshield.co.uk/industries/get?filter=fish HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Where a product _QuestionSet_ includes a question for industry you must send the correct ABI code.

You can use the Get Industries endpoint to return a list of valid industry codes and associated descriptions.

You can add a filter to the request.  

Please see the code pane for example requests.  Note that in the second request we will return all industries that include the word 'Fish' in the description.


## Occupations

> Example occupations request without filter

```http
GET https://apiuat.paymentshield.co.uk/occupations HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

> Example Occupations request with filter

```http
GET https://apiuat.paymentshield.co.uk/occupations/get?filter=carpet HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Where a product _QuestionSet_ includes a question for occupation you must send the correct ABI code.

You can use the Get Occupations endpoint to return a list of valid occupation codes.

You can add a filter to the request.  

Please see the code pane for example requests.  Note that in the second request we will return all occupations that include the word 'Carpet' in the description.


## Products

You can use the Get Products request to return a list of valid Branch Numbers and associated maximum commission sacrifice values for a specified userId.

> Example Products request

```http
GET https://apiuat.paymentshield.co.uk/products HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

### Products Response

 > Example Products Response

```json
[
    {
        "WebProductGroupId": 1008,
        "Branches": [
            {
                "BranchNo": "PS123456",
                "MaxSacrificeLevel": 0
            },
            {
                "BranchNo": "PS123457",
                "MaxSacrificeLevel": 25
            }
        ]
    },
    {
        "WebProductGroupId": 1013,
        "Branches": [
            {
                "BranchNo": "PS123456",
                "MaxSacrificeLevel": 0
            },
            {
                "BranchNo": "PS123457",
                "MaxSacrificeLevel": 25
            }
        ]
    }
]
```


You should use the GET Products request to return a list of products the UserId is authorised to sell and the Branch Numbers they can sell them under.  We will also return the `MaxSacrificeLevel` for the UserId/Branch Number combination.

Please see the Code Pane for an example of a Products Response.

Note that the example response shows that UserId 123456 is authorised to sell ProductIds 1008 (Home Insurance) and 1013 (Tenants Contents) under two Branch Numbers.  Under Branch Number PS123457 the user has a maximum commission sacrifice level of 25%


## SDN Templates

> Get SDN Templates for a submitted QuoteRequest

```http
GET https://apiuat.paymentshield.co.uk/sdntemplate/6543211 HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```


A Statement of Demands and Needs (SDN) document contains a number of statements which users can choose to agree are important to them. When creating an SDN using our API, it's your choice as to what free text goes into these demands and needs, but for reference and feature parity with Adviser Hub, you can use the `SdnTemplate` endpoint to get the default list of statements used by Paymentshield.

Make a `GET` request to `/SdnTemplate/{QuoteRequestId}`

We require `QuoteRequestId` to fetch the appropriate SDN statements for that product, and to make sure that the quote you are talking about has really been submitted, as this is the only state where SDN generation is valid.

### SdnTemplate Response

> Example SdnTemplate response

```json
{
    "StatusCode": "OK",
    "Messages": [],
    "WebProductGroupId": 1008,
    "Sections": [
        {
            "InterfaceKey": "YourDemandsAndNeeds",
            "Title": "YOUR DEMANDS AND NEEDS",
            "SubText": "Find insurance that:",
            "SortOrder": 1,
            "TemplateStatements": [
                {
                    "Text": "Allows you to utilise your no claims discount",
                    "SortOrder": 1
                },
                {
                    "Text": "Covers your home from loss or damage",
                    "SortOrder": 2
                },
				...
            ]
        },
        ...
    ]
}
```

An example response is listed in the code pane.

The response consists mainly of a list of `Sections`. The `InterfaceKey` corresponds to what you will send to the `POST /SDN/` endpoint, and the list of `TemplateStatements` are the texts used by Paymentshield. The `SortOrder` properties for this response are informational only and you can choose whether or not to use them. The `SortOrder` of the Sections is used on the final PDF document so it can be helpful to show users this order during document construction.


