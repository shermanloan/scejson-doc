# Retirement Annuities

> "I have a fast swing and I can outdrive a lot of  
>  the men I play with. Of course, I only play  
>  with people from retirement communities."  
>  --- Tea Leoni

An annuity is a stream of income paid in a series of regular monthly payments.
Annuities are usually set up to disburse these payments from a retirement
account, and act as an income stream.

When planning for retirment, it is natural to ask what amount of monthly income
will I receive given a planned retirment account balance of X dollars, with a
planned retirement term of 360 months? Similarly, given a planned retirment
account balance of X dollars and a desired monthly income of Y dollars, how many
months may I draw income? Finally, given a desired term of income and monthly
income amount, what would my retirement balance need to be?

Thus, there are three basic variables involved in computing an annuity: the
beginning balance, monthly income amount, and the number of months of income.
Given any of two, one may find the third. Instead of creating three different
request and response JSON interfaces, only one request format and one response
format are used, each of which is detailed below.

## Annuity Request Data Object Field Definition

**Example - Request Envelope for Annuity Module**

The following is a sample SCE request for an Annuity calculation:

```json
{
  "Module" : "Annuity",
  "Data" : {
    "Calculate" : "Income",
    "Balance" : "500000.00",
    "Rate" : "5.000",
    "Term" : "360"
  }
}
```

The fields of the `Data` object supported by the Annuity module request are defined
in alphabetical order below:

### 🟦 Balance

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | n/a |

The beginning balance is the amount which is present in the retirment
account when the annuity starts. Interest will accrue monthly on this 
amount, and the income stream is taken from this balance during the
term of the annuity. If you are computing the beginning balance
(e.g. `"Calculate" : "Balance"`), then this field may be omitted.

### 🟦 Calculate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Balance, Income, Term | Income |

This field informs the SCE which of the three basic variables involved in an annuity calculation is
to be calculated (e.g. `Balance`, `Income`, or `Term`).

In the sample above, the calculation type being requested is `Income` (the monthly
income amount). Thus, the `Balance`, and `Term` fields are included in the request,
whereas the `Income` field is omitted.

### 🟦 Format

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Format` object is one of the first objects parsed from a request, as various
fields affect how date and numeric fields are parsed and validated.

<details><summary><b>Format fields</b></summary>
---

🟦 **Format.CurrencyDecimals**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0 or 2 | 2 |

When displaying and parsing Currency fields, this field determines the maximum
number of decimal places allowed after the `DecimalSeparator`.

🟦 **Format.DateFormat**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YMD, MDY, or DMY  | YMD |

When displaying and parsing Date fields, this field determines the expected
format for all Date fields. The following `DateFormat` options are allowed:

* `YMD` - All dates should be formated as YYYY-MM-DD.
* `MDY` - All dates should be formated as MM-DD-YYYY.
* `DMY` - All dates should be formated as DD-MM-YYYY.

Note that the character which separates the individual month, day, and year
portions of the date is configurable via the `DateSeparator` field.

🟦 **Format.DateSeparator**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | empty or a single character  | "-" |

When displaying and parsing Date fields, this field determines the character
used to separate the individual month, day, and year portions of a date field.

🟦 **Format.DecimalSeparator**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | empty or a single character  | "." |

When displaying and parsing Currency, Percentage, or Floating numeric fields,
this field determines the character used to separate the fractional part from
the whole.

🟦 **Format.StrictDP**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false  | true |

If the value of this field is `true`, then the SCE will strictly verify the
number of decimal places allowed for currency input values. Thus, if the calling
application sends in a request with a currency amount of 1000.005, the SCE will
return an error code.

If the value of this attribute is set to `false`, then currency values sent in
with an invalid number of decimal places will be rounded to the correct number
of decimal places by the SCE (using five/four rounding), and a warning message
with this information will be returned with the response.

🟦 **Format.ThousandSeparator**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | empty or a single character  | "" |

When displaying numeric fields, this field determines the character used to
separate the thousands places from the hundreds. Note that when parsing
numeric fields, the value of this field is ignored.

---

</details>

### 🟦 Income

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | n/a |

The value of this field specifies the desired monthly income amount. If you are
computing the monthly income amount (e.g. `"Calculate" : "Income"`), then you
may omit this field from the request.

### 🟥 Rate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

This field dictates the rate at which interest accrues throughout the
term of the annuity.
 
### 🟦 Term

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | n/a |

This field specifies the number of months in the term of the annuity.
This corresponds to the length of the monthly income stream which is
disbursed from the beginning balance. If you are computing the number
of months of income (e.g. `"Calculate" : "Term"`), then this field may
be omitted.

## Annuity Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Annuity Module**

```json
{
  "Result" : 200,
  "Module" : "Annuity",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Balance" : "500000.00",
    "Rate" : "5.000",
    "Income" : "5018.38",
    "Term" : "129",
    "TotalIncome" : "647371.02",
    "Gain" : "147371.02"
  }
}
```

The `Data` object for a response from the Annuity module is defined below, in the
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
  "Module" : "Annuity",
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
  "Module" : "Annuity",
  "Data" : {
    "Errors" : [
      "Data.Balance (StringFloat) not found.",
      "Data.Rate (StringFloat) not found.",
      "Data.Term (StringInt) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}
```

### 🟥 Balance

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The beginning balance is the amount which is present in the retirment
account when the annuity starts.

### 🟥 Rate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The rate at which interest will accrue on the balance during the term of the
annuity.

### 🟥 Income

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The amount of the monthly disbursement.

### 🟥 Term

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This number of months of income provided by the annuity.

### 🟥 TotalIncome

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The `TotalIncome` is the sum of all monthly disbursements made during
the term of the annuity.

### 🟥 Gain

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The `Gain` is the difference between the total income and the beginning balance.
It is the total amount of interest accrued over the term of the annuity.


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Individual Retirement Accounts](module-ira.md) | [SCEJSON Reference Manual](README.md) | [Calculation Notes](appendix-calcnotes.md) |
