# ResidentIQ Partner API

The ResidentIQ Partner API provides a simple access point for ordering and receiving consumer reports.

## Partner Setup (Checklist)

Any client location with an API partner setup on them can receive screenings over the RIQ Partner API but a few things need to be setup for this to work.

1. A client and property must be setup in STP with credentials and product name set (requires a CRA).
2. At least one user must be setup under the client with admin rights (requires a CRA).
3. ApiPartner has been set on the client location (requires a developer)
4. ApiPartner has been setup in the API (requires a developer)
    * add partner to the enum of ApiPartners
    * setup an api partner event handler for any new partner

## Authentication

Currently all RIQ API partners must send valid credentials in the xml payload <BackgroundSearch> element. If there is an error identifying the client or mismatch in credential API Partner will receive a standard Error Response.

## Requests/Routes

### Environment URLs:
1. development : https://apidev.residentiq.com
2. production : https://api.residentiq.com

# Note :
1. Please use ssns that corresponds with test accounts : 333221111,111223333.
2. Report Urls returned from the API are transient and only live for 5 minutes. this can be overriden for a given location if needed.

***
### POST : /api/partner
The standard post route accepts a standard xml payload in the request body and will return a success or failure response as a 200 http code. Accepted header would include application/xml.

**Grouping :** To create a record on the post route that should be immediately grouped with an existing co applicant API partners should include an additional items element with the existing applicant report number to group on.
```xml
  <AdditionalItems type="x:link_order">
     <Text>{Existing Applicant Report Number to Group On}</Text>
  </AdditionalItems>
```
**WebHooks :** STP will send updates as a post to any url provided by the api partner in an included additional items element.
```xml
  <AdditionalItems type="x:postback_url">
     <Text>{API Partner Url to call with updates}</Text>
  </AdditionalItems>
```

### Examples:
1. **Success**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
    <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
    <ScreeningStatus>
      <OrderStatus>x:pending</OrderStatus>
    </ScreeningStatus>
  </BackgroundReportPackage>
</BackgroundReports>
```

1. **Failure**: If there are any errors during validation or processing the api will return an error code with description in a standard error response.
```xml
<BackgroundReportPackage>
    <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <ScreeningStatus>
      <OrderStatus>x:error</OrderStatus>
    </ScreeningStatus>
    <ErrorReport>
      <ErrorCode>1</ErrorCode>
      <ErrorDescription>{Error Message from Partner API}</ErrorDescription>
    </ErrorReport>
  </BackgroundReportPackage>
</BackgroundReports>
```

***

### POST : /api/partner/status
The report status route takes a much simpler xml request body but follows the standard validation rules for location credentials. The Report status Order Decision element will reflect the overall group decision for any applicant tied to a group.

### Examples:
1. **Request**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{STP internal report number}</OrderId>
    <report-type>{full/decision}</report-type>
  </BackgroundSearchPackage>
</BackgroundCheck>
```

2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
    <ScreeningStatus>
      <OrderStatus>x:ready</OrderStatus>
      <OrderDecision>{Report Recommended Decision}</OrderDecision>
    </ScreeningStatus>
    <ReportURL>{Url To View Report in Browser}</ReportURL>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.

***

### POST : /api/partner/link
The link route will let API Partners group applicants after they have already been submitted to STP. STP takes a simple xml payload with 2 ids:
1. Applicant1 Id : The report number of the applicant to group
2. Applicant2 Id : The report number of the applicant or applicant in a group that applicant 1 should be associated with.

### Examples:
1. **Request**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{Applicant 1 Report Number}</OrderId>
  <AdditionalItems type="x:link_order"><Text>{Applicant 2 Report Number}</Text></AdditionalItems></BackgroundSearchPackage>
</BackgroundCheck>
```
2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
    <ScreeningStatus>
      <OrderStatus>x:pending</OrderStatus>
      <OrderDecision>{Report Recommended Decision}</OrderDecision>
    </ScreeningStatus>
    <ReportURL>{Url To View Report in Browser}</ReportURL>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.

***

### POST : /api/partner/unlink
The unlink route will remove applicant with the supplied OrderId from its current group and update the group calculations for all remaining members. 

### Examples:
1. **Request**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{STP Report Number To Decouple}</OrderId>
 </BackgroundSearchPackage>
</BackgroundCheck>
```
2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
    <ScreeningStatus>
      <OrderStatus>x:pending</OrderStatus>
      <OrderDecision>{Report Recommended Decision}</OrderDecision>
    </ScreeningStatus>
    <ReportURL>{Url To View Report in Browser}</ReportURL>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.

### POST : /api/partner/document/aal
The Adverse Action Letter route will return all AAL's on file in STP for a given report number

### Examples:
1. **Request**: If the payload passes validation api partner will receive a supporting documents response with document node populated for all records
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{STP Report Number}</OrderId>
 </BackgroundSearchPackage>
</BackgroundCheck>
```
2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
   <ReferenceId>{Partner Provided ReferenceId}</ReferenceId>
    <OrderId>{STP internal report number}</OrderId>
   <SupportingDocumentation>
      <Document>
        <Name>{File Name}</Name>
        <EncodedContent>{Base64 encoded string of File Bytes}</EncodedContent>
      </Document>
    </SupportingDocumentation>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.
