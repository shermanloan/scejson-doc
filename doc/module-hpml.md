# High Cost Mortgages

> "I can make more generals,  
>  but horses cost money."  
>  --- Abraham Lincoln

TODO

## Hpml Data Object Field Definition

**Example - Request Envelope for Hpml Module**

The following is a sample SCE request for a Hpml calculation:

```json
{
    "Module": "Hpml",
    "Data": {
        "LienType": "first",
        "IsJumbo": false,
        "RateType": "fixed",
        "LockInDate": "2022-03-22",
        "RegZApr": "5.125",
        "TermInYears": "30"
    }
}
```

The fields of the `Data` object supported by a Hpml module request are defined
in alphabetical order below:

## Hpml Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Hpml Module** *(hosted on AWS)*

```json
{
    "Result": 200,
    "Module": "Hpml",
    "Data": {
        "Errors": [],
        "Warnings": [],
        "IsHpml": false,
        "Apor": "4.230",
        "Spread": "1.500",
        "Difference": "-0.605",
        "Date": "2022-03-21"
    }
}
```

The `Data` object for a response from the Hpml module is defined below, in the
order the fields are returned:

### 🟥 Errors

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |

The `Errors[]` field contains an array of Strings which describe any errors encountered
while handling the request. If the length of the `Errors[]` Array is zero (0), then the
module processed the request successfully, and the Data object can be further processed
by the calling application for the returned data.

On the other hand, if the length of the `Errors[]` Array is greater than zero (0), then
this indicates that an error condition has been detected, and the calling application
should *not* process the respons Data object further. In this case, the contents of the
`Errors[]` array will describe the error(s) encountered.

Typical errors include the omission of *🟥 required* fields, invalid field values, etc.

### 🟥 Warnings

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |

The `Warnings[]` field contains an array of Strings which describe any warnings generated
by the module handling the request. The most common warnings returned by modules inform
the calling application that the module does not recognize a specified field (which may
help to isolate a field name spelling error in the calling application's code). Note that
field names which start with "//" will bre treated as comment fields by the SCEJSON, and
no warnings will be generated for these unrecognized fields.

**Example - Request and response illustrating warnings when passing unrecognized fields** *(hosted on AWS)*

```json
{
  "Module" : "Hpml",
  "Data" : {
    "//" : "This is a comment.",
    "Hello" : "Friend!",
    "How" : "are you?"
  }
}
```

```json
{
    "Result": 200,
    "Module": "Hpml",
    "Data": {
        "Errors": [
            "Data.LienType (String) not found.",
            "Data.IsJumbo (Boolean) not found.",
            "Data.RateType (StringFloat) not found.",
            "Data.LockInDate (StringDate) not found.",
            "Data.RegZApr (StringFloat) not found.",
            "Data.TermInYears (StringInt) not found."
        ],
        "Warnings": [
            "Request field Data.Hello (String) not recognized.",
            "Request field Data.How (String) not recognized."
        ]
    }
}   
```



| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [High Cost Mortgages](module-hcm.md) | [SCEJSON Reference Manual](README.md) | |
