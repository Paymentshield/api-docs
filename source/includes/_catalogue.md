# Catalogue Service

You can use the Catalogue Service to retrieve industry and occupation lists.  You can also retrieve lists of valid Branch Number and associated maximum commission sacrifice levels for a user.

## Industries


> Example industry request without filter

```http
GET http://apiuat.paymentshield.co.uk/industries HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

> Example industry request with filter

```http
GET http://apiuat.paymentshield.co.uk/industries/get?filter=fish HTTP/1.1
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
GET http://apiuat.paymentshield.co.uk/occupations HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

> Example Occupations request with filter

```http
GET http://apiuat.paymentshield.co.uk/occupations/get?filter=carpet HTTP/1.1
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
GET http://apiuat.paymentshield.co.uk/products HTTP/1.1
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



