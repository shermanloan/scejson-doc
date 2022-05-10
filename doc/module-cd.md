# Certificates of Deposit

> "If only God would give me some clear sign!  
>  Like making a large deposit in my  
>  name at a Swiss bank."  
>  --- Woody Allen

A certificate of deposit (hereafter referred to as a CD) is a time deposit held
in a bank or savings and loan for a fixed period of time at a fixed rate.
Interest accrues on this deposit with a fixed periodicity (e.g. monthly or
daily). The amount of money initially deposited in the CD is called the *present
value*, and the final balance of the CD at the end of its term (which equals the
initial deposit plus all accrued interest) is called the *future value*.

There are four basic variables involved in computing a CD: present value,
interest rate, term, and future value. Given any of the three, you may find the
fourth. Instead of creating four different request and response JSON interfaces,
only one request format and one response format are used, each of which is
detailed below.

## Cd Data Object Field Definition

**Example - Request Envelope for Cd Module**

The following is a sample SCE request for a Hpml calculation:

```json
{
  "Module" : "Cd",
  "Data" : {
    "Calculate" : "FV",
    "Date" : "2022-05-01",
    "PV" : "200000.00",
    "Rate" : "6.0",
    "Term" : "12",
    "TermUnits" : "Months",
    "Compounding" : "Monthly"
  }
}
```

The fields of the `Data` object supported by a Cd module request are defined
in alphabetical order below:

### 🟦 BasisDays

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 360, 365 | 365 |

If the compounding frequency (see below) is based upon a daily method (e.g. 
daily, continuous, or simple), then the number of days in a year is specified
with this attribute. If no value is specified, then 365 is assumed.

### 🟦 Calculate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | PV, FV, Rate, Term | FV |

This field informs the SCE which of the four basic variables involved in a CD is
to be calculated (e.g. `PV`, `FV`, `Rate`, or `Term`).

In the sample above, the calculation type being requested is `FV` (future
value). Thus, the `PV`, `Rate`, and `Term` fields are included in the request,
whereas the `FV` field is omitted.

### 🟦 Compounding

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Annual, Semiannual, Quarterly, Bimonthly, Monthly, Semimonthly, Daily, Continuous, Simple | Monthly |

The `Compounding` field defines the periodicity at which interest is
computed. The specified compounding period affects the allowable values
(and units) for the `Term` and `TermUnits` fields - please see the
description below.

The simple compounding frequency does no compounding of interest at all.
Instead, interest is computed at the end of the CD's term (much like a
single payment note using a simple interest accrual method). 

### 🟦 FV

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

The future value is the amount that the CD is worth at the end of it's
term. The future value is equal to the present value plus all interest
accrued during the CD's term. If you are computing the future value
(e.g. `"CalcType" : "FV"`), then you may omit this field.

### 🟦 PV

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

The present value is the amount which is deposited in the CD on the given
deposit date. Interest accrues on this initial amount, much like interest
accrues on the principal balance of a loan. If you are computing the
present value (e.g. `"CalcType" : "PV"`), then you may omit this field.

### 🟦 Rate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

This field dictates the rate at which interest accrues throughout the
term of the CD. If you are computing the interest rate (e.g. 
`"CalcType" : "Rate"`), then you may omit this field.

### 🟦 Term

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

The `Term` field specifies the number of unit term periods after which
the CD has matured. This numerical value can specify the number of
days, months, or years which make up the term of the CD. To specify
the units, please see the `TermUnits` field below.

Also note that the value of this field is somewhat restricted given
certain compounding frequencies and term units. The table below will
summarize the restrictions.

| Compounding | Units Allowed        | Term Restriction |
| :-----      | :------              | :----- |
| Annual      | Years, Months        | Months must be divisible by 12 |
| Semiannual  | Years, Months        | Months must be divisible by 6 |
| Quarterly   | Years, Months        | Months must be divisible by 3 |
| Bimonthly   | Years, Months        | Months must be divisible by 2 |
| Monthly     | Years, Months        | n/a  |
| Semimonthly | Years, Months        | n/a |
| Daily       | Years, Months, Days  | n/a |
| Continuous  | Years, Months, Days  | n/a |
| Simple      | Years, Months, Days  | n/a |

### 🟦 TermUnits

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Years, Months, Days | Months |

This field specifies the units used for the CD's term. The three
options are: Years, Months, and Days.

## Cd Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Cd Module** *(hosted on AWS)*

```json
{
  "Result" : 200,
  "Module" : "Cd",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "PV" : "200000.00",
    "Rate" : "6.000",
    "Term" : "12",
    "TermUnits" : "Months",
    "FV" : "212335.56",
    "Maturity" : "2023-05-01",
    "Gain" : "12335.56",
    "APY" : "6.168"
  }
}
```

The `Data` object for a response from the Cd module is defined below, in the
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
  "Module" : "Cd",
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
  "Module" : "Cd",
  "Data" : {
    "Errors" : [
      "Data.Date (StringDate) not found.",
      "Data.PV (StringFloat) not found.",
      "Data.Rate (StringFloat) not found.",
      "Data.Term (StringFloat) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}```

### 🟥 PV

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The present value of the CD, representing the initial deposit made on the specified deposit
date.

### 🟥 Rate

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - % |

The rate at which interest will accrue during the term of the CD.

### 🟥 Term

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The `Term` is the number of unit periods (as specified in the `TermUnits` field)
between the deposit date and the CD's maturity date.

### 🟥 TermUnits

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Years, Months, Days |

This field returns the units used for the CD's term. The three
options are: Years, Months, and Days.

### 🟥 FV

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The future value of the CD at its maturity.

### 🟥 Maturity

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The date of maturity for the CD. All dates are in the form of YYYY-MM-DD, and
must be 10 characters long.

### 🟥 Gain

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The gain is the difference between the future value and the present value. It is the
amount by which the present value changes over the term of the CD.

### 🟥 APY

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - % |

The APY (annual percentage yield) is the rate of return on an investment over the
term of a year. The APY takes compounding into account, and if the compounding frequency
is other than annual, the APY will differ from the interest rate.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Higher Priced Mortgage Loans (HPML)](module-hpml.md) | [SCEJSON Reference Manual](README.md) | [Calculation Notes](appendix-calcnotes.md) |
