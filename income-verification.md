# ResidentIQ Pre-Screening Income Verification

## Setup

The ResidentIQ Income Verification product (IV for short) requires an API Key for initial access and all API transactions. This key will be provided by the ResidentIQ team.

## Usage

To request an income verification link for an applicant, the API partner will send a POST request to https://finapi.residentiq.com/api/link. The body of this request will include one parameter in JSON format:
```json
{
    "postbackUrl": "{YOUR POSTBACK URL}"
}
```

If the request is successful, this api call will result in the following response:
```json
{
    "verificationToken": "47bf8051-bed0-4e17-98d4-e6998b978308",
    "verificationUrl": "https://fin.residentiq.com/verify/47bf8051-bed0-4e17-98d4-e6998b978308"
}
```

The verification URL will take an applicant to the RIQ Income Verification application, where they will be asked a series of questions to determine their monthly income. This application features an integration with Finicity Connect, allowing a user to select their bank(s) and accounts, then have their monthly income automatically calculated. Please note, this information is NOT stored within the RIQ system; all bank information is kept within Finicity's system, and all transactional information is transient, only.

Once the applicant finishing the IV process, a payload will be posted to the postbackUrl provided during link generation. This postback will include the following information:
```json
{
  "verificationToken": "7794981a-bafa-491d-8eef-2ac133e10210",
  "calculatedIncome": 0.00,
  "additionalIncome": 725,
  "hasHousingVoucher": true,
  "voucherAmount": 567,
  "hasBankAccount": false,
  "userSpecifiedIncome": 2500,
  "uploadedDocuments": [
    {
      "verificationToken": "7794981a-bafa-491d-8eef-2ac133e10210",
      "fileName": "test copy.jpg",
      "timestamp": "2022-02-11T20:43:56.07",
      "documentContents": "REMOVED FOR EXAMPLE",
      "documentType": 0
    }
  ]
}
```

Please note that all income sources that the user specifies are transmitted separately. Calculated Income will only be populated if the user completes the FinicityConnect scenario.

Once this postback is received, the applicant's IV is marked as complete. At this point, the link will be deactivated, and no additional changes can be made to the provided data.

