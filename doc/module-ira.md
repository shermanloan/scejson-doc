# Individual Retirement Accounts

> "When a man retires and time is no longer a  
>  matter of urgent importance, his colleagues  
>  generally present him with a watch."  
>  --- R.C. Sherriff

An individual retirement account (hereafter referred to as an IRA) is a deposit
held in an account for a period of time, during which further deposits may be
made and interest is accrued. Interest accrues on the initial balance and
periodic deposits with a fixed periodicity (e.g. monthly, daily). The amount of
money initially deposited in the IRA is called the *beginning balance*, and the
final balance of the IRA at the end of its term (which depends upon the current
and retirement ages of the account holder) is called the *future value*.

The SCE allows the computation of an IRA two different ways: (i) specify the
initial balance, periodic deposit amount, current and retirement ages, and
interest rate to compute the future value; or (ii) compute the necessary
periodic deposit amount to achieve a target future value.

## Ira Request Data Object Field Definition

**Example - Request Envelope for Ira Module**

The following is a sample SCE request for an Ira calculation:

```json
{
  "Module" : "Ira",
  "Data" : {
    "Calculate" : "FV",
    "Compounding" : "Annual",
    "BeginBalance" : "2000.00",
    "Deposit" : "2000.00",
    "DepositFreq" : "Annual",
    "CurrentAge" : "30",
    "RetireAge" : "65",
    "Rate" : "5.000"
  }
}
```

The fields of the `Data` object supported by a Ira module request are defined
in alphabetical order below:

### 🟥 BeginBalance

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

The beginning balance is the amount which is currently present in the IRA account.
Interest accrues on this initial amount (plus the periodic deposit amounts), much
like interest accrues on the principal balance of a loan.

### 🟦 Calculate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | FV, Deposit | FV |

This field informs the SCE which of the two IRA calculations is to be performed.

In the sample above, the calculation type being requested is `FV`, which directs
the SCE to compute the *future value* of the IRA. Thus, the periodic deposit
value is specified in the appropriate field, and the future value field is
omitted.

### 🟦 Compounding

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Annual, Semiannual, Quarterly, Bimonthly, Monthly, Daily | Annual |

The `Compounding` field defines the periodicity at which interest is computed.
The specified compounding period affects the allowable values for the
`DepositFreq` field - please see the description below.

### 🟥 CurrentAge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The current age of the IRA account owner.

### 🟦 Deposit

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | n/a |

The `Deposit` field specifies the periodic deposit amount to be made to the IRA
account. The periodicity of the deposit is dictated by the `DepositFrequency`
field, detailed below.

If you are computing the periodic deposit amount necessary to generate a
specified future value (e.g. `"Calculate" : "FV"'), then omit this field from
the request.

### 🟦 DepositFreq

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Annual, Semiannual, Quarterly, Bimonthly, Monthly | Annual |

This field specifies the periodicity of the deposit amount. Thus, if you
wanted to make monthly deposits to an IRA account, the attribute value should
be specified as `monthly`. The frequency of deposits limits the 
allowable compounding frequencies (see table below).

| Deposit Frequency | Compounding Frequency Allowed |
| :----             | :---- |
| Annual            | Annual, Semiannual, Quarterly, Bimonthly, Monthly, Daily |
| Semiannual        | Semiannual, quarterly, Bimonthly, Monthly, Daily |
| Quarterly         | Quarterly, Monthly, Daily |
| Bimonthly         | Bimonthly, Monthly, Daily  |
| Monthly           | Monthly, Daily |

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

🟦 **Format.ThousandSeparator**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | empty or a single character  | "" |

When displaying numeric fields, this field determines the character used to
separate the thousands places from the hundreds. Note that when parsing
numeric fields, the value of this field is ignored.

---

</details>

### 🟥 Rate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - % | n/a |

This field dictates the rate at which interest accrues throughout the
term of the IRA.

### 🟥 RetireAge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The desired retirement age of the IRA account owner. This value must be greater
than that found in the `CurrentAge` field. By subtracting the current age from
the retirement age, the SCE determines the number of years the IRA account will
be allowed to grow.

### 🟦 FV

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | n/a |

The future value is the balance of the IRA account at the borrower's specified
retirement age. The future value is equal to the present value plus all periodic
deposits plus all interest accrued during the IRA's term. If you are computing
the future value (e.g. `"Calculate" : "FV"`), then you may omit this field.

## Ira Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Ira Module**

```json
{
  "Result" : 200,
  "Module" : "Ira",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "BeginBalance" : "2000.00",
    "Deposit" : "2000.00",
    "DepositFreq" : "Annual",
    "Rate" : "5.000",
    "CurrentAge" : "30",
    "RetireAge" : "65",
    "Term" : "35",
    "TermUnits" : "Years",
    "FV" : "200704.68",
    "Gain" : "128704.68",
    "Yield" : "5.000"
  }
}
```

The `Data` object for a response from the Ira module is defined below, in the
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
  "Module" : "Ira",
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
  "Module" : "Ira",
  "Data" : {
    "Errors" : [
      "Data.Deposit (StringFloat) not found.",
      "Data.Rate (StringFloat) not found.",
      "Data.CurrentAge (StringInt) not found.",
      "Data.RetireAge (StringInt) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}
```

### 🟥 BeginBalance

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The beginning balance is the amount which is currently present in the IRA account.
Interest accrues on this initial amount (plus the periodic deposit amounts), much
like interest accrues on the principal balance of a loan.

### 🟥 Deposit

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The `Deposit` field specifies the periodic deposit amount to be made to the IRA
account.

### 🟥 DepositFreq

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Annual, Semiannual, Quarterly, Bimonthly, Monthly | Annual |

This field specifies the periodicity of the deposit amount.

### 🟥 Rate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The rate at which interest will accrue during the term of the IRA.

### 🟥 CurrentAge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The current age of the IRA account owner.

### 🟥 RetireAge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The desired retirement age of the IRA account owner.

### 🟥 Term

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term is the number of years during which periodic deposit amounts and accrued
interest are applied to the IRA account. It is equal to the individual's retirement
age less current age.

### 🟥 TermUnits

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Years |

The `Term` field is always expressed as a number of years.

### 🟥 FV

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The future value of the IRA at retirement age.

### 🟥 Gain

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The gain is the difference between the future value and the sum of the initial balance
and all periodic deposits made. It is the amount of interest accrued over the term of
the IRA.

### 🟥 Yield

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The APY (annual percentage yield) is the rate of return on an investment over the
term of a year. The APY takes compounding into account, and if the compounding frequency
is other than annual, the APY will differ from the interest rate.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Certificates of Deposit](module-cd.md) | [SCEJSON Reference Manual](README.md) | [Retirement Annuities](module-annuity.md) |
