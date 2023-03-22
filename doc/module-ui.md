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

If the loan setup fil was successfully loaded and parsed, the value of this
field will be `Loaded`. Any other return value indicates an error has occurred
while trying to loan the loan setup file for the account number requested.

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

### 🟥 Cl

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The `Cl` object returns information on life protection configuration setup file
data.

<details>
<summary><b>Cl fields</b></summary>

---

🟥 **Cl.Active**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If life coverage is configured and allowed on one or more loan types, then the
value of this field will be `true` and the rest of the object should be parsed
for further details.

A value of `false` indicates that life coverage is not confiured nor allowed for
this account. Furthermore, no other fields of the `Cl` object will be returned
to the calling application.

🟥 **Cl.AllowOn**

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `AllowOn` object contains fields which inform the calling application the
loan types which allow for the inclusion of this life product.

<details>
<summary><b>AllowOn fields</b></summary>

---

🟦 **AllowOn.ARM**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Adjustable Rate
Mortgages](module-arm.md).

🟦 **AllowOn.Balloon**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Balloon Payment
Loans](module-balloon.md).

🟦 **AllowOn.Equal**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Equal Payment
Loans](module-equal.md).

🟦 **AllowOn.IntOnly**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Interest Only
Loans](module-interestonly.md).

🟦 **AllowOn.Irregs**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Skip, Pickup and
Irregular Payment Loans](module-irregular.md).

🟦 **AllowOn.PrinPlus**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Fixed Principal
Plus Interest Loans](module-principalplus.md).

🟦 **AllowOn.SinglePay**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that life coverage is allowed on [Single Payment
Notes](module-singlepmt.md).

---

</details>

🟦 **Cl.IsDP**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the life product has been configured as a debt protection product, then the
value of this attribute will be `true`. A value of `false` indicates that the
life product is considered insurance.

🟥 **Cl.Method**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | SinglePremium, StandardMob, TrueMOB, CUNASP |

The `Method` field returns the general method used to compute the life products
premiums/fees. `SinglePremium` indicates that a single premium or fee is
computed and (usually) financed. The `StandardMob` method computes a premium/fee
with each payment. The `TrueMOB` method computes a premium/fee on a specified
day number every month. Finally, `CUNASP` is a single premium method implemented
specifically for CUNA Mutual.

The value of this field also corresponds to the name of the `Cl` object field
which will return further information useful for this general method.

As an example, if `Cl.Method` is `SinglePremium`, then there will be a
`Cl.SinglePremium` object which should be parsed (detailed below) for additional
information realted to this method.

🟦 **Cl.CUNASP**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `CUNASP` object will only be returned when `Cl.Method` is `CUNASP`. It
contains additional information which should be parsed related to this general
method.

<details>
<summary><b>CUNASP fields</b></summary>

---

🟥 **CUNASP.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured life calculation option. The
value of this element defaults to `CL`, although customers who wish to customize
this abbreviation may do so. Customers who offer debt cancellation instead of
insurance will also usually wish to change the abbreviations of their life
product for regulatory reasons (e.g. `DCL` for debt cancellation life).

The title may be used for inputs (e.g. `CL Option`) and outputs (e.g. `CL
Coverage Term`).

🟥 **CUNASP.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured life product. This title defaults to `Credit Life`, although
customers who wish to customize the name of this offering may do so. For
example, it could simply be called `Life`, or `Death`. Customers who offer debt
cancellation instead of insurance will also usually wish to change the name of
their life product for regulatory reasons.

The title may be used for inputs (e.g. `Credit Life Option`) and outputs (e.g.
`Credit Life Premium/Fee`).

---

</details>

🟦 **Cl.SinglePremium**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `SinglePremium` object will only be returned when `Cl.Method` is
`SinglePremium`. It contains additional information which should be parsed
related to this general method.

<details>
<summary><b>SinglePremium fields</b></summary>

---

