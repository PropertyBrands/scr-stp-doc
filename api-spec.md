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
1. sandbox : https://apiqa.residentiq.com
2. production : https://api.residentiq.com

# Note :
1. Please use ssns that corresponds with test accounts : 333221111,111223333.
2. Report Urls returned from the API are transient and only live for 5 minutes. this can be overriden for a given location if needed.

***
### POST : /api/partner
The standard post route accepts a standard xml payload in the request body and will return a success or failure response as a 200 http code. Accepted header would include application/xml.

[Sample Payload](https://github.com/PropertyBrands/scr-stp-doc/blob/master/partner-post-sample.xml)

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
The link route will let API Partners group applicants after they have already been submitted to STP. STP takes a simple xml payload with the a base OrderId and the other RIQ report numbers to include in the group:
1. Applicant1 Id : The report number of the applicant to group
2. Additional Report Ids : The report numbers of the applicants applicant 1 should be linked with.

### Examples:
1. **Request**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{Applicant 1 Report Number}</OrderId>
  <AdditionalItems type="x:link_order"><Text>{Applicant 2 Report Number}</Text></AdditionalItems></BackgroundSearchPackage>
  <AdditionalItems type="x:link_order"><Text>{Applicant 3 Report Number}</Text></AdditionalItems></BackgroundSearchPackage>
  <AdditionalItems type="x:link_order"><Text>{Applicant 4 Report Number}</Text></AdditionalItems></BackgroundSearchPackage>
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

### POST : /api/partner/unlink/many
The unlink route will remove applicants with the supplied Ids from its current group and update the group calculations for all remaining members. RIQ does not support single applicant groups and will automatically clear a group from the system if only one report remains in it.

### Examples:
1. **Request**: If the payload passes validation api partner will receive an immediate response indicating the record has been received and is pending.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status">
    <OrderId>{STP Report Number To Decouple}</OrderId>
 </BackgroundSearchPackage>
  <AdditionalItems type="x:link_order"><Text>{Applicant 2 Report Number to unlink}</Text></AdditionalItems></BackgroundSearchPackage>
  <AdditionalItems type="x:link_order"><Text>{Applicant 3 Report Number to unlink}</Text></AdditionalItems></BackgroundSearchPackage>
  <AdditionalItems type="x:link_order"><Text>{Applicant 4 Report Number to unlink}</Text></AdditionalItems></BackgroundSearchPackage>
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

### POST : /api/partner/packages
The Packages Route will let API Partners pull a list of available products for a set of client location credentials

### Examples:
1. **Request**: If the payload passes validation api partner will receive a product list response.
```xml
<?xml version="1.0"?>
<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundSearchPackage action="status" />
</BackgroundCheck>
```
2. **Response-Success**: 
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
    <ProductsAvailable>
        <Product>Product 1</Product>
        <Product>Product 2</Product>
        <Product>Product 3</Product>
    </ProductsAvailable>
  </BackgroundReportPackage>
</BackgroundReports>
```
3. **Response-Error**: Returns a standard Error response with description.

### POST : /api/partner/decision
The decision route will allow partners to set a property decision for a specific screening. Allowed property decisions are Decline, Conditional-Low, Conditional-High, and Accept. The property decision will be used when STP generates an adverse action letter on behalf of the API partner.

### Examples
1. **Request**: Similar to the status route, the API will look for a "decision" action on the BackgroundSearchPackage element, followed by a child PropertyDecision element:
```xml
<?xml version="1.0"?>
	<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
		<BackgroundSearchPackage action="decision">
			<OrderId>{STP Internal report Number}</OrderId>
			<PropertyDecision>{Accept / Conditional-High / Conditional-Low / Decline}</PropertyDecision>
		</BackgroundSearchPackage>
	</BackgroundCheck>
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

### POST : /api/partner/statistics
The statistics route will pull all applicants under a client and provide statistics on each. The statistics provided include Eviction and Bankruptcy data coupled with the applicants credit score. If a credit score is not on file in RIQ the RIQ calculated score is returned in the Credit Score element. There is a maximum 60 day window for collecting stats. if the requested timeline exceeds this a standard error is thrown with message indicating the issue.

### Examples
1. **Request**: The statistics route expects valid location credentials(from which to infer the client ID) and a Start/End date pair (MM-DD-YYYY) embeded in a Stats Element for lookups.
```xml
<?xml version="1.0"?>
	<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
		<BackgroundSearchPackage action="decision">
			<Stats>
				<StartDate>04-01-2020</StartDate>
				<EndDate>05-01-2020</EndDate>
			</Stats>
		</BackgroundSearchPackage>
	</BackgroundCheck>
```
2. **Response Success**: If the API call passes validation, the API will return collection of all reports for the client found within the time window based on date created in STP.
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <Stats>
        <Stat>
            <ReportNumber>12345</ReportNumber>
            <CreditScore>31</CreditScore>
            <Evictions Count="1">
                <Eviction>
                    <FileDate>2/13/2019</FileDate>
                </Eviction>
            </Evictions>
            <Bankruptcies Count="1">
                <Bankruptcy>
                    <Type>BankruptcyChapter7</Type>
                    <CourtName>US BKPT CT CO DENVER</CourtName>
                    <Status>Discharged</Status>
                </Bankruptcy>
            </Bankruptcies>
        </Stat>
    </Stats>
</BackgroundReports>
```

3. **Response-Error**: Returns a standard Error response with description.

### POST : /api/partner/document/upload
The Document upload route will allow partners to submit additional documents for a report after it has been sent to screening.

### Examples
1. **Request**: Similar to the status route, the API will look for a "decision" action on the BackgroundSearchPackage element, followed by a child PropertyDecision element:
```xml
<?xml version="1.0"?>
	<BackgroundCheck userId="{ClientLocationUN}" password="{ClientLocationPW}">
		<BackgroundSearchPackage action="decision">
			<OrderId>{STP Internal report Number}</OrderId>
			 <SupportingDocumentation>
				<Document>
				    <OriginalFileName>Doc1.png</OriginalFileName>
				    <FileType>3</FileType>
				    <EncodedContent>{BASE64 STRING}</EncodedContent>
				</Document>
				<Document>
				    <OriginalFileName>Doc2.pdf</OriginalFileName>
				    <FileType>3</FileType>
				    <EncodedContent>{BASE64 STRING}</EncodedContent>
				</Document>
			    </SupportingDocumentation>
		</BackgroundSearchPackage>
	</BackgroundCheck>
```
2. **Response Success**: If the API call passes validation, the API will return the following payload indicating if the upload was successful
```xml
<?xml version="1.0"?><BackgroundReports userId="{ClientLocationUN}" password="{ClientLocationPW}">
  <BackgroundReportPackage>
    <OrderId>{STP Internal Report Number}</OrderId>
    <VerificationDocumentation>Document(s) uploaded for this report.</VerificationDocumentation>
  </BackgroundReportPackage>
</BackgroundReports>
```

3. **Response-Error**: Returns a standard Error response with description.
