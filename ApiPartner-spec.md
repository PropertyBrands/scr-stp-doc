# ResidentIQ Partner API

The ResidentIQ Partner API provides a simple access point for pulling report URL's and setting property decisions.

## API Partner Setup (Checklist)

1. API Partner Credentials have been setup in RIQ and passed to partner
2. API Partner client mappings have been setup for report access.

## Authentication

All calls to RIQ partner api require a valid JSON Web Token, presented as a bearer token in the Authorization header for each request.

## Standard Error Response 
```xml
<?xml version="1.0"?><BackgroundReports>
  <BackgroundReportPackage>
    <ReferenceId>{Report Number}</ReferenceId>
    <ScreeningStatus>
      <OrderStatus>x:error</OrderStatus>
    </ScreeningStatus>
    <ErrorReport>
      <ErrorCode>1</ErrorCode>
      <ErrorDescription>{Error Message from RIQ}</ErrorDescription>
    </ErrorReport>
  </BackgroundReportPackage>
</BackgroundReports>
```

## Requests/Routes

### Environment URLs:
1. sandbox : https://apiqa.residentiq.com
2. production : https://api.residentiq.com

# Note :
2. Report Urls returned from the API are transient and only live for 5 minutes. this can be overriden for a given location if needed.
3. All requests should have a Content-Type header of application/json

***
### POST : /partner-token
The Partner Token route takes a simple JSON object with auth credentials(username,password) and returns an auth token that can be used on subsequent requests.

### Examples:
1. **Request**: 
```JSON
{
    "username": "provided by RIQ",
    "password": "provided by RIQ"    
}
```

### Examples:
1. **Success**: Successful auth results in a 200 HTTP response with a token object in the body.
```
{
    "token": "JWT Bearer Token"
}
```

1. **Failure**: If there are any errors during the authentication process, an error object will be returned in response body.
```
{
    "error": true,
    "message": "Could not verify username and password"
}
```

***

### GET : /api/apipartner/status/{reportNumber : int}
The report status route takes a report Id that exists within RIQ and returns the standard xml output with report url, if the requesting user is mapped to the client said report ran under.

2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{Will be empty}" password="{Will be empty}">
  <BackgroundReportPackage>
   <ReferenceId>{Will be empty}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
    <ScreeningStatus>
      <OrderStatus>{x:ready | x: pending}</OrderStatus>
      <OrderDecision>{Report Recommended Decision}</OrderDecision>
    </ScreeningStatus>
    <ReportURL>{Url To View Report in Browser}</ReportURL>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.

***

### POST : /api/apipartner/decision
The decision route will allow partners to set a property decision for a specific screening. Allowed property decisions are Decline, Conditional-Low, Conditional-High, and Accept. The property decision will be used when STP generates an adverse action letter on behalf of the API partner.

### Examples
1. **Request**: Similar to the status route, the API will look for a "decision" action on the BackgroundSearchPackage element, followed by a child PropertyDecision element:
```xml
{
    "decision": "{Decline|Conditional-Low|Conditional-High|Accept}",
    "reportId" : {integer id of the report in RIQ}
}
```
2. **Response Success**: If the API call passes validation, the API will return a success element in its response indicating if the update was successful or not:
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
    <ReferenceId>{Partner Provided referenceId}</ReferenceId>
    <OrderId>{STP Internal Report Number}</OrderId>
    <Success>{True / False}</Success>
  </BackgroundReportPackage>
</BackgroundReports>
```

3. **Response-Error**: Returns a standard Error response with description.

### GET : /api/apipartner/aal/{reportNumber : int}
The apipartner/aal route will pull back the most recently created adverse action letter for the requested report number. If no AAL files exist for the report, see item 1.1. Otherwise, assuming no error, the API will respond with a file name and file bytes, per the payload in 1.

1. **Response Success**: If the API call passes validation and the report has an adverse action letter associated with it, the API will return:
``` xml
<?xml version="1.0"?><BackgroundReports userId="{will be empty}" password="{will be empty}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{reportNumber}</OrderId>
   <SupportingDocumentation>
      <Document>
        <Name>{File Name}</Name>
        <EncodedContent>{Base64 encoded string of file bytes}</EncodedContent>
      </Document>
    </SupportingDocumentation>
  </BackgroundReportPackage>
</BackgroundReports>
```
1.1 **Response Success, no files**: If no AALs are stored for the requested report number, the API will return:
``` xml
<?xml version="1.0"?><BackgroundReports userId="{will be empty}" password="{will be empty}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{reportNumber}</OrderId>
   <SupportingDocumentation>
      No documentation exists for this report.
    </SupportingDocumentation>
  </BackgroundReportPackage>
</BackgroundReports>
```
2. **Response Failure**: Returns a standard error response with a standard error message.

### POST : /api/apipartner/client/create
The apipartner/client create route allows api partners to map their existing clients into RIQ for running reports against. The client create route is different from the other in it's JSON body vs XML. All properties on the addressa are required.

**Sample Request**: If the API call passes validation and the report has an adverse action letter associated with it, the API will return:
``` json
{
    "clientName": "Test Client", 
    "mainContactName": "Ringo Star",
    "mainPhone": "213-123-4572",
    "fax": "(213) 123-4572",
    "email": "primary.contact@company.com",
    "rentalUnitCount": 20001,
    "ownerName": "Owner Name",
    "ownerCompany": "Parent Company Name",
    "ownerPhone": null,
    "ownerEmail": "owner.contact@company.com",
    "billingContact": "Billing Admin Name",
    "billingPhone": null,
    "billingEmail": "billing.contact@company.com", -- RIQ will use the Main contact for invoice distribution if this field is null
    "associatedSalesRep": "Test Representatice",
    "addresses": [
        {
            "postalCode": "89110",
            "region": "NV",
            "city": "Las Vegas",
            "street": "8524 Yuka Ave",
            "street2": null,
            "currentAddress": false,
            "physical": true, -- Is this the clients Physical Address
            "billing": true -- Is this the clients billing Address
        }
    ]
}
```
**Response Success**: If the API call passes validation and the new client is created successfully in RIQ, the API will return the created client record with all its corresponding RIQ Id's set for mapping in the partner's system.

**Response Fail**: If the API call fails for some reason the end system can get back a couple different error responses, the API will return the created client record with all its corresponding RIQ Id's set for mapping in the partner's system.

**Validation Errors** : ex. A Required field is not included or set to null in the create request.{
```json
{
    "errors": {
        "ClientName": [
            "The ClientName field is required."
        ]
    },
    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "|d0185e90-4c9dd1ad58da36ea."
}
```

**Processing Errors** : A general Error occured during client creation that requires a developers assistance.
```json
{
    "errors": {
        "message": "Error Creating Client, please contact support"
    }
}
```
