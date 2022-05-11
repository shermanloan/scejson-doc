# Higher Priced Mortgage Loans (HPML)

> "Some men can live up to their loftiest ideals  
>  without ever going higher than a basement."  
>  --- Theodore Roosevelt

Higher Priced Mortgage Loans are defined as consumer-purpose, closed-end loans
secured by a consumer's principal dwelling that have an annual percentage rate
(APR) equal to or greater than the Average Prime Offer Rate (APOR) by 1.5
percentage points for first-lien loans, or 3.5 percentage points for
subordinate-lien loans for a comparable transaction.

The APOR is published weekly by the federal government, and the two files (one
for fixed rates and one for adjustable rates) are available for download at the
following web site: https://ffiec.cfpb.gov/tools/rate-spread. Please note that
it is necessary that these files be updated weekly, or else the SCE will not be
able to determine if a loan is a HPML for loans whose lock in dates fall outside
the range of dates provided in the APOR files. If you are using a version of the
SCE that is hosted by J. L. Sherman and Associates, Inc., then the APOR files
will be automatically updated weekly for your use.

## Hpml Request Data Object Field Definition

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

### 🟦 DataPath

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | see below | see below |

If this field is set, the SCE will look for a data folder containing the
APOR files in the path specified. Note that the APOR files must be
named as follows: `YieldTableFixed.txt` and `YieldTableAdjustable.txt`.

If this field is not set, the SCE will attempt to locate the file in the
default data directory. Note that the AWS hosted version of the SCE API server
places the necessary files in the correct default directory, and hence
specifying this field is not required.

Since the APOR files are global in nature, you only need to have
one copy of these files available even if your installation hosts multiple data
paths for different clients. The APOR files are identical to those needed for
the Hcm module, and thus a single location containing the APOR files can be
used for both modules.

### 🟥 IsJumbo

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | yes | true, false | n/a |

Effective June 1, 2013, first-lien loans that exceed the maximum principal 
obligation eligible for purchase by Freddie Mac (i.e., a "jumbo" loan)
have a rate spread of 2.5% instead of the normal first-lien value of 1.5%.

Note that the value of this field will be ignored for subordinate-lien
loans.

### 🟥 LienType

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | First, Subordinate | n/a |

The type of lien determines the amount over the APOR at which time a loan's
Regulation Z APR is considered a HPML. For first-lien loans, that value is 1.5%
(or 2.5% for first-lien jumbo loans as of 6/01/2013); for subordinate-lien
loans, that value is 3.5%.

### 🟥 LockInDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

The value of this field holds the date on which the interest rate of the loan
was locked in. All dates must be in the form of YYYY-MM-DD, and be 10 characters
long.

The lock in date is used to lookup the APOR rate from the appropriate file. Each
row of the file begins with a date, and the row which will be used will have a
date which is either on the lock in date, or does not preceed it by more than 6
days. Since the rates are updated weekly, there should never be more than six
days between the date of the row used and the specified lock in date.

If the SCE can not find a row in the APOR file which is appropriate for use with
the specified lock in date, then an error message will be returned informing the
calling application.

### 🟥 RateType

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Adjustable, Fixed | n/a |

The APOR is looked up in one of two tables, determined by the loan's rate type.
If the loan uses a fixed rate, then the SCE will look-up the APOR from the
`YieldTableFixed.txt` file. If the loan uses a variable or adjustable rate, then
the SCE will look-up the APOR from the `YieldTableAdjustable.txt` file.

### 🟥 RegZApr

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - % | n/a |

The value of this field holds the computed Regulation Z APR for the loan we are
checking. The Regulation Z APR should be expressed as a percentage.

### 🟥 TermInYears

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number | n/a |

The `TermInYears` field holds the term of the loan, in whole years. Valid terms
range from 1 year to 50 years, inclusive. If the loan's `RateType` is
`Adjustable`, then the value of this element represents the initial, fixed rate
term of the adjustable rate loan.

## Hpml Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Hpml Module**

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

**Example - Request and response illustrating warnings when passing unrecognized fields**

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

### 🟥 IsHpml

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

The value of this field will either be true or false, and will indicated whether
or not the submitted loan is a higher priced mortgage loan.

### 🟥 Apor

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - % |

The value of this field contains the Average Prime Offer Rate for the submitted
loan. This value is retrieved from the appropriate APOR file (fixed or
adjustable).

### 🟥 Spread

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The spread value depends upon the type of lien. First-lien loans have a spread
of 1.5, and subordinate-lien loans have a spread of 3.5.

### 🟥 Difference

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The value of this field returns the difference between the submitted Regulation
Z APR, and the APOR plus the spread. If the difference is greater than or equal
to zero (which means that the Reg. Z APR is greater than or equal to the APOR
plus spread), then the loan is a HPML.
 
### 🟥 Date

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The Date field contains the date on which the APOR data was effective. As an
example, if the lock in date was 2009-12-22, then the date returned should be
2009-12-21 (if the APOR rates files are up to date).


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [High Cost Mortgages](module-hcm.md) | [SCEJSON Reference Manual](README.md) | [Certificates of Deposit](module-cd.md) |
