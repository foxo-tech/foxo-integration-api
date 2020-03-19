# Foxo Integration API
Docs on using the Foxo integration API

API Endpoints can be created in your Foxo account under Profile > Settings > Integrations. Endpoints created are unique to your account and carry authentication thus should be shared with care.

Testing the API can be done by creating an account on our staging server [app-staging.foxo.com](https://app-staging.foxo.com).

## Postman Demo
Covers the core patient workflows.
https://www.getpostman.com/collections/cecabfb648419ce2055b

## Basics
Since there is a single endpoint actions are determined by method and `X-Foxo-Action`.

A version header may be passed (`X-Foxo-Version`) but is ignored for now, since we will aim to be as backwards compatible as possible with all changes to the api.

## Actions

### Create Patient
This will create a patient in Foxo and attach the endpoint owner as the primary carer.
##### Method: POST
##### Headers
```
X-Foxo-Action: create-patient
```
##### Body
JSON payload
```javascript
{
  // everything is optional except GivenName & FamilyName fields or FirstName & LastName fields respectively
  "Title": "Ms",
  "GivenName": "Jane",
  "FamilyName": "Doe",
  "OtherNames": null,
  "DateOfBirth": "1980-01-01T00:00:00.0000000",
  "Sex": "Female",
  "HomePhone": "0812344321",
  "WorkPhone": null,
  "MobilePhone": null,
  "EmailAddress": null,
  "Location": "123 fake street",
  "Suburb": "Osborne Park",
  "PostCode": "6017",
  "State": "WA",
  "Country": "Australia",
  "ClinicalInfo": "Clinical info to be attached to patient"

  // Idenifier fields are wildcards will use last defined, and can be omitted
  "[a-z0-9]*Identifier": "1234",

  // optional will over write any other identifer fields set.
  "KarismaIdentifierType": "Internal",
  "WorkSiteIdentifierType": "Logan UR",
  "Identifiers": [
      {
          "Type": "Logan UR",
          "Value": "1234"
      },
      {
          "Type": "Redlands UR",
          "Value": "4321"
      },
      {
          "Type": "Internal",
          "Value": "5678"
      },
      {
          "TypeN": "Name",
          "ValueN": "Value"
      }
  ],

  // only use if the card needs to be sent to a team after created
  "sendToTeam": {
    "team_uid":"b43e9c91-4b19-493b-9548-594266efacf0",
    "case_subject": "Subject for case"
    "case_message": "first message to be sent in the case thread"
  }
}
```
##### 200 Response
```json
{
  "message": "Success, Patient will now be available in Foxo."
}
```
##### 400 Response
```json
{
    "message": "Foxo API Failed to parse request. Please check API endpoint and data before trying again."
}
```
### List Available Teams
Teams allow patients to be managed by a group of users. Patients can be forwarded onto a team automatically.
##### Method: GET
##### Headers
```
X-Foxo-Action: list-teams
```
##### 200 Response
```json
{
  "message": "Teams a available to user.",
  "data": [
    {
      "team_uid": "b43e9c91-4b19-493b-9548-594266efacf0",
      "name": "Concierge Service Team"
    }
  ]
}
```
##### 400 Response
```json
{
    "message": "Unable to find teams available to this user."
}
```
