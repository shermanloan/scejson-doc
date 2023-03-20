# User Interface Module

> "I am suggesting that we recognize that  
>  in network and interface research there  
>  is something as profound (and potentially  
>  wild) as Artificial Intelligence."  
>  --- Vernor Vinge

The SCE's user interface request allows the user to determine how an
*account* has been set up for the SCE in a given directory path, and indicates
which protection products are available and what options are available for each
active product. An account is defined as a set of setup files associated with
an account number which starts at 1 and has a maximum value of 9,999. The setup
files for a given account number configure the SCE to compute loans with
protection in a particular manner.

To get the names of all accounts configured in a given directory path, please
see the [Account Module](module-account.md).

## User Interface Request Data Object Field Definition

**Example - Request Envelope for User Interface Module**

```json
{
  "Module" : "Ui",
  "Data" : {
    "Account" : "1",
    "Compact" : false,
    "DataPath" : "/etc/sce/"
  }
}
```

The fields of the `Data` object supported by a User Interface module request are defined
in alphabetical order below:

### 🟦 Account

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1 |

The `Account` field specifies the numeric setup file account number that will be
parsed to determine the user interface related response fields. Each account is
numbered from 1 to 9,999, and each account corresponds to a set of setup file
which define numerous settings which may affect the loan calculation, such as
the accrual method, payment protection methods / rates / caps, etc.

### 🟦 Compact

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to a value of `true`, then the SCE will only return
response fields if the value of the field is different from the default
value.

### 🟦 DataPath

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Text | default data path |

If this field is set, the SCE will look for a `/data` folder containing the
setup files in the path specified. Thus, if the `DataPath` is set to `/etc/sce`
(as above), the SCE will look for the setup files in `/etc/sce/data`.

If the calling application wishes to specify the data directory path in its
entirety (e.g. the calling app does not want the SCE to append `/data` to the
provided path), then simply terminate the specified `DataPath` with an asterisk
(`*`). Thus, if the `DataPath` is set to `/etc/sce/bank1/*`, the SCE will look for
the setup files in `/etc/sce/bank1/`.

If this field is not set, the SCE will attempt to located the `/data` folder in
the default data directory path location, which can be retrieved using the
`Version` module.

This field is useful if you wish to use only a single installation of the SCE,
but have many different setup file groupings. By specifying a different
`DataPath` for each grouping, you can easily separate the groups from one
another instead of grouping them all together in a single directory.

## User Interface Response Data Object Field Definition

**Example - Response Envelope for User Interface Module**

```json
{
  "Result" : 200,
  "Module" : "Ui",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Ls" : {
      "Status: " : "Loaded",
      "LoanTypes" : {
        "ARM" : true,
        "Balloon" : true,
        "Construction" : false,
        "Equal" : true,
        "IntOnly" : true,
        "PrinPlus" : true,
        "SinglePay" : true,
        "SkipsIrregs" : true,
        "SkipsPickups" : true
      },
      "Accrual" : {
        "Code" : "320",
        "LastDay" : false,
        "InterestRound" : "Nearest",
        "PaymentRound" : "Up",
        "SinglePayDivisor" : "365"
      }
    },
    "Cl" : {
      "Active" : true,
      "AllowOn" : {
        "ARM" : false,
        "Balloon" : false,
        "Equal" : true,
        "IntOnly" : false,
        "PrinPlus" : false,
        "SinglePay" : false,
        "Irregs" : false
      },
      "IsDP" : false,
      "Method" : "TrueMOB",
      "TrueMOB" : {
        "Abbrev" : "CL",
        "Title" : "Credit Life",
        "Prompts" : {
          "SingleOnCoborrower" : true
        }
      }
    },
    "Ah" : {
      "Active" : true,
      "Abbrev" : "AH",
      "Title" : "Disability",
      "AllowOn" : {
        "ARM" : false,
        "BalloonGross" : false,
        "BalloonMobNet" : false,
        "Equal" : true,
        "IntOnly" : false,
        "PrinPlus" : false,
        "SinglePay" : false,
        "Irregs" : false
      },
      "IsDP" : false,
      "IsPackagedDP" : false,
      "Method" : "TrueMOB",
      "Prompts" : {
        "CoverageBenefit" : false,
        "CoverageTermGross" : false,
        "CoverageTermMobNet" : true,
        "Joint" : true,
        "SingleOnCoborrower" : true
      },
      "Rules" : {
        "MustHaveLife" : false,
        "PPYMoreThan12" : true,
        "PPYLessThan12" : false
      },
      "Tables" : [
        "14 Retro"
      ]
    },
    "Iu" : {
      "Active" : false
    },
    "Pp" : {
      "Active" : false
    }
  }
}
```

