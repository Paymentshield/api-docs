# Quote Service

You can use the Quote Service to get Quotes and QuickQuotes and retrieve those you created earlier. The Quote service relies upon some external services to PSL to get prices, which can mean it takes a while to get prices, particularly for complex quotes featuring multiple tiers of cover.

### Full vs Partial integration

We use the term 'full integration' to mean integrations which complete a quote and buy journey using only API endpoints. In contrast, a 'partial integration' gets a quote or quick quote from the API and then uses a ContinuationUri that we provide to transfer the user for them to complete the journey in our Adviser Hub web frontend.

<aside class="notice">
If you are a partial integrator and need to know when a quote has been submitted through Adviser Hub, you can sign up for <a href="#webhooks">our webhooks system</a>.
</aside>

### QuoteRequests and Quotes

A **QuoteRequest** is a well-defined request for cover which we submit to one or more insurers. QuoteRequests have sequential integer IDs. If insurers can provide prices for the specified cover, we return these in the response and refer to each insurer offer as a **Quote**, which has a GUID (i.e. `12345678-1234-1234-1234-123456789abc`). A customer can apply for a Quote. 

## Create Quote

> POST to the `/Quote` endpoint to create a quote.

```php
<?php

// Use the token from your login response
$userId = 123456;
$token = '9c92d88f-d28f-4eb6-8e69-f96707113544'; 
$systemId = '56cba828-1376-4ced-96d4-11a950e4afe8';

$data = [
	'ProductId' => 1010,
	'BranchNumber' => 'PS180092',
	'UseDefaults': true,
	'IsIndicativeQuote': false,
	'CommissionSacrifice': 0,
	'UserReference' => 'MyRef_123',
	'Answers': [
		['InterfaceKey' => 'Applicant1Title', 'Value' => 'Miss'],
		['InterfaceKey' => 'Applicant1Forename', 'Value' => 'Jessica'],
		...
	]
];

$url  = 'https://apiuat.paymentshield.co.uk/Quote';
$json = json_encode($data);
$curl = curl_init();

curl_setopt_array($curl, [
  CURLOPT_URL => $url,
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_ENCODING => "",
  CURLOPT_MAXREDIRS => 10,
  CURLOPT_TIMEOUT => 30,
  CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
  CURLOPT_CUSTOMREQUEST => 'POST',
  CURLOPT_POSTFIELDS => $json,
  CURLOPT_HTTPHEADER => [
	"Content-Type: application/json",
	"UserId: $userId",
	"Token: $token",
	"SystemId: $systemId",
  ]
]);

$response = curl_exec($curl);
$err = curl_error($curl);

curl_close($curl);

$response = $err
	? "cURL Error $err"
	: $response;
```

