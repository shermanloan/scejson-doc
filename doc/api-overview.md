# API Overview

> "It cannot be denied that he (*Pierre de Fermat*) has had many exceptional ideas,  
>  and that he is a highly intelligent man. For my part, however, I have always been  
>  taught to take a broad overview of things, in order to be able to deduce from  
>  them general rules, which might be applicable elsewhere."  
>                                         --- Rene Descartes

The SCEJSON offers different functionality in distinct *modules*, and each
module is documented separately in the reference manual. Each module has a
different format for its input request and output response, which is detailed in
the chapter dedicated to that module.  However, the general (or top-level)
format of the data sent to the SCEJSON needs to be consistent for correct
parsing, and is called the "request envelope".

## The Request Envelope Object

The request envelope is simply a JSON object which identifies the requested
`Module`, along with a `Data` object that contains the module specific data
fields. The request `Data` object for each module is thoroughly documented in
the chapter dedicated to that module.

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

### 🟥 Module

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Apr, Hcm, Hpml, Loan, Version | n/a |

The value of the `Module` field determines which module within the SCEJSON is
being requested.

### 🟥 Data

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `Data` field contains the module specific request data, and differs
depending upon the desired `Module`. Please reference the chapter dedicated to
the module for the required format of the `Data` object.

## Calling the SCEJSON API

Once the request envelope has been created by your application, you will need to
send it to the SCEJSON using the appropriate method. Typically, this is done via
an HTTP POST request with the body of the request consisting of the request
envelope described above. You may also be required to include a specific HTTP
header specifyng your API key. These details will be provided by J. L. Sherman
and Associates, Inc. and will depend upon how you are accessing the SCEJSON.

Once the request envelope has been successfully sent to the SCEJSON, the SCEJSON
will attempt to process the request and will then return a response envelope
with the results of the request.

## The Response Envelope Object

The response envelope is simply a JSON object which identifies the numeric
`Result` of the call to the SCEJSON, a response `Module`, along with a `Data`
object that contains the response module specific data fields. The response
`Data` object for each module is thoroughly documented in the chapter dedicated
to that module.

**Example - Response Envelope for Version Module**

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

### 🟥 Result

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Number - Integer | yes | 200, 400, 403 |

The `Result` field indicates the status of the API request. Please see the
following table for what the API Result code mean. A successful call will result
in a Result field value of 200 (API_OK). Any other value indicates that an error
was encountered.

| Result | Symbol | Description |
| :----: | :----: | :---------- |
| 200 | API_OK | Indicates that the SCEJSON was able to successfully dispatch the request to the specified module. Note that this does not guarantee a successful response from the module (for that, check response Data.Errors[]). |
| 400 | API_ERR_BADREQUEST | One of the following occurred: (i) an exception was encountered while attempting to parse the request, (ii) could not parse the request into a valid JSON object, (iii) no `Module` field of type String was found in the envelope, or (iv) no `Data` field of type Object was found. An [API Error response](#api-error-response) will be returned to the calling application. |
| 403 | API_ERR_NOTFOUND | The `Module` specified was not found. An [API Error response](#api-error-response) will be returned to the calling application. |

### 🟥 Module

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Apr, Hcm, Hpml, Loan, Version |

The value of the `Module` field determines which module within the SCEJSON
generated the response. If the value of the `Result` field is `200` (API_OK),
then the value of `Module` will be the same as the value of the `Module` field
in the request envelope. If the value `Result` field is something other than
`200` (API_OK), then the SCEJSON has encountered an API ERROR condition, and the
value of the this field will be set to `"Error"` to indicate that the `Data`
field should be parsed as an [API Error response](#api-error-response).

### 🟥 Data

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `Data` field contains the module specific response data, and differs
depending upon the returned `Module`. Please reference the chapter dedicated to
the module for the required format of the `Data` object.

## API Error Response

As mentioned above, if the SCEJSON was not able to successfully dispatch a
request due to one of several possible error conditions, an API Error Response
will be returned. Here are a few examples of SCEJSON requests that generate API
Error Responses

**Example - Request with invalid JSON and resulting response**
```json
{
  "Module" : "FooBar"
  "Data" : {}
}
```
```json
{
  "Result" : 400,
  "Module" : "Error",
  "Data" : {
    "Errors" : [
      "Exception while parsing JSON input. \"EJSONParser: Error at line 3, Pos 8:Expected , or ], got token \"Data\".\""
    ]
  }
}
```

**Example - Request with no Module field and resulting response**
```json
{
  "Data" : {}
}
```
```json
{
  "Result" : 400,
  "Module" : "Error",
  "Data" : {
    "Errors" : [
      "Bad API request. No 'Module' string field present."
    ]
  }
}
```

**Example - Request with no Data field and resulting response**
```json
{
  "Module" : "Version"
}
```
```json
{
  "Result" : 400,
  "Module" : "Error",
  "Data" : {
    "Errors" : [
      "Bad API request. No 'Data' object field present."
    ]
  }
}
```

**Example - Request with invalid Module and resulting response**
```json
{
  "Module" : "FooBar",
  "Data" : {}
}
```
```json
{
  "Result" : 403,
  "Module" : "Error",
  "Data" : {
    "Errors" : [
      "API not found. No Module named 'FooBar' found."
    ]
  }
}
```

## API Error Response Data Object Field Definition

The `Data` object for an API Error Response contains a single field which returns a description of the API Error.

### 🟥 Errors

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |

The `Errors[]` field contains an array of Strings which describe the error
conditions which were encountered. For an API Error Response, the length of the
`Errors[]` Array should always be one (1).

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Introduction](introduction.md) | [SCEJSON Reference Manual](README.md) | [Account](module-account.md) |