🟥 **SinglePremium.DlAbbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured decreasing life calculation
option. The value of this element defaults to `CL`, although customers who wish
to customize this abbreviation may do so. Customers who offer debt cancellation
instead of insurance will also usually wish to change the abbreviations of their
life product for regulatory reasons (e.g. `DCL` for debt cancellation life).

The title may be used for inputs (e.g. `CL Option`) and outputs (e.g. `CL
Coverage Term`).

🟥 **SinglePremium.DlTitle**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured decreasing life product. This title defaults to `Credit Life`,
although customers who wish to customize the name of this offering may do so.
For example, it could simply be called `Life`, or `Death`. Customers who offer
debt cancellation instead of insurance will also usually wish to change the name
of their life product for regulatory reasons.

The title may be used for inputs (e.g. `Credit Life Option`) and outputs (e.g.
`Credit Life Premium/Fee`).

🟥 **SinglePremium.LlAbbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Similar to `SinglePremium.DlAbbrev`, except this abbreviation is for the
configured level life product.

🟥 **SinglePremium.LlTitle**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Similar to `SinglePremium.DlTitle`, except this abbreviation is for the
configured level life product.

🟦 **SinglePremium.Gross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If gross coverage is allowed for the configured life product, then the value of
this field will be `true`. A value of `false` indicates that gross coverage is
not supported.

🟦 **SinglePremium.Level**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If level coverage is allowed for the configured life product, then the value of
this field will be `true`. A value of `false` indicates that level coverage is
not supported.

🟦 **SinglePremium.Net**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If net coverage is allowed for the configured life product, then the value of
this field will be `true`. A value of `false` indicates that net coverage is
not supported.

🟦 **SinglePremium.OrdinaryLife**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If an ordinary life product has been configured for the life product, then the
value of this field will be `true`. A value of `false` indicates that no odinary
life product is supported.

🟥 **SinglePremium.Prompts**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Prompts` object returns information to the calling application about optional
prompts/inputs that may be allowed for this life product.

<details>
<summary><b>Prompts fields</b></summary>

---

🟦 **Prompts.CoverageAmount**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the account has been configured to allow for a coverage amount to be
specified by the user, then the value of this field will be `true`, which
indicates that the user interface should prompt the user for a life coverage
amount. A value of `false` indicates that only the default coverage amount will
be considered, and hence there is no need to prompt for a user specified
coverage.

🟦 **Prompts.CoverageTermGross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for single premium gross life. A value of `false` indicates that user
specified coverage term truncation is not allowed and should not be prompted.

🟦 **Prompts.CoverageTermNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for single premium net life. A value of `false` indicates that user
specified coverage term truncation is not allowed and should not be prompted.

🟦 **Prompts.Dismemberment**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Some accounts allow for an additional dismemberment option to be written with
the life policy (with an associated rate increase). If dismemberment is an
option for this account, then the value of this field will be `true`. Otherwise,
a value of `false` indicates that no dismemberment option is offered.

🟦 **Prompts.GrossNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Some accounts using single premium life may support both net and gross credit
life. If this is the case, this element will hold a value of `true` to signal
the user interface that the user should be prompted for the life method. A value
of `false` indicates that prompting for the life method is not required, as only
one method is allowed.

🟦 **Prompts.SingleOnCoborrower**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If single coverage is allowed on the co-borrower, then this field will hold a
value of `true`. If it holds a value of `false`, then single life is only
allowed on the primary borrower.

🟦 **Prompts.UseLevelRatesGross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

In some states, it is legal to compute gross decreasing life using level rates.
If that is an option for this account, then this value of this field will be
`true`, which indicates that the user interface should prompt for the option to
use level rates with gross life. A value of `false` indicates that this option
is not allowed, and hence the prompt for using level rates should not be
presented to the user.

🟦 **Prompts.UseLevelRatesNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

In some states, it is legal to compute net decreasing life using level rates.
If that is an option for this account, then this value of this field will be
`true`, which indicates that the user interface should prompt for the option to
use level rates with net life. A value of `false` indicates that this option
is not allowed, and hence the prompt for using level rates should not be
presented to the user.

</details>

---

</details>

🟦 **Cl.StandardMob**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `StandardMob` object will only be returned when `Cl.Method` is
`StandardMob`. It contains additional information which should be parsed related
to this general method.

<details>
<summary><b>StandardMob fields</b></summary>

---

🟥 **StandardMob.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured life calculation option. The
value of this element defaults to `CL`, although customers who wish to customize
this abbreviation may do so. Customers who offer debt cancellation instead of
insurance will also usually wish to change the abbreviations of their life
product for regulatory reasons (e.g. `DCL` for debt cancellation life).

The title may be used for inputs (e.g. `CL Option`) and outputs (e.g. `CL
Coverage Term`).

🟥 **StandardMob.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured life product. This title defaults to `Credit Life`, although
customers who wish to customize the name of this offering may do so. For
example, it could simply be called `Life`, or `Death`. Customers who offer debt
cancellation instead of insurance will also usually wish to change the name of
their life product for regulatory reasons.

The title may be used for inputs (e.g. `Credit Life Option`) and outputs (e.g.
`Credit Life Premium/Fee`).

🟥 **StandardMob.Prompts**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Prompts` object returns information to the calling application about optional
prompts/inputs that may be allowed for this life product.

