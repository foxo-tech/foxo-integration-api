# Foxo Integration API
Docs on using the Foxo integration API

API Endpoints can be created in your Foxo account under Profile > Settings > Integrations. Endpoints created are unique to your account and carry authentication thus should be shared with care.

Testing the API can be done by creating an account on our staging server [app-staging.foxo.com](https://app-staging.foxo.com).

## Postman Demo
Covers the core patient workflows.a
https://www.getpostman.com/collections/657d763ee00af736afbd

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
  // these fields are not case sensitive and syntax style converted IE GivenName = givenname = given_name
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

  // optional
  "medicalProperties": {
      "allergies": [
          {
              "name":"ACE Inhibitors",
              "manifestation":"Rash (Moderate)"
          },
          {
              "name":"Bee stings",
              "manifestation":"Anaphylaxis (Severe)"
          }
      ],
      "history": [
          {
              "name":"Diabetes Mellitus, Type 2",
              "status":"active",
              "date":"2021-05-20"
          },
          {
              "name":"Pregnant",
              "status":"inactive",
              "date":"2021-01-01"
          },
          {
              "name":"Pregnant",
              "status":"inactive",
              "date":"2020-07-01"
          },
          {
              "name":"Plantar wart removal",
              "status":"complete",
              "date":"2020-09-02"
          },
          {
              "name":"Check up",
              "status":"complete",
              "date":"2020-07-01"
          }
      ],
      "medications":[
          {
              "name":"Trandate 200mg Tablet",
              "dosage":"2 At midday without regard to meals prn (100) 5 repeats"
          },
          {
              "name":"Prantal 2% Powder",
              "dosage":"2 In the morning (1x120g)"
          },
          {
              "name":"Pentasa 500mg Tablet, modified release",
              "dosage":"1 In the morning before meals (200) 5 repeats"
          }
      ]
  },

  // optional
  "clinicalProperties": {
      "scan_type": "XRay"
  }

  // only use if the card needs to be sent to one or multiple teams after created
  "sendToTeam": {
    // send to one team only, not needed if `team_uids` is provided
    "team_uid":"b43e9c91-4b19-493b-9548-594266efacf0",
    // send to multiple teams, not needed if `team_uid` is provided
    "team_uids": ["b43e9c91-4b19-493b-9548-594266efacf0", "f669a821-303b-4f8f-a32f-ac6ed75dd566"]
    "case_subject": "Subject for case"
    "case_message": "first message to be sent in the case thread",
    // case webhooks will be triggered on case state change events. it will post a json object to the endpoint with 3 values state, case_key, case_webhook
    "case_webhook": "https://dothis.com/webhook?id=sample123"
  }
}
```
##### 200 Response
```json
{
  "message": "Success, Patient will now be available in Foxo.",
  "patient_uid": "fd0613f3-46cf-4e78-80a5-e67c1d0ab024",
  // if `team_uid` was provided in the request
  "case_key": "b43e9c91-4b19-493b-9548-594266efacf0:32",
  // if `team_uids` was provided in the request
  "case_keys": [
    "b43e9c91-4b19-493b-9548-594266efacf0:32",
    "f669a821-303b-4f8f-a32f-ac6ed75dd566:7"
  ]
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

### Upload and attach media
Add media to cases, patient cards or chat groups.
##### Method: POST
##### Headers
```
X-Foxo-Action: upload-media
```
##### Body
form-data
```
case_key=$case_key(optional)
patient_uid=$patient_uid(optional)
media=(file or files data)
```
##### 200 Response
```json
{
  "message":"Upload Success",
  "data": {
    "message":"Media added."
  }
}
```
