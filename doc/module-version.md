# Version Module

> "UNIVAC: a device, which contained 20,000  
>  vacuum tubes, occupied 1,500 square feet  
>  and weighed 40 tons; there was also a  
>  laptop version weighing 27 tons."  
>  --- Dave Barry

The SCEJSON's version module is the simplest contained in the SCEJSON,
and as such makes an ideal starting point. The version of the SCEJSON in use
can also be of critical importance, so let us tackle it now.

## Version Request Data Object Field Definition

**Example - Request Envelope for Version Module**

```json
{
  "Module" : "Version",
  "Data" : {}
}
```

The `Data` object in the version module request should be empty (e.g `{}`) but present.

## Version Response Data Object Field Definition

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

The Data Object for a response from the version module is defined below:

**Errors[]** - *JArray of JString : {❗required }*

The `Errors[]` field contains an array of JStrings which describe any errors encountered
while handling the request. If the length of the `Errors[]` JArray is zero (0), then the
module processed the request successfully, and the Data object can be further processed
by the calling application for the returned data.

On the other hand, if the length of the `Errors[]` JArray is greater than zero (0), then
this indicates that an error condition has been detected, and the calling application
should *not* process the respons Data object further. In this case, the contents of the
`Errors[]` array will describe the error(s) encountered.

Typical errors include the omission of *❗required* fields, invalid field values, etc.

**Warnings[]** - *JArray of JString : {❗required }*

The `Warning[]` field contains an array of JStrings which describe any warnings generated
by the module handling the request. The most common warnings returned by modules inform
the calling application that the module does not recognize a specified field (which may
help to isolate a field name spelling error in the calling application's code).

**Example - Request and response illustrating warnings when passing unrecognized fields** *(hosted on AWS)*
```json
{
  "Module" : "Version",
  "Data" : {
    "Hello" : "Friend!",
    "How" : "are you?"
  }
}
```

```json
{
  "Result" : 200,
  "Module" : "Version",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
      "Request field 'Data.Hello' not recognized.",
      "Request field 'Data.How' not recognized."
    ],
    "Product" : "SCEJSON",
    "Version" : "2021.07.0",
    "Rootpath" : "/var/task",
    "Copyright" : "(c) 2021 J. L. Sherman and Associates, Inc."
  }
}
```

**Product** - *JString : {❗required }*

For the SCEJSON, the value of this field will always be set to `"SCEJSON"`.

**Version** - *JString : {❗required }*

Holds the version information for the SCEJSON.

The SCEX is versioned with a number formatted
as "YYYY.MM.RR". J. L. Sherman and Associates has chosen to base this number upon dates and
releases, instead of arbitrary version numbers. Specifically, the first part of the version
number (YYYY) refers to the year of the release. The second part of the version number (MM)
refers to the month of the quarterly release. As it currently stands, the quarterly releases
will be made in January, April, July and October. Thus, the value of MM will always be in {1,
4, 7, 10}. Finally, the last part of the version number (RR) signifies the release number for
the given quarter, with zero (0) signifying the first release of the quarter.

Hence, a version number of 2022-01-0 refers to the first release of the SCEJSON for the
January, 2022 quarter. Similarly, a version number of 2021-07-02 refers to the third release
of the SCEJSON in the July, 2021 quarter.

**Rootpath** - *JString : {❗required }*

Contains the fully qualified path for the default directory in which the
SCEJSON will look for a `data/` folder containing setup files, if required
by the request.

**Copyright** - *JString : {❗required }*

Describes the copyright information for the SCEJSON.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [API Overview](api-overview.md) | [SCEJSON Reference Manual](README.md) | [Loan Module](module-loan.md) |