<details>
<summary><b>Prompts fields</b></summary>

---

🟦 **Prompts.CoverageAmount**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the account has been configured to allow for a coverage amount to be
specified by the user, then the value of this field will be `true`, which
indicates that the user interface should prompt the user for a life coverage
amount. A value of `false` indicates that only the default coverage amount will
be considered, and hence there is no need to prompt for a user specified
coverage.

🟦 **Prompts.CoverageTerm**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for life. A value of `false` indicates that user specified coverage
term truncation is not allowed and should not be prompted.

🟦 **Prompts.Dismemberment**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

Some accounts allow for an additional dismemberment option to be written with
the life policy (with an associated rate increase). If dismemberment is an
option for this account, then the value of this field will be `true`. Otherwise,
a value of `false` indicates that no dismemberment option is offered.

🟦 **Prompts.SingleOnCoborrower**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If single coverage is allowed on the co-borrower, then this field will hold a
value of `true`. If it holds a value of `false`, then single life is only
allowed on the primary borrower.

</details>

---

</details>

🟦 **Cl.TrueMob**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `TrueMob` object will only be returned when `Cl.Method` is `TrueMob`. It
contains additional information which should be parsed related to this general
method.

<details>
<summary><b>TrueMob fields</b></summary>

---

🟥 **TrueMob.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured life calculation option. The
value of this element defaults to `CL`, although customers who wish to customize
this abbreviation may do so. Customers who offer debt cancellation instead of
insurance will also usually wish to change the abbreviations of their life
product for regulatory reasons (e.g. `DCL` for debt cancellation life).

The title may be used for inputs (e.g. `CL Option`) and outputs (e.g. `CL
Coverage Term`).

🟥 **TrueMob.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured life product. This title defaults to `Credit Life`, although
customers who wish to customize the name of this offering may do so. For
example, it could simply be called `Life`, or `Death`. Customers who offer debt
cancellation instead of insurance will also usually wish to change the name of
their life product for regulatory reasons.

The title may be used for inputs (e.g. `Credit Life Option`) and outputs (e.g.
`Credit Life Premium/Fee`).

