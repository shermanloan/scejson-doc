# API Overview

The SCEJSON offers different functionality in distinct *modules*, which are described below:

| Module | Description |
| ------ | ----------- |
| [Version](module-version.md) | A simple module which returns the version of the SCEJSON in use, along with a few other userful data fields. |
| [Loan](module-loan.md)    | The Loan module allows you to build almost any kind of loan you can imagine - one or more advances, adjustable interest rates, overlapping payment schedules, etc.  |
| [Apr](module-apr.md)     | Our APR calculation and verification module. Note that APRs are returned in the Loan module. However, this module is typically used to calculate and/or verify APRs for loan computed by another calculation methodology. |
| [Hcm](module-hcm.md)     | A compliance module which implements the logic necessary to determine if a specified mortgage is considered High Cost, per [§ 1026.32 Requirements for high-cost mortgages](https://www.consumerfinance.gov/rules-policy/regulations/1026/32/). |
| [Hpml](module-hpml.md)    | A compliance module which implements the logic necessary to determine if a specified mortgage is considered a Higher Priced Mortgage Loan, per [§ 1026.35 Requirements for higher-priced mortgage loans](https://www.consumerfinance.gov/rules-policy/regulations/1026/35/).|

Each of the modules listed above has a different format for its input request and output response, which are detailed in the chapters dedicated to each module.  However, the general (or top-level) format of the data sent to the SCEJSON needs to be consistent for correct parsing, and is called the "request envelope".

## The Request Envelope Object

The request envelope is simply a JSON object which identifies the requested `Module`, along with a `Data` object that contains the module specific data fields. The request `Data` object for each module is thoroughly documented in the chapter dedicated to that module.

**Example - Request Envelope for Version Module**

```json
{
  "Module" : "Version",
  "Data" : {}
}
```

**Example - Request Envelope for Loan Module**
```json
{
  "Module" : "Loan",
  "Data" : {...}
}
```

## Request Envelope Object Field Definitions

**Module** - *JString : (Apr, Hcm, Hpml, Loan, Version) {❗required }*

The value of the `Module` field determines which module within the SCEJSON is being requested.

**Data** - *JObject : (see documentation for the desired module) {❗required }*

The `Data` field contains the module specific request data, and differs depending upon the desired `Module`. Please reference the chapter dedicated to the module for the required format of the `Data` object.

## Calling the SCEJSON API

Once the request envelope has been created by your application, you will need to send it to the SCEJSON using the appropriate method. Typically, this is done via an HTTP POST request with the body of the request consisting of the request envelope described above. You may also be required to include a specific HTTP header specifyng your API key. These details will be provided by J. L. Sherman and Associates, Inc. and will depend upon how you are accessing the SCEJSON.

Once the request envelope has been successfully sent to the SCEJSON, the SCEJSON will attempt to process the request and will then return a response envelope with the results of the request.

## The Response Envelope Object

The response envelope is simply a JSON object which identifies the numeric `Result` of the call to the SCEJSON, a response `Module`, along with a `Data` object that contains the response module specific data fields. The response `Data` object for each module is thoroughly documented in the chapter dedicated to that module.

**Example - Response Envelope for Version Module** *(hosted on AWS)*

```json
{
  "Result" : 200,
  "Module" : "Version",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Product" : "SCEJSON",
    "Version" : "2021.07.0",
    "Rootpath" : "/var/task",
    "Copyright" : "(c) 2021 J. L. Sherman and Associates, Inc."
  }
}
```

## Response Envelope Object Field Definitions

**Result** - *JInteger : (200, 400, 403) {❗required }*

The `Result` field indicates the status of the API request. Please see the following table for what the API Result code mean. A successful call will result in a Result field value of 200 (API_OK). Any other value indicates that an error was encountered.

| Result | Symbol | Description |
| :----: | :----: | :---------- |
| 200 | API_OK | Indicates that the SCEJSON was able to successfully dispatch the request to the specified module. Note that this does not guarantee a successful response from the module (for that, check response Data.Errors[]). |
| 400 | API_ERR_BADREQUEST | One of the following occurred: (i) an exception was encountered while attempting to parse the request, (ii) could not parse the request into a valid JSON object, (iii) no `Module` field of type JString was found in the envelope, or (iv) no `Data` field of type JObject was found. An API Error response will be returned to the calling application. |
| 403 | API_ERR_NOTFOUND | The `Module` specified was not found. An API Error response will be returned to the calling application. |

**Module** - *JString : (Apr, Error, Hcm, Hpml, Loan, Version) {❗required }*

The value of the `Module` field determines which module within the SCEJSON generated the response. If the value `Result` field is `200` (API_OK), then the value of `Module` will be the same as the value of the `Module` field in the request envelope. If the value `Result` field is other than `200` (API_OK), then the SCEJSON has encountered an API ERROR condition, and the value of the this field will be set to `"Error"` to indicate that the `Data` field should be parsed as an [API Error response](#API-Error-Response-Data-Object-Field-Definition).

**Data** - *JObject : (see documentation for the desired module) {❗required }*

The `Data` field contains the module specific response data, and differs depending upon the returned `Module`. Please reference the chapter dedicated to the module for the required format of the `Data` object.

## API Error Response - Data Object Field Definition

TODO

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Introduction](introduction.md) | [SCEJSON Reference Manual](README.md) | [Version Module](module-version.md) |
