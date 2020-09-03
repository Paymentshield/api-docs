# Webhooks

## About our Webhooks

If you need to know when a quote has been submitted through Adviser Hub, you can ask to be included in our webhooks system.

Once you subscribe, we can notify your chosen server URL when certain events happen in our system. Currently the only notifiable event is when a policy is applied for (submitted).

Please get in touch with your technical contact to arrange this, and they will ask for your webhook URL, and a notification email address for when something goes wrong. These settings are scoped to your `SystemId`. We will send you a secret, which is a GUID specific to your integration.

## Trust

Our webhook payloads include an additional header, `X-Paymentshield-Secret` which will contain the GUID secret shared with you earlier. You can use this as a security measure to make sure that the webhook payload is coming from us.

## Apply Webhook

> An example of an Apply Response that would be sent to your webhooks endpoint

```json
{
  "StatusCode": "OK",
  "Messages": [],
  "QuoteRequestId": 4793160,
  "QuoteReference": "PSL-BCHI-4793160",
  "ApplicationReference": "PSL-INT-0710529",
  "QuoteStatus": "SUBMITTED",
  "ProductId": "1008",
  "QuoteSessionId": "483c607c-553a-48ac-ab76-9e6c6f6c4fd3",
  "QuoteSessionContinuationUri": "https://uat.paymentshield-advisers.co.uk/QuoteRequests/...",
  "QuoteDate": "2020-08-19T10:37:09+00:00",
  "QuoteExpiryDate": "2020-10-18T00:00:00+00:00",
  "CommissionSacrifice": 0.00,
  "CardPayment": null,
  "LinkedQuotes": [],
  "Quote": {
    "QuoteId": "e118b54b-36c4-4377-89e0-90306d3d3939",
    "QuoteContinuationUri": null,
    "InsurerName": "UKG",
    "ProductName": "Home Insurance",
    "ApplicationId": null,
    "Offer": null,
    "Cover": {
      "IsRequested": false,
      "BuildingsAD": "Excluded",
      "ContentsAD": "Excluded",
      "BuildingsCoverLevel": 500000.0,
      "ContentsCoverLevel": 50000.0
    },
    "Premiums": [
      {
        "PaymentMethod": "DirectDebit",
        "PaymentFrequency": "Annual",
        "PaymentFrequencyDescription": "Annually",
        "CreditDetails": null,
        "PriceID": "e88d0787"
      }
    ],
    "Endorsements": []
  },
  "Price": {
    "ID": "e88d0787",
    "Values": {
      "TotalCostPerPaymentPeriod": 351.63,
      "TotalPremiumIncludingIPT": 351.63,
      "TotalPremiumExcludingIPT": 317.81,
      "AdministrationChargeIncludingIPT": 36.00,
      "AdministrationChargeExcludingIPT": 36.00,
      "InsurancePremiumTax": 33.82,
      "HouseholdInsurancePremiumIncludingIPT": 290.18,
      "HouseholdInsurancePremiumExcludingIPT": 259.09,
      "HomeEmergencyPremiumIncludingIPT": 0.0,
      "HomeEmergencyPremiumExcludingIPT": 0.0,
      "LegalExpensesPremiumIncludingIPT": 25.45,
      "LegalExpensesPremiumExcludingIPT": 22.72,
      "BuildingsCoverGrossExcludingIPT": 136.24,
      "ContentsCoverGrossExcludingIPT": 85.81,
      "PersonalCoverGrossExcludingIPT": 37.04,
      "TotalAmountPayable": 351.63,
      "TransactionCharge": 0.00
    }
  },
  "Answers": [
    ...
  ]
}
```

When you are subscribed to the `Apply` action and a policy for your `SystemId` is submitted, we will:

* Send an ['Apply Response'](#apply-response) payload to your webhook URL, and
* Expect a reply with status `200 OK` within 30 seconds.

If we don't receive the expected response, you should receive an error notification email, and the system will retry on a backing-off basis. The latter is a work in progress.

You can use the Apply Response to update the status of the quote in your system. You can also take it as notification that the final documents, such as QAS and SDN, are available to download.

Test.