🟥 **TrueMob.Prompts**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Prompts` object returns information to the calling application about optional
prompts/inputs that may be allowed for this life product.

<details>
<summary><b>Prompts fields</b></summary>

---

🟦 **Prompts.SingleOnCoborrower**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If single coverage is allowed on the co-borrower, then this field will hold a
value of `true`. If it holds a value of `false`, then single life is only
allowed on the primary borrower.

</details>

</details>

---

</details>

### 🟥 Ah

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The `Ah` object returns information on accident and health protection (A&H)
configuration setup file data. Note that packaged debt protection products use
the `Ah` product for implementation (see `Ah.IsPackagedDP`).

<details>
<summary><b>Ah fields</b></summary>

---

🟥 **Ah.Active**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If A&H coverage is configured and allowed on one or more loan types, then the
value of this field will be `true` and the rest of the object should be parsed
for further details.

A value of `false` indicates that A&H coverage is not confiured nor allowed for
this account. Furthermore, no other fields of the `Ah` object will be returned
to the calling application.

🟥 **Ah.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured A&H calculation option. The
value of this element defaults to `AH`, although customers who wish to customize
this abbreviation may do so. Customers who offer debt cancellation instead of
insurance will also usually wish to change the abbreviations of their A&H
product for regulatory reasons.

The title may be used for inputs (e.g. `AH Option`) and outputs (e.g. `AH
Coverage Term`).

🟥 **Ah.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured disability product. This title defaults to `Accident & Health`, although
customers who wish to customize the name of this offering may do so. Customers who offer debt
cancellation instead of insurance will also usually wish to change the name of
their disability product for regulatory reasons.

The title may be used for inputs (e.g. `Accident & Health Option`) and outputs (e.g.
`Accident & Health Premium/Fee`).

🟥 **Ah.AllowOn**

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `AllowOn` object contains fields which inform the calling application the
loan types which allow for the inclusion of this product.

<details>
<summary><b>AllowOn fields</b></summary>

---

🟦 **AllowOn.ARM**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Adjustable Rate Mortgages](module-arm.md).

🟦 **AllowOn.BalloonGross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Balloon Payment Loans](module-balloon.md) covered with single premium gross
life.

🟦 **AllowOn.BalloonMobNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Balloon Payment Loans](module-balloon.md) covered with single premium net or
monthly outstanding balance life.

🟦 **AllowOn.Equal**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Equal
Payment Loans](module-equal.md).

🟦 **AllowOn.IntOnly**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Interest Only Loans](module-interestonly.md).

🟦 **AllowOn.Irregs**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Skip,
Pickup and Irregular Payment Loans](module-irregular.md).

🟦 **AllowOn.PrinPlus**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Fixed
Principal Plus Interest Loans](module-principalplus.md).

🟦 **AllowOn.SinglePay**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Single
Payment Notes](module-singlepmt.md).

---

</details>

🟦 **Ah.IsDP**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the A&H product has been configured as a debt protection product, then the
value of this attribute will be `true`. A value of `false` indicates that the
A&H product is considered insurance.

🟦 **Ah.IsPackagedDP**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the A&H product has been configured as a packaged debt protection product, then the
value of this attribute will be `true`. A value of `false` indicates that the
A&H product is not configured as packaged debt protection.

When A&H is configured as packaged debt protection, the debt protection plans
offered are all enumerated under `Ah.Tables[]`. As an example, if there are four
packaged debt protection plans configured, then the `Ah.Tables[]` array will
return the name of each plan, in order. Thus, if `Ah.Tables` is equal to
`["Life", "Life and Disability", "Life, Disability, and Unemployment"]` then
`Table` #1 corresponds to `Life`, etc.

🟥 **Ah.Method**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | SinglePremium, StandardMob, TrueMOB, CUNASP |

The `Method` field returns the general method used to compute the A&H product
premiums/fees. `SinglePremium` indicates that a single premium or fee is
computed and (usually) financed. The `StandardMob` method computes a premium/fee
with each payment. The `TrueMOB` method computes a premium/fee on a specified
day number every month. Finally, `CUNASP` is a single premium method implemented
specifically for CUNA Mutual.

🟥 **Ah.Prompts**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Prompts` object returns information to the calling application about optional
prompts/inputs that may be allowed for this A&H product.

<details>
<summary><b>Prompts fields</b></summary>

---