```http
POST https://apiuat.paymentshield.co.uk/Quote/ HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8

{
  "ProductId": 1008,
  "BranchNumber": "AB000000",
  "UseDefaults": true,
  "IsIndicativeQuote": false,
  "CommissionSacrifice": 0,
  "UserReference": "MyRef_123",
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

Create a new full Quote Request. A *Quote Request* is our term for a well-specified request for insurance quotes, for a given customer, based on the customer information you provide. As long as your request provides valid answers to all mandatory questions, a Quote Request will be created in our database and we will pass back a `QuotesResponse` with a `QuoteRequestId`. From here, you can continue the user journey in our web frontend using the `ContinuationUri` in the response.

You can find the types and examples of each request field in the Swagger definition of the API.

Please see the code pane for an example of the HTTP headers and JSON request you should send.  The fields we require in the request are explained in more detail below:

### ProductId

Not all integrators can sell the full range of products. If you want to be authorised to sell a different range of products, please [Get in touch](https://paymentshield.co.uk). 

The links in the table below will take you to our Question Set Explorer tool which will give you details of each question.

| ProductId                                                                          | Product                            |
| ---------------------------------------------------------------------------------- | ---------------------------------- |
| [1008](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/68)  | Home - Buildings and Contents      |
| [1009](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/127) | Landlords - Buildings and Contents |
| [1010](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/80)  | Tenants Liability                  |
| [1011](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/136)  | Rent Protection                    |
| [1003](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/48)  | Mortgage Protection                |
| [1004](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/99)  | Income Protection                  |
| [1013](https://servicesuat.paymentshield.co.uk/Quotations/QuestionSet/Details/133) | Tenants Contents                   |

### Other fields in the create quote request

| Field                   | Details                                                                                                                                                                                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **BranchNumber**        | The `BranchNumber` is provided as part of a user's configuration in our back end system.  To create a Quote Request a user must be authorised to sell the product under the branch number you send in the request.                                             |
| **UseDefaults**         | We recommend that `UseDefaults` is set to true. It sets some Answers, including calculated Answers (total coverage value for STIP/MPPI), to their sensible default or calculated value if you don't specify one.                                               |
| **IsIndicativeQuote**   | We recommend that `IsIndicativeQuote` is set to false.  If this is set to true the resulting quotes cannot be applied for.                                                                                                                                     |
| **CommissionSacrifice** | The maximum amount of commission that can be sacrificed by a user is provided as part of a user's configuration in our back end system.  The value sent in the `CommissionSacrifice` field must not exceed the maximum for the user/branch number combination. |
| **UserReference**       | An optional field that you can use to record your own cross reference against a quote. You can also use the `UserReference` to search for quotes.                                                                                                              |
| **Answers**             | You should include the **InterfaceKey/Values** required for the specified product in the `Answers` array.  Each product has a certain number of mandatory answers that must be provided to get a quote.                                                        |

<aside class="notice">
To find out what values you should send in the <code>Answers</code> array, you can use the Questionset browser for each product by clicking the ProductId in the above table.
</aside>

### Repeating/multi-part Answers

> Example request with multi-part answers

```json
{
  "ProductId": 1008,
  "BranchNumber": "AB000000",
  "UseDefaults": true,
  "IsIndicativeQuote": false,
  "CommissionSacrifice": 0,
  "Answers": [
    ...
	{
      "Value": "true",
      "InterfaceKey": "ContentsItemsToSpecify"
    },
    {
      "Value": "ComputerEquipment",
      "InterfaceKey": "SpecifiedItemType",
      "RepeatingQuestionSetIndex": 0
    },
    {
      "Value": "High Performance PC",
      "InterfaceKey": "SpecifiedItemDescription",
      "RepeatingQuestionSetIndex": 0
    },
    {
      "Value": "3000",
      "InterfaceKey": "SpecifiedItemValue",
      "RepeatingQuestionSetIndex": 0
    },
    {
      "Value": "JewelleryOrWatches",
      "InterfaceKey": "SpecifiedItemType",
      "RepeatingQuestionSetIndex": 1
    },
    {
      "Value": "Watch",
      "InterfaceKey": "SpecifiedItemDescription",
      "RepeatingQuestionSetIndex": 1
    },
    {
      "Value": "3000",
      "InterfaceKey": "SpecifiedItemValue",
      "RepeatingQuestionSetIndex": 1
    },
    ...
  ]
}
```

Some of our products have questions that allow repeating answers and have multiple parts to the answer. An example of this is `ContentsItemsToSpecify`, which, when set to true, requires at least one group of the following answers:

| RepeatingQuestionSetIndex | InterfaceKey             |
| ------------------------- | ------------------------ |
| 0                         | SpecifiedItemType        |
| 0                         | SpecifiedItemDescription |
| 0                         | SpecifiedItemValue       |

You must send all three answers for the specified item to be added to the quote.

To add a second specified item, you would include another group of answers to these three questions, marked with an incremented **RepeatingQuestionSetIndex** of 1.

Each set of multi-part answers, whether for specified items, personal possessions, or previous claims, must have a globally unique integer value for **RepeatingQuestionSetIndex**. For example, you should not use 0 to mark both a Personal Possession and a Previous Claim; they should have different indices:

| RepeatingQuestionSetIndex | InterfaceKey              | Value              |
| ------------------------- | ------------------------- | ------------------ |
| 0                         | PPItemType                | JewelleryOrWatches |
| 0                         | PPItemDescription         | Swiss watch        |
| 0                         | PPItemValue               | 3500               |
| 1                         | PreviousClaimClaimedUnder | BuildingsOnly      |
| 1                         | PreviousClaimTypeOfLoss   | AccidentalDamage   |
| 1                         | PreviousClaimDateOfLoss   | 20/08/2015         |

### Strategies for unique RepeatingQuestionSetIndex

Here are two strategies for setting correct values for `RepeatingQuestionSetIndex` in your integration:

1. You could keep a global count of repeating sections in your state store and apply it to the corresponding Answer nodes in your request
2. You could assume a maximum for each repeating group and treat it as an offset, so that all SpecifiedItems begin at 0, PersonalPossessions begin at 10, and PreviousClaims begin at 20, for example.

The numbering of `RepeatingQuestionSetIndex` in our **response** will always start at **0**, even if your request started at 1 (or higher). Our numbering in the response will be **contiguous** (i.e. no gaps in numbering starting from 0) even if your request had gaps. 

<aside class="warning">
You must not depend on the answer responses having the same indices with which you submitted them. Always treat the Answers array as a random access array and look up or iterate over the items according to the concrete information present in the Answers array.
</aside>

The example in the code pane shows how to add two specified items to a quote - in this case a PC and a Watch.

### User Reference

You can pass an optional UserReference field in a quote request to allow you to identify the quote as per your own requirements.  For example, a customer or fact find reference from your system.

The UserReference will be returned in the Quote Response and can be used to search for quotes using a [GET Quotes](#search-quotes) request specifying the UserReference.

<aside class="notice">
If you use <code>Edit Quote</code> to create a variant of a quote request linked to the original, by default any <code>UserReference</code> attached to the original quote will be copied to the <b>Linked Quote</b>.  You may override this by setting <code>UserReference</code> explicitly on the new linked quote.
</aside>

## Quote Response

> Example Quote Response

```json
{
    "StatusCode": "OK",
    "Messages": [],
    "QuoteRequestId": 641234,
    "QuoteReference": "PSL-BCHI-641234",
    "QuoteStatus": "QUOTESRETRIEVED",
    "ProductId": "1008",
    "QuoteSessionId": "932d804e-20a0-4a7e-9c28-a6f123aa16d4",
    "QuoteSessionContinuationUri": "http://test3.paymentshield-advisers.co.uk/QuoteRequests/LoginWithAccessToken?userId=336427&token=82dec108-cd8f-4b51-b429-a6bf49dc3978&redirectUri=%2FQuoteRequests%2FContinueQuoteSession%2F932d804e-20a0-4a7e-9c28-a6f123aa16d4",
    "QuoteRequestContinuationUri": "http://test3.paymentshield-advisers.co.uk/QuoteRequests/LoginWithAccessToken?userId=336427&token=82dec108-cd8f-4b51-b429-a6bf49dc3978&redirectUri=%2FQuoteRequests%2FContinue%2F641234",
    "QuoteDate": "2019-07-31T15:26:24.0263172+01:00",
    "QuoteExpiryDate": "2019-09-29T00:00:00+01:00",
    "IsMultiTier": true,
    "IsIndicativeQuote": false,
    "CommissionSacrifice": 0,
	"UserReference": "YourRef_123",
    "Quotes": [
        {
            "QuoteId": "4b3ccf9a-1df7-2d6c-3cd0-48f3bd8e1111",
            "InsurerName": "ABC Underwriting Services Limited",
            "ProductName": "Home Insurance",
            "Cover": {
                "IsRequested": true,
                "BuildingsAD": "Excluded",
                "ContentsAD": "Excluded",
                "BuildingsCoverLevel": 500000,
                "ContentsCoverLevel": 50000
            },
            "Premiums": [
                {
                    "PaymentMethod": "DirectDebit",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "5bc2cd93"
                },
                {
                    "PaymentMethod": "CreditCard",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "5bc2cd93"
                },
                {
                    "PaymentMethod": "DebitCard",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "5bc2cd93"
                },
                {
                    "PaymentMethod": "DirectDebit",
                    "PaymentFrequency": "Monthly",
                    "PaymentFrequencyDescription": "Monthly",
                    "CreditDetails": {
                        "Apr": 23.7,
                        "CreditCharge": 25.13,
                        "FixedInterestRate": 12,
                        "InitialPayment": 19.57,
                        "NumberOfSubsequentPayments": 11,
                        "SubsequentPayments": 19.54
                    },
                    "PriceID": "0f2a819f"
                }
            ],
            "Endorsements": []
        },
        {
            "QuoteId": "ddfe58c7-b70a-4901-ba40-c1de23c4dadb",
            "InsurerName": "ABC Underwriting Services Limited",
            "ProductName": "Home Insurance",
            "Cover": {
                "IsRequested": false,
                "BuildingsAD": "Included",
                "ContentsAD": "Excluded",
                "BuildingsCoverLevel": 500000,
                "ContentsCoverLevel": 50000
            },
            "Premiums": [
                {
                    "PaymentMethod": "DirectDebit",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "c9218424"
                },
                {
                    "PaymentMethod": "CreditCard",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "c9218424"
                },
                {
                    "PaymentMethod": "DebitCard",
                    "PaymentFrequency": "Annual",
                    "PaymentFrequencyDescription": "Annually",
                    "PriceID": "c9218424"
                },
                {
                    "PaymentMethod": "DirectDebit",
                    "PaymentFrequency": "Monthly",
                    "PaymentFrequencyDescription": "Monthly",
                    "CreditDetails": {
                        "Apr": 23.7,
                        "CreditCharge": 27.57,
                        "FixedInterestRate": 12,
                        "InitialPayment": 21.4,
                        "NumberOfSubsequentPayments": 11,
                        "SubsequentPayments": 21.45
                    },
                    "PriceID": "4a975c3f"
                }
            ],
            "Endorsements": []
			
		...	
        
		}

    ],
    "Prices": [
        
        {
            "ID": "0f2a819f",
            "Values": {
                "TotalCostPerPaymentPeriod": 19.54,
                "TotalPremiumIncludingIPT": 209.38,
                "TotalPremiumExcludingIPT": 190.81,
                "AdministrationCharge": 36,
                "InsurancePremiumTax": 18.57,
                "HouseholdInsurancePremiumExcludingIPT": 137.77,
                "HomeEmergencyPremiumExcludingIPT": 0,
                "LegalExpensesPremiumExcludingIPT": 17.04,
                "BuildingsCoverGrossExcludingIPT": 75.17,
                "ContentsCoverGrossExcludingIPT": 62.6,
                "TotalAmountPayable": 234.51
            }
        },
        {
            "ID": "4a975c3f",
            "Values": {
                "TotalCostPerPaymentPeriod": 21.45,
                "TotalPremiumIncludingIPT": 229.78,
                "TotalPremiumExcludingIPT": 209.02,
                "AdministrationCharge": 36,
                "InsurancePremiumTax": 20.76,
                "HouseholdInsurancePremiumExcludingIPT": 155.98,
                "HomeEmergencyPremiumExcludingIPT": 0,
                "LegalExpensesPremiumExcludingIPT": 17.04,
                "BuildingsCoverGrossExcludingIPT": 93.38,
                "ContentsCoverGrossExcludingIPT": 62.6,
                "TotalAmountPayable": 257.35
            }
        ...
		
        }
    ],
    "Answers": [
        {
            "InterfaceKey": "ExcessBuildings",
            "Value": "TwoHundredAndFifty"
        },
        {
            "InterfaceKey": "ExcessContents",
            "Value": "TwoHundredAndFifty"
        },
        {
            "InterfaceKey": "PersonalPossessionsToSpecify",
            "Value": "False"
        },
       
	...
	
    ]
}
```

A successful Quote response includes the following information:

| Field                           | Details                                                                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **StatusCode**                  | A successful response will have a status code of OK.                                                                                 |
| **Messages**                    | For a successful response, this will be an empty array.                                                                              |
| **QuoteRequestId**              | The unique integer ID for the Quote Request.  Used in other API calls to retrieve Quotes and Documents.                              |
| **QuoteReference**              | The full Paymentshield Quote Reference that should be relayed to a customer and will display on documents.                           |
| **QuoteStatus**                 | Reflects the state of the quote in the Paymentshield system.  A new quote will have a status of 'QUOTESRETRIEVED'                    |
| **ProductId**                   | Confirmation of the ProductId sent on the quote request.                                                                             |
| **QuoteSessionId**              | A reference unique to a series of linked quotes.                                                                                     |
| **QuoteSessionContinuationUri** | A continuation URI that can be used to transfer the user to the Paymentshield quote and apply journey to continue the session.       |
| **QuoteRequestContinuationUri** | A continuation URI that can be used to transfer the user to the Paymentshield quote and apply journey to continue the quote request. |
| **QuoteDate**                   | Confirmation of the date and time the quote was created in our system.                                                               |
| **QuoteExpiryDate**             | The received quote can be applied for up to the Quote Expiry Date.                                                                   |
| **IsMultiTier**                 | A flag set in our back end system that determines the number of alternative quotes that we will return in the quote response.        |
| **IsIndicativeQuote**           | Confirmation of the value passed in the Quotes request message.                                                                      |
| **CommissionSacrifice**         | Confirmation of the value passed in the Quotes request message.                                                                      |
| **UserReference**               | Confirmation of the value passed in the Quotes request message.                                                                      |
| **Quotes array**                | Explained below                                                                                                                      |
| **Prices array**                | Explained below                                                                                                                      |
| **Answers array**               | We use the **Answers** array to replay the answers that were sent in the Quotes request message.                                     |

### Quotes array

For each insurer quote we receive, we return the following data:

| Field            | Details                                                                                                                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **QuoteId**      | A unique quote GUID.  Used in other API calls to retrieve documents and to apply for a quote.                                                                                                     |
| **InsurerName**  | The customer facing insurer name as will appear on customer documents.                                                                                                                            |
| **ProductName**  | The customer facing product name as will appear on customer documents.                                                                                                                            |
| **Cover**        | For products that support multi-tier the cover node provides a summary of the basis of each quote.  The **IsRequested** flag will be `true` for the cover that exactly matches the quote request. |
| **Premiums**     | The premiums node provides a link to the **PriceId** for each payment type/frequency option available for the quote.                                                                              |
| **Endorsements** | If one of our insurers returns an endorsement the text will be provided in the endorsements field for you display to your customer.                                                               |

### Prices array

For each unique **PriceId** returned in **Quotes**, we provide a breakdown of values relevant to the product:

| Field      | Details                                                                                                                                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ID**     | Each **ID** in the **Prices** node will be referenced by a **PriceId** in the **Quotes** node. These IDs are ephemeral and will not be the same between two responses, even if the Price data is the same. |
| **Values** | The breakdown of values returned will be specific to the productId requested.                                                                                                                              |

### Continuation URI

For 'Partial Integrators', who wish to create a quote but transfer the user to our web journey to complete the quote and apply journey, we provide ContinuationUris in the Quotes response. 
As long as the User has activated their Adviser Hub account the URI provides a silent logon to our quote and apply application.
You may encounter three types of ContinuationUris as per the table below:

| URI type                    | Details                                                                                                                                                                                                                                                                                  |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| QuoteSessionContinuationUri | This is returned in a successful Post and Get Quote response.  The URI can be used to transfer the user to latest stage they were at in the quote journey.  If multiple linked quotes exist the `QuoteSessionContinuationUri` will transfer the user to the latest quote created.        |
| QuoteRequestContinuationUri | This is returned in a successful Post Quote response and in a Get Quote response providing neither the quote request nor any of its linked quotes have been applied for.  The `QuoteRequestContinuationUri` will transfer the user to the latest point in the quote sent in the request. |
| QuickQuoteContinuationUri   | This is returned in a successful Post QuickQuote response.  The URI can be used to transfer the user to our web based quote and apply journey to confirm assumptions and continue to generate a full quote.                                                                              |

## Create Partial Quote

> Json showing the IsPartialQuote parameter used to create a Partial Quote

```json
{
 "ProductId": 1008,
 "BranchNumber": "AB000000",
 "UseDefaults": true,
 "IsIndicativeQuote": false,
 "CommissionSacrifice": 0,
 "IsPartialQuote": true,
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

There are two main reasons that you might build your integration to submit ['partial' quote requests](#full-vs-partial-integration):

1. If you want to start a quote with a small amount of information, save it, and update it with more information when available or resume the journey later in our Adviser Hub web frontend.
2. If you want to capture some information in your application, and transfer right away to our web frontend to complete the quote and buy journey.

If you send the `IsPartialQuote` parameter with a value of `true` in a [POST Quote request](#create-quote), then we skip part of the validation logic, so that the request is not refused, but instead returns a response where the `QuoteStatus` is `INPROGRESS`.

If omitted, the `IsPartialQuote` value defaults to `false`.

In the response, you will receive a `QuoteRequestId` and a `QuoteSessionContinuationUri` that you can use to transfer to our quote and apply journey to complete the remaining questions and obtain a full quote.  When you transfer to our journey the answers you provided will be pre-populated.

<aside class="notice">
If you send a Quotes request with <code>IsPartialQuote</code> set to <code>true</code>, but send answers to all mandatory questions, the <code>QuoteStatus</code> will still be returned as <code>INPROGRESS</code>.  This allows for subsequent addition of non-mandatory answers before requesting the full quote response.
</aside>

### Update a Partial Quote

You can send updates to a Partial Quote by POSTing to the `/Quote` endpoint specifying the QuoteRequestId to be updated.

For example, to update the QuoteRequestId `641234`, the endpoint is `/Quote/641234`.

You can use the update function to add more information to a QuoteRequest, or change answers previously sent.
The update is not a differential; it will completely replace the existing answers for the QuoteRequest. If you exclude Answers from your update which were previously set, this will effectively remove them.  

<aside class="notice">
To update answers for an existing quote, it must be at status <code>INPROGRESS</code>.  If it is at status <code>QUOTESRETRIEVED</code> the request will be interpreted as an <code>Edit</code> resulting in a new Linked Quote.
</aside>

## Create QuickQuote

> Create a QuickQuote

```http
POST https://apiuat.paymentshield.co.uk/QuickQuote/ HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8

{
  "ProductId": 1008,
  "BranchNumber": "AB000000",
  "UseDefaults": true,
  "IsIndicativeQuote": false,
  "CommissionSacrifice": 0.0,
  "UseThirdPartyData": false,
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

A QuickQuote is an indicative price which you can get by submitting less information about the customer. You can allow your customer to pick up their QuickQuote journey in our frontend using the `QuickQuoteContinuationUri`. 

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

We add assumptions for all other mandatory questions in order to return a price in the response.  The assumptions we use are replayed back to you in the Quick Quote Response.

Please see the code pane for an example of QuickQuote request message.

### Flexible Quick Quote

If you are integrating from another application and have more information available than the above list of minimum data requirements you can send additional fields.  The more information you are able to provide in the quick quote request the more accurate the quote response will be.  

### Third Party Property Data

If you send the `UseThirdPartyData` parameter with a value of `true` in a POST QuickQuote request, you can omit the following three fields from the QuickQuote request:

* PropertyType
* NumberOfBedrooms
* BuildYear

We will use the address details sent in the request to call a third party data provider to source the property details which will then be added to the request.

If omitted, the `UseThirdPartyData` value defaults to `false`.

### QuickQuotesResponse

We deliver a `QuickQuotesResponse` in response to both the 'Create' and 'Get' QuickQuote operations. This object has some differences from a normal QuotesResponse:

* It has a `QuickQuoteContinuationUri` instead of `QuoteSessionContinuationUri`
* Only the cheapest Price is returned, instead of many prices
* We confirm the source of each `Value` in the `Answers` array using a `Source` field which will either return as 'User' if supplied in the request, 'ThirdParty' if the value has been sourced from an external data provider or 'Assumed' if the value has been assumed.

> Error returned when unable to source ThirdPartyData

```json
{
    "StatusCode": "NoDataWarning",
    "Messages": [
        {
            "MessageType": "Error",
            "Summary": "Sorry, we couldn't find any information for that property"
        }
    ]
}
```

If you send the `UseThirdPartyData` parameter with a value of `true` and we are unable to source the property data we will return an `error` response as per the example provided.

## Edit Quote

You can create a new version of a quote linked to an existing QuoteRequestId by POSTing to the `/Quote` endpoint specifying the QuoteRequestId to link the new quote to.  

For example, to create a new QuoteRequestId that is linked to the existing QuoteRequestId `645678`, the endpoint is `/Quote/645678`.

Use the edit function to create variants of a quote for a single customer.  As the quotes are [Linked](#linked-quotes) it is only possible to apply for one of them.  This removes the chance of inadvertently creating multiple policies for a single customer.

The edit function creates a totally new `QuoteRequestId` so the request should include all required information and not just the answers that have been changed from the original.   

<aside class="notice">
To create a new version of a quote using the edit function, the existing quote must be at status <code>QUOTESRETRIEVED</code>. 
</aside>

## Retrieve Quote

> Retrieve a set of Quotes by QuoteRequestId

```http
GET https://apiuat.paymentshield.co.uk/Quote/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

Retrieve a fully-hydrated Quote Response from a previously created quote. This allows you to show your user up-to-date quote state and whatever values they filled in originally. You can use this feature to resume a journey. Because we store the full quote information associated with a `QuoteRequestId`, you don't have to store full quote information in your database; you can look it up with this operation.

Pass the integer `QuoteRequestId` from the original Quote Response.

### Linked Quotes

> Example of LinkedQuotes in a GET Quote Response

```json
{
    "StatusCode": "OK",
    "Messages": [],
    "QuoteRequestId": 640111,
    "QuoteReference": "PSL-BCHI-640111",
    "QuoteStatus": "QUOTESRETRIEVED",
    "ProductId": "1008",
    "QuoteSessionId": "1fa06ed9-64f0-4ee3-be1f-c123c45c6d7f",
    "QuoteSessionContinuationUri": "http://test3.paymentshield-advisers.co.uk/QuoteRequests/LoginWithAccessToken?userId=336427&token=5689552a-4608-44b3-b554-5c02408304d6&redirectUri=%2FQuoteRequests%2FContinueQuoteSession%2F1fa06ed9-64f0-4ee3-be1f-c123c45c6d7f",
	"QuoteRequestContinuationUri": "http://test3.paymentshield-advisers.co.uk/QuoteRequests/LoginWithAccessToken?userId=336427&token=5689552a-4608-44b3-b554-5c02408304d6&redirectUri=%2FQuoteRequests%2FContinue%2F640111",
    "QuoteDate": "2019-08-01T10:41:11.43",
    "QuoteExpiryDate": "2019-09-30T00:00:00",
    "IsMultiTier": true,
    "IsIndicativeQuote": false,
    "CommissionSacrifice": 25.00,
    "LinkedQuotes": [
        {
            "QuoteRequestId": 640112,
            "Link": "http://apiuat.paymentshield.co.uk/Quote/640112",
            "QuoteStatus": "QUOTESRETRIEVED",
            "QuoteDate": "2019-08-01T10:44:06.8",
            "IsApplied": false
        },
        {
            "QuoteRequestId": 640113,
            "Link": "http://apiuat.paymentshield.co.uk/Quote/640113",
            "QuoteStatus": "SUBMITTED",
            "QuoteDate": "2019-08-01T10:55:20.813",
            "IsApplied": true
        }
    ],
 ...
 
}
```

There are two ways to create linked quotes:

1. When your user transfers to our quote and apply journey (using one of the above continuation uris), it is possible for them to use features on our website to create variations of the quote.  
2. Using the [Edit Quote](#edit-quote) function to create a new variant of a quote linked to another QuoteRequestId.

In both of the above cases, each quote variant is given its own unique **Quote Request Id**.  We will return details of all linked quotes when you next perform a Get Quote request.  

Please see the code pane for an example of this.  In the response shown the user has created three linked quotes and has applied for a quote within 640113.

Note that each `QuoteRequestId` has its own `QuoteStatus`.  Only one of the linked quotes can have a status of `SUBMITTED`.

### QuoteStatus Codes

You may encounter the following `QuoteStatus` codes in a Get Quote response:

| QuoteStatus     | Meaning                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------- |
| INPROGRESS      | A quote has been started in our system but not yet been submitted for pricing.              |
| QUOTESRETRIEVED | The Quote Request has been submitted for pricing and prices have been returned.             |
| SUBMITTED       | One of the quotes returned under the Quote Request ID has been applied for.                 |
| UNABLETOQUOTE   | The Quote Request has been submitted for pricing but we have been unable to return a quote. |

### Retrieve QuickQuote

> Retrieve a QuickQuote the same way you retrieve a Quote

```http
GET https://apiuat.paymentshield.co.uk/Quote/{QuoteRequestId} HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

You can retrieve a QuickQuote the same way you would [Retrieve a quote](#retrieve-quote), passing the integer `QuoteRequestId` from the original QuickQuote Response.

If the user has progressed a QuickQuote by adding information, so that it now has many prices, then we return a full `QuotesResponse`.

## Search Quotes

> Search for Quotes by UserReference

```http
GET https://apiuat.paymentshield.co.uk/Quotes/?UserReference=YourRef_123 HTTP/1.1
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8
```

> Example of GET Quotes response

```json
{
    "StatusCode": "OK",
    "Messages": [
        {
            "MessageType": "ResultsLimited",
            "Summary": "We've limited the number of results to 100"
        }
    ],
    "Results": [
        {
            "QuoteRequestId": 634560,
            "Link": "http://s-pstest-web002.paymentshield.co.uk:1993/Quote/634560",
            "QuoteStatus": "SUBMITTED",
            "QuoteDate": "2020-01-30T11:20:31.343",
            "Applicant1Surname": "Matip",
            "Applicant1DoB": "1970-05-01T00:00:00+01:00",
            "Postcode": "PR9 0SL",
            "ProductId": "1008",
            "ApplicationRef": "PSL-INT-9005780"
        },
        {
            "QuoteRequestId": 634562,
            "Link": "http://s-pstest-web002.paymentshield.co.uk:1993/Quote/634562",
            "QuoteStatus": "SUBMITTED",
            "QuoteDate": "2020-01-30T11:20:59.373",
            "Applicant1Surname": "Robertson",
            "Applicant1DoB": "1989-05-01T00:00:00+01:00",
            "Postcode": "PR8 3AE",
            "ProductId": "1008",
            "ApplicationRef": "PSL-INT-9005782"
        },
        {
            "QuoteRequestId": 640753,
            "Link": "http://s-pstest-web002.paymentshield.co.uk:1993/Quote/640753",
            "QuoteStatus": "QUOTESRETRIEVED",
            "QuoteDate": "2020-02-11T13:23:58.13",
            "Applicant1Surname": "Becker",
            "Applicant1DoB": "1986-05-01T00:00:00+01:00",
            "Postcode": "PR9 0SL",
            "ProductId": "1008",
            "QuoteRequestContinuationUri": "http://test3.paymentshield-advisers.co.uk/QuoteRequests/LoginWithAccessToken?userId=336427&token=a9501932-5e32-4a12-be5a-589cd7e39d1c&redirectUri=%2FQuoteRequests%2FContinue%2F640753"
        },
		...
	
}
```

Use the `GET Quotes` (plural) endpoint to search for quotes based on different criteria. Pass the criteria in the querystring of the GET request (see example)

### Supported Fields

We are expanding this feature. It currently supports the following fields as search criteria:

| Field         | Definition                                                           |
| ------------- | -------------------------------------------------------------------- |
| UserReference | Your reference attached to the quote using the `UserReference` field |

Use the `GET Quotes` endpoint specifying a UserReference to get a list of quotes with a matching UserReference value. You must provide the full reference, not a fragment or wildcard.

We will return details of all quotes that have a UserReference matching the request.  In the example provided, all quotes with a UserReference of 'YourRef_123' will be returned in the response.  

Please see the code pane for an example of the JSON response.  In the example used, the search has returned more than 100 results.  We will only return the first 100 results in the response.

We return some key information about each QuoteRequest to help you to identify the quote required. 
You can use the `Link` returned in the response to make a `GET Quote` request to retrieve the fully-hydrated quote response.

## Apply

```http
POST https://apiuat.paymentshield.co.uk/Quote/{QuoteId}/Apply HTTP/1.1
Content-Type: application/json
UserId: 123456
Token: 9c92d88f-d28f-4eb6-8e69-f96707113544
SystemId: 56cba828-1376-4ced-96d4-11a950e4afe8

{
  "UseDefaults": true,
  "CommissionSacrifice": 0.0,
  "Answers": [
    {
      "Value": "01/10/2019",
      "InterfaceKey": "PolicyStartDate"
    },
    {
      "Value": "01704555444",
      "InterfaceKey": "Applicant1TelNo"
    },
    {
      "Value": "Online",
      "InterfaceKey": "DocumentDeliveryPreference"
    },
    {
      "Value": "applicant@example.com",
      "InterfaceKey": "Applicant1EmailAddress"
    },
    {
      "Value": "Advised",
      "InterfaceKey": "BasisOfAdvice"
    },
    {
      "Value": "TCTest01",
      "InterfaceKey": "YourRef"
    },
    {
      "Value": "DirectDebit",
      "InterfaceKey": "PaymentMethod"
    },
    {
      "Value": "Annual",
      "InterfaceKey": "PaymentFrequency"
    },
    {
      "Value": "Mr T Tester",
      "InterfaceKey": "BankAccountName"
    },
    {
      "Value": "101010",
      "InterfaceKey": "BankSortCode"
    },
    {
      "Value": "12345678",
      "InterfaceKey": "BankAccountNo"
    },
    {
      "Value": "First",
      "InterfaceKey": "DirectDebitDate"
    }
  ]
}
```

You can apply for an existing Quote by POSTing to the `Apply` action on the quote resource, passing answers for the 'Application Stage' questions.

For example, to apply for the quote with QuoteId `cbf8196c-7757-49a9-8511-c5abf6b4b1c6`, the endpoint is `/Quote/cbf8196c-7757-49a9-8511-c5abf6b4b1c6/Apply`.

You can find the list and descriptions of the 'Application Stage' questions in the Question Set Explorer tool, which you can find from the [ProductId](#productid) table earlier in this document.

### Apply Response

The Apply Response resembles the [Quote Response](#quote-response), with many of the same fields.

On successful application, the `QuoteStatus` will be `SUBMITTED`. If the request fails, the status will stay at `QUOTESRETRIEVED`.