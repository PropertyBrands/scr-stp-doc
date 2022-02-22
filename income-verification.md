# ResidentIQ Pre-Screening Income Verification

## Setup

The ResidentIQ Income Verification product (IV for short) requires an API Key for initial access and all API transactions. This key will be provided by the ResidentIQ team.

## Usage

To request an income verification link for an applicant, the API partner will send a POST request to https://finapi.residentiq.com/api/link. The body of this request will include two parameters in JSON format:
```json
{
    "postbackUrl": "{YOUR POSTBACK URL}",
    "referenceId": "any string of length < 50"
}
```

The postbackUrl is where results will be delivered when a user is done with the IV process. The ReferenceId is optional and must be less than 50 total characters. This ReferenceId will be provided with the final postback.

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
  "verificationToken": "47bf8051-bed0-4e17-98d4-e6998b978308",
  "calculatedIncome": 0.00,
  "additionalIncome": 725,
  "hasHousingVoucher": true,
  "voucherAmount": 567,
  "referenceId": "id povided during link generation",
  "hasBankAccount": false,
  "totalIncome": 3250.00,
  "userSpecifiedIncome": 2500,
  "uploadedDocuments": [
    {
      "verificationToken": "47bf8051-bed0-4e17-98d4-e6998b978308",
      "fileName": "test copy.jpg",
      "timestamp": "2022-02-11T20:43:56.07",
      "documentContents": "REMOVED FOR EXAMPLE"
    }
  ]
}
```

Please note that all income sources that the user specifies are transmitted separately. The TotalIncome field provides a simplified field that adds the CalculatedIncome and/or UserSpecifiedIncome with the AdditionalIncome. Calculated Income will only be populated if the user completes the FinicityConnect scenario.

Once this postback is received, the applicant's IV is marked as complete. At this point, the link will be deactivated, and no additional changes can be made to the provided data.

## Screening

When the API partner receives the above postback, that Income Verification can be associated to a screening in the ResidentIQ system. The following node can be used during the Screening-Create process to associate a new screening record to an existing IV:
```xml
<AdditionalItems type="x:income_verification_token">
    <Text>{token | GUID}</Text>
</AdditionalItems>
```

Note, income verifications can only be assigned to ONE screening.

## Troubleshooting

To ensure the income verification token is valid, the following checks are performed during screening creation:
* If the screening SearchPackage contains an income verification search, does the screening payload include a token? If not, the API will pass back an error message of Income Verification Token is required for this package.
* If the token cannot be parsed into a valid GUID, the API will pass back: Income verification token cannot be parsed.
* If the verification token has previously been mapped to a screening, the API will respond with: Verification Token has been previously mapped to another report.

## Testing

The following curl request can be used to test connectivity:

```curl
curl -H "api-key: {API KEY HERE}" -i https://finapi.residentiq.com/api/link
```

If successful, the curl request should return a 200.

Please note, as of 2/22/2022, any testing that uses the FinicityConnect scenario is going to their sandbox environment; any data captured during this time will be lost when we migrate to production.

To test Finicity Connect, a user may use the following information in lieu of live data:
```
Bank Name: Finbank
UserName: Testing
Password: Profile_02
```

The password here can be updated to pull back different result sets. The following passwords are valid for testing purposes:
- Profile_02
- Profile_03
- Profile_04
- Profile_05
- Profile_06

The Bank Name and UserName fields should remain the same.

Please note that **all banking information is stored in the Finicity system, NOT ResidentIQ.** 