🟦 **Prompts.CoverageBenefit**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the account has been configured to allow for a benefit amount to be specified
by the user, then the value of this field will be `true`, which indicates that
the user interface should prompt the user for an A&H benefit amount. A value of
`false` indicates that only the default benefit amount will be considered, and
hence there is no need to prompt for a user specified benefit.

🟦 **Prompts.CoverageTermGross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for A&H when the loan is covered with single premium gross life. A
value of `false` indicates that user specified coverage term truncation is not
allowed with this life coverage type and should not be prompted.

🟦 **Prompts.CoverageTermMobNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for life when the loan is covered with single premium net or monthly
outstanding balance life. A value of `false` indicates that user specified
coverage term truncation is not allowed with this life coverage type and should
not be prompted.

🟦 **Prompts.Joint**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If joint coverage A&H is allowed, then the value of this field will be `true`.
Otherwise, a value of `false` indicates that no joint coverage option is
offered.

🟦 **Prompts.SingleOnCoborrower**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If single coverage is allowed on the co-borrower, then this field will hold a
value of `true`. If it holds a value of `false`, then single coverage is only
allowed on the primary borrower.

---

</details>

🟥 **Ah.Rules**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Ruless` object returns information to the calling application about coverage
rules that may be in effect for this A&H product.

<details>
<summary><b>Rules fields</b></summary>

---

🟦 **Rules.MustHaveLife**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If A&H may only be offered on loans that include life coverage, then
the value of this field will be `true`. A value of `false` means that
A&H may be offered independently of any life product.

🟦 **Rules.PPYLessThan12**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If A&H is allowed on loans with annual, semiannual, quarterly, or bimonthly
payment frequencies, then the value of this field will be `true`.

🟦 **Rules.PPYGreaterThan12**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If A&H is allowed on loans with semimonthly, biweekly, or weekly payment
frequencies, then the value of this field will be `true`.

---

</details>

🟥 **Ah.Tables**

| Type  | Required |
| :---: |   :---:  |
| array of String objects | no |

The `Tables[]` array holds the name of each allowed set of A&H rates. If A&H has
been configured for packaged debt protection (see `Ah.IsPackagedDP`), then this
array holds the name of each allowed packaged debt protection plan.

</details>

### 🟥 Iu

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The `Iu` object returns information on involuntary unemployment protection (IU)
configuration setup file data.

<details>
<summary><b>Iu fields</b></summary>

---

🟥 **Iu.Active**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If IU coverage is configured and allowed on one or more loan types, then the
value of this field will be `true` and the rest of the object should be parsed
for further details.

A value of `false` indicates that IU coverage is not confiured nor allowed for
this account. Furthermore, no other fields of the `Ah` object will be returned
to the calling application.

🟥 **Iu.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

Like the `Title` field below, this field provides an abbreviation to be used for
the product associated with the SCE's configured IU calculation option. The
value of this element defaults to `IU`, although customers who wish to customize
this abbreviation may do so. Customers who offer debt cancellation instead of
insurance will also usually wish to change the abbreviations of their IU product
for regulatory reasons.

The title may be used for inputs (e.g. `IU Option`) and outputs (e.g. `IU
Coverage Term`).

🟥 **Iu.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | text |

The value of this field holds the product title associated with the SCE's
configured unemployment product. This title defaults to `Involuntary
Unemployment`, although customers who wish to customize the name of this
offering may do so. Customers who offer debt cancellation instead of insurance
will also usually wish to change the name of their unemployment product for
regulatory reasons.

The title may be used for inputs (e.g. `Involuntary Unemployment Option`) and
outputs (e.g. `Involuntary Unemployment Premium/Fee`).

🟥 **Iu.AllowOn**

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `AllowOn` object contains fields which inform the calling application the
loan types which allow for the inclusion of this product.

<details>
<summary><b>AllowOn fields</b></summary>

---

🟦 **AllowOn.ARM**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Adjustable Rate Mortgages](module-arm.md).

🟦 **AllowOn.BalloonGross**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Balloon Payment Loans](module-balloon.md) covered with single premium gross
life.

🟦 **AllowOn.BalloonMobNet**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Balloon Payment Loans](module-balloon.md) covered with single premium net or
monthly outstanding balance life.

🟦 **AllowOn.Equal**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Equal
Payment Loans](module-equal.md).

🟦 **AllowOn.IntOnly**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on
[Interest Only Loans](module-interestonly.md).

🟦 **AllowOn.Irregs**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Skip,
Pickup and Irregular Payment Loans](module-irregular.md).

🟦 **AllowOn.PrinPlus**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Fixed
Principal Plus Interest Loans](module-principalplus.md).

🟦 **AllowOn.SinglePay**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` indicates that coverage for this product is allowed on [Single
Payment Notes](module-singlepmt.md).

