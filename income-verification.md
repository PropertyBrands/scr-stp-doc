## Overview
The RIQ Argyle Connector lets api partners fetch a link to Argyle and present to applicants. The Argyle integration facilitates payroll system sync and employment verification after successful login to the payroll provider by applicants. Once authenticated, API partners can call the /init route to create an invitation in Argyle. Currently RIQ only supports payroll sync via argyle, OCR for bank statements and pay stubs will be added in future iterations.

### Environment URLs:
1. sandbox : https://apiqa.residentiq.com


***

## Authentication

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
```JSON
{
    "token": "JWT Bearer Token"
}
```

1. **Failure**: If there are any errors during the authentication process, an error object will be returned in response body.
```JSON
{
    "error": true,
    "message": "Could not verify username and password"
}
```

***

### POST : /incomever/init
**Request**: 
```json
{
    "firstName" : "{Applicant First Name}",
    "lastName" : "{Applicant Last Name}",
    "email" : "{Applicant Email}",
    "phoneNumber": "{Valid Applicant Phone Number}",
    "ssn":"{Applicant SSN}",
    "postback_url" : "{Your Postback URL}",
    "logo_url" : "{url of the png log to use on invitations}",
    "dob":{
        "year": 1971,
        "month" : 4,
        "day":1
    },
    "address": {
        "line1": "{Applicant Street}",
        "city": "{Applicant City}",
        "state": "{Applicant State}",
        "postalCode": "{Applicant Zip}"
    },
    "credentials" : {
        "username": "{ClientLocationUN}",
        "password":"{ClientLocationPW}"
    }
}
```

**Success Response** : 
```json
{
    "success": true,
    "response": {
        "externalId": "{Guid : This is how you will identify the record to RIQ moving forward}",
        "firstName": "{Applicant First Name}",
        "lastName": "{Applicant Last Name}",
        "email": "{Applicant Email}",
        "argyle_Url": "{Argyle URL to present applicant}"
    }
}
```

**Failure Response** : 
```json
{
    "success": false,
    "response": [
        "Valid First Name is Required.",
        "Other Error Messages"
    ]
}
```
## Postbacks
RIQ will post back to the supplied URL with high level information regarding the state of your income verification. This postback will include any Events received from our vendor as well as some high level employment data for all connected accounts.

```json
[
  {
    "invite_sent_date": "2024-10-11T19:22:19.853",
    "first_name": "Test",
    "last_name": "User",
    "invite_sent_to": "test.user@email.com",
    "paystubs_retrieved": 10,
    "employer_name": "Target",
    "employment_status": "terminated",
    "overview": {
      "connection": {
        "status": "error",
        "error_code": "auth_required",
        "error_message": "This user's connection has expired and requires re-authentication.",
        "updated_at": "2024-10-12T19:47:06.840688Z"
      },
      "availability": {
        "paystubs": {
          "status": "synced",
          "updated_at": "2024-10-11T19:44:12.200578Z",
          "available_count": 123,
          "available_from": "2022-04-11T19:42:52Z",
          "available_to": "2024-08-12T00:00:00Z"
        },
        "identities": {
          "status": "synced",
          "updated_at": "2024-10-11T19:42:56.179869Z"
        }
      }
    }
  },
  {
    "invite_sent_date": "2024-10-11T19:22:19.853",
    "first_name": "Test",
    "last_name": "User",
    "invite_sent_to": "test.user@email.com",
    "paystubs_retrieved": 10,
    "employer_name": "DoorDash",
    "employment_status": "inactive",
    "overview": {
      "connection": {
        "status": "connected",
        "error_code": null,
        "error_message": null,
        "updated_at": "2024-10-11T19:46:53.731592Z"
      },
      "availability": {
        "paystubs": {
          "status": "synced",
          "updated_at": "2024-10-11T19:48:51.369949Z",
          "available_count": 143,
          "available_from": "2021-12-13T19:46:53Z",
          "available_to": "2024-09-02T00:00:00Z"
        },
        "identities": {
          "status": "synced",
          "updated_at": "2024-10-11T19:48:51.653044Z"
        }
      }
    }
  }
]
```


## Report Submission
When submitting a report to RIQ that has an income verification tied to it, The supplied ExternalId can be sent as an Additional Item
```xml
<AdditionalItems type="x:income_verification_token">
   <Text>{externalId}</Text>
</AdditionalItems>
```

# RIQ Income Verification Direct
API Partners can also let RIQ run the income verification directly to an applicant via email/sms by supplying the income ver flag in their xml request.

```xml
<AdditionalItems type="x:income_verification">
    <Text>true/false</Text>
</AdditionalItems>
```

# RIQ Income Embed
API Accounts can also send a request to create a user and embed the argyle experience in their application via iframe

### POST : /incomever/init/user
**Request**: 
```json
{
    "firstName" : "{Applicant First Name}",
    "lastName" : "{Applicant Last Name}",
    "email" : "{Applicant Email}",
    "phoneNumber" : "{Applicant Phone}",
    "credentials" : {
        "username": "{ClientLocationUN}",
        "password":"{ClientLocationPW}"
    }
}
```

**Success Response** : 
```json
{
    "success": true,
    "response": {
        "externalId": "{Guid : This is how you will identify the record to RIQ moving forward}",
        "userId": "{Guid : Token to use in your iframe}"
    }
}
```