The `Data` object for a response from the User Interface module is defined below:

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
  "Module" : "Ui",
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
  "Module" : "Ui",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ],
    ...
  }
}   
```

### 🟥 Ls

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The `Ls` object returns information on general loan configuration setup file
data.

<details>
<summary><b>Ls fields</b></summary>

---

🟥 **Ls.Status**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Discloses the account number associated with this account.

🟥 **Ls.LoanTypes**

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `LoanTypes` object returns fields which indicate if a loan of a given
repayment schedule is allowed, according to the loan setup file associated with
the account.

<details>
<summary><b>LoanTypes fields</b></summary>

---

🟦 **LoanTypes.ARM**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if adjustable rate loans available for this account.

🟦 **LoanTypes.Balloon**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if balloon loans available for this account?

🟦 **LoanTypes.Construction**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if construction loans available for this account.

🟦 **LoanTypes.Equal**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if equal payment loans available for this account.

🟦 **LoanTypes.IntOnly**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if interest only loans available for this account.

🟦 **LoanTypes.PrinPlus**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if fixed principal plus interest loans available for
this account.

🟦 **LoanTypes.SinglePay**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if single payment notes available for this account.

🟦 **LoanTypes.SkipsIrregs**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if loans with skipped and specified irregular payments
available for this account.

🟦 **LoanTypes.SkipsPickups**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Returns a value of `true` if loans with skipped and pickup payments available
for this account.

---
</details>

🟥 **Ls.Accrual**

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `Accrual` object returns information regarding how various interest accrual
settings have been configured in the loan setup file associated with the
account.

<details>
<summary><b>Accrual fields</b></summary>

---

🟥 **Accrual.Code**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| String | true | see table | n/a |

The method of interest accrual configured for the account is returned in this field.
All valid accrual codes are described in the table below.

| Code | Description |
| :---: | :--- |
| 100 | Actuarial (using Federal calendar)  |
| 101 | Actuarial w/ Toyota Motor Credit calendar  |
| 201 | Unit period simple w/ 360 day divisor  |
| 202 | Unit period simple w/ 365 day divisor  |
| 203 | Unit period simple w/ days per period divisor (e.g. 91 for quarterly payment frequencies)   |
| 204 | Unit period simple w/ true 360 calendar and 360 day divisor.  |
| 205 | Unit period simple w/ true 360 calendar and 365 day divisor.  |
| 206 | Unit period simple w/ true 360 calendar and days per period divisor (e.g. 91 for quarterly payment frequencies)   |
| 207 | Unit period simple w/ Federal Calendar   |
| 210 | Actual / 360 simple  |
| 211 | True 365 / 360 simple  |
| 220 | Actual / 365 simple  |
| 221 | True 365 / 365 simple  |
| 230 | Actual / Actual simple  |
| 231 | Midnight 366 simple  |
| 240 | Actual / 365.25 simple  |
| 250 | Unit period simple w/ variable DPY divisor  |
| 301 | Unit period US Rule w/ 360 day divisor  |
| 302 | Unit period US Rule w/ 365 day divisor  |
| 303 | Unit period US Rule w/ days per period divisor (e.g. 91 for quarterly payment frequencies)   |
| 304 | Unit period US Rule w/ true 360 calendar and 360 day divisor.  |
| 305 | Unit period US Rule w/ true 360 calendar and 365 day divisor.  |
| 306 | Unit period US Rule w/ true 360 calendar  and days per period divisor (e.g. 91 for quarterly payment frequencies)   |
| 307 | Unit period US Rule w/ Federal Calendar   |
| 310 | Actual / 360 US Rule  |
| 311 | True 365 / 360 US Rule  |
| 320 | Actual / 365 US Rule  |
| 321 | True 365 / 365 US Rule  |
| 330 | Actual / Actual US Rule  |
| 331 | Midnight 366 USRule  |
| 340 | Actual / 365.25 US Rule  |
| 350 | Unit period US Rule w/ variable DPY divisor  |
| 400 | Add On  |

🟦 **Accrual.LastDay**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

This setting only applies to loans computed with an actual day accrual calendar
where the first payment date falls at the end of a month with fewer than 31
days. As an example, if the first payment date is on April 30, should the
following payment dates be made on the 30th of each month, or on the last day of
the month?

A value of `true` means that when conditions triggering a last day situation are
active, this will result in payments which fall at the end of the month. A value
of `false` indicates that when dictated, subsequent payment dates will not be
moved to the last day of the month.

🟦 **Accrual.InterestRound**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| String | false | Nearest, Up, Down | Nearest |

When amortizing interest in the amortization schedule, the value of this field
dictates how the SCE rounds off interest with fractional currency units.

🟦 **Accrual.PaymentRound**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| String | false | Nearest, Up, Down, Best | Nearest |

When the SCE computes the payment which best amortizes the loan to zero, the value
of this field dictates how the payment is rounded.

A value of `Best` means
to choose the payment that returns a minimal ending balance at the end of
amortization. If two payments result in the same minimal error magnitude, the
smaller payment is chosen.

🟥 **Accrual.SinglePayDivisor**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| String | false | see table | n/a |

When computing a single payment note, the value of this field determines the number
of days in the interest accrual factor's divisor.

| Code | Description |
| :---: | :--- |
| 360 | 360 day divisor |
| 365 | 365 day divisor |
| 366 | 366 day divisor |

</details>

---

</details>

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Account](module-account.md) | [SCEJSON Reference Manual](README.md) | [Version](module-version.md) |