---

</details>

🟦 **Iu.IsDP**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the IU product has been configured as a debt protection product, then the
value of this attribute will be `true`. A value of `false` indicates that the
IU product is considered insurance.

🟥 **Iu.Method**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | SinglePremium, StandardMob, TrueMOB, CUNASP |

The `Method` field returns the general method used to compute the IU product
premiums/fees. `SinglePremium` indicates that a single premium or fee is
computed and (usually) financed. The `StandardMob` method computes a premium/fee
with each payment. The `TrueMOB` method computes a premium/fee on a specified
day number every month. Finally, `CUNASP` is a single premium method implemented
specifically for CUNA Mutual.

🟥 **Iu.Prompts**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Prompts` object returns information to the calling application about optional
prompts/inputs that may be allowed for this IU product.

<details>
<summary><b>Prompts fields</b></summary>

---

🟦 **Prompts.CoverageBenefit**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If the account has been configured to allow for a benefit amount to be specified
by the user, then the value of this field will be `true`, which indicates that
the user interface should prompt the user for an IU benefit amount. A value of
`false` indicates that only the default benefit amount will be considered, and
hence there is no need to prompt for a user specified benefit.

🟦 **Prompts.CoverageTerm**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

A value of `true` implies that the user may be prompted for a desired term of
coverage for IU. A value of `false` indicates that user specified coverage term
truncation is not allowed and should not be prompted.

🟦 **Prompts.Joint**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If joint coverage IU is allowed, then the value of this field will be `true`.
Otherwise, a value of `false` indicates that no joint coverage option is
offered.

🟦 **Prompts.SingleOnCoborrower**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If single coverage is allowed on the co-borrower, then this field will hold a
value of `true`. If it holds a value of `false`, then single coverage is only
allowed on the primary borrower.

---

</details>

🟥 **Iu.Rules**

| Type  | Required |
| :---: |  :---: |
| Object | true |

The `Ruless` object returns information to the calling application about coverage
rules that may be in effect for this IU product.

<details>
<summary><b>Rules fields</b></summary>

---

🟦 **Rules.JointMustHaveJointDis**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If joint IU may only be offered on loans that include joint A&H coverage, then
the value of this field will be `true`. A value of `false` means that joint IU
may be offered with single A&H.

🟦 **Rules.JointMustHaveJointLife**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If joint IU may only be offered on loans that include joint life coverage, then
the value of this field will be `true`. A value of `false` means that joint IU
may be offered with single life.

🟦 **Rules.MustHaveDis**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If IU may only be offered on loans that include A&H coverage, then
the value of this field will be `true`. A value of `false` means that
IU may be offered independently of any disability product.

🟦 **Rules.MustHaveLife**

| Type  | Required | Values | Default |
| :---: |  :---: | ---   | :---: |
| Boolean | false | true, false | false |

If IU may only be offered on loans that include life coverage, then
the value of this field will be `true`. A value of `false` means that
IU may be offered independently of any life product.

</details>

---

</details>

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Account](module-account.md) | [SCEJSON Reference Manual](README.md) | [Version](module-version.md) |
