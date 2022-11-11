# APY Calculation

> "To strive, to seek, to find,  
>  and not to yield."  
>  --- Alfred Lord Tennyson

The APY Calculation provides an independent verification for the Federal Truth
in Savings Act's Annual Percentage Yield on a deposit instrument. By supplying
key pieces of data, the SCE will pass back an exhaustive analysis of the APY for
the deposit in question.

## Apy Request Data Object Field Definition

**Example - Request Envelope for Apy Module**

The following is a sample SCE request for an Apy calculation:

```json
{
  "Module": "Apy",
  "Data": {
    "Compounding": "Daily",
    "Decimals": "4",
    "Statement": "Monthly",
    "Principal": "1000.00",
    "Interest": "61.68",
    "InvestDate": "2014-04-16",
    "MaturityDate": "2015-04-16"
  }
}
```

The fields of the `Data` object supported by a Apy module request are defined
in alphabetical order below:

### 🟦 Compounding

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Annual, Semiannual, Quarterly, Bimonthly, Monthly, Biweekly, Weekly, Daily | Daily |

The `Compounding` field specifies the frequency that interest is compounded. If
the deposit compounds continuously, then specify a value of `Daily`.

### 🟦 Decimals

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 3, 4, 5 | 3 |

The number of decimal places of accuracy for the disclosed APY
is determined by the value of this field.

### 🟥 Interest

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

The total interest earned on the principal

(a) for the entire duration of the deposit if the compounding frequency is
greater than or equal to the statement disclosure frequency, or

(b) for the statement period if statements are disclosed more frequently than
the deposit is compounded.

### 🟦 InvestDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD | n/a |

The value of this optional field contains the date on which interest accrual
begins. If statements are disbursed more frequently than interest is compounded,
then you may omit this element. As an example, if interest is compunded annually
and statements are disclosed monthly, then this element may be omitted.

### 🟦 MaturityDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD | n/a |

This optional field holds the date on which the deposit matures. If statements
are disbursed more frequently than interest is compounded, then you may omit
this element. As an example, if interest is compunded annually and statements
are disclosed monthly, then this element may be omitted.

### 🟥 Principal

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

The value of this field specifies the total amount earning interest.

### 🟦 Statement

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Annual, Semiannual, Quarterly, Bimonthly, Monthly | Monthly |

The `Statement` field determines the frequency that statements are provided to
the customer by the deposit institution.

## Apy Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Apy Module**

```json
{
  "Result": 200,
  "Module": "Apy",
  "Data": {
    "Errors": [],
    "Warnings": [],
    "Apy": "6.1680%",
    "Formula": "General",
    "CompoundingFreq": "Daily",
    "CompoundingDays": "1",
    "StatementFreq": "Monthly",
    "StatementDays": "30",
    "InvestDate": "2014-04-16",
    "MaturityDate": "2015-04-16",
    "DaysToMaturity": "365",
    "Principal": "1000.00",
    "Interest": "61.68"
  }
}
```

The `Data` object for a response from the Apy module is defined below, in the
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

**Example - Request and response illustrating warnings when passing unrecognized fields**

```json
{
  "Module" : "Apy",
  "Data" : {
    "//" : "This is a comment.",
    "Hello" : "Friend!",
    "How" : "are you?"
  }
}
```

```json
{
  "Result" : 200,
  "Module" : "Apy",
  "Data" : {
    "Errors" : [
      "Data.Interest (StringFloat) not found.",
      "Data.Principal (StringFloat) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}       
```

### 🟥 Apy

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The APY, as defined in Appendix A to Part 1030 of Truth in Savings, and computed
by the SCE.

### 🟥 Formula

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | General, Special |

The `Formula` field describes the formula used to compute the APY. The `General`
formula is described in Appendix A to Part 1030, Part I Section A. The `Special`
forumula is described in Appendix A to Part 1030, Part II Section B. Note that
the Special formula is only used when statements are sent more frequently than
the deposit is compounded.

### 🟥 CompoundingFreq

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Daily, Weekly, BiWeekly, Monthly, BiMonthly, Quarterly, SemiAnnual, Annual |

The frequency that interest is compounded.

### 🟥 CompoundingDays

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The value of this field specifies the number of days in each compounding period.

### 🟥 StatementFreq

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Monthly, BiMonthly, Quarterly, SemiAnnual, Annual |

The value of this field specifies the frequency that statements are
sent to the customer by the deposit institution.

### 🟥 StatementDays

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The number of days in each statement period.

### 🟥 Term

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The `Term` is the number of unit periods (as specified in the `TermUnits` field)
between the deposit date and the CD's maturity date.

### 🟦 InvestDate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | YYYY-MM-DD |

Returns the investment date which was passed to the SCE. If no investment date
was specified, then this field will not appear in the output.

### 🟦 MaturityDate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | YYYY-MM-DD |

Returns the maturity date which was passed to the SCE. If no maturity date
was specified, then this field will not appear in the output.

### 🟦 DaysToMaturity

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | YYYY-MM-DD |

The number of days between the investment and maturity dates.

### 🟥 Principal

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The value of this field specifies the total amount earning interest.

### 🟥 Interest

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field holds the value of the `Interest` field passed into the SCE.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Higher Priced Mortgage Loans (HPML)](module-hpml.md) | [SCEJSON Reference Manual](README.md) | [Certificates of Deposit](module-cd.md) |
