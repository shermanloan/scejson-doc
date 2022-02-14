# Loan Module

> "Debt is part of the human condition. Civilization  
>  is based on exchanges - on gifts, trades, loans -  
>  and the revenges and insults that come when they  
>  are not paid back"  
>                           --- Margaret Atwood

The SCEJSON's loan module may be used to compute almost any loan imaginable -
from the simplest equal payment loan to loans that have exotic
and highly customized features (e.g. payments that occur outside of a
fixed periodicity). If you have used the SCEX in the past, then you will know
this module by a slightly different name - "Loan Builder".

## Loan Request Data Object Field Definition

**Example - Request Envelope for Loan Module**

The following example is a request for an equal payment loan with an advance
of $1,000, interest will accrue at a 10% rate using an actual day / 365 U.S.
Rule accrual method (code `320`), and with a repayment schedule of 12 monthly
payments. The final payment will be adjusted for perfect amortization.

```json
{
  "Module" : "Loan",
  "Data" : {
    "BusinessRules" : {
      "AmError" : "AdjPmt"
    },
    "EditOutput" : {
      "ShowType" : true
    },
    "Advances" : [
      {
        "Date" : "2019-01-01",
        "Amount" : "1000.00"
      }
    ],
    "AccrualConfigs" : [
      {
        "Date" : "2019-01-01",
        "IntRate" : "10.000",
        "AccrualCode" : "320"
      }
    ],
    "Fees" : [
      {
        "Name" : "DocFee",
        "Amount" : "50",
        "AddToPrin" : false,
        "AddToFinChg" : true
      }
    ],
    "PmtStreams" : [
      {
        "Date" : "2019-02-01",
        "Term" : "12",
        "PPY" : "12"
      }
    ]
  }
}
```

The `Data` object in the loan module request is defined below:

🟦 **BusinessRules** - *Object { optional }*

Many loan calculation *business rules* may be toggled using the fields of this
object.

<details><summary><b>BusinessRules fields</b></summary>

---  
🟦 **BusinessRules.AmError** - *String (Allow, AdjPmt, AdjPrin, AdjInt) : "Allow" { optional }*

When amortizing a loan, often times there is a non-zero ending balance due to
rounding. This attribute determines what to do (if anything) with a non-zero
ending balance. This non-zero balance is referred to as the amortization error.

* **Allow** - Do not do anything with the non-zero ending balance.

* **AdjPmt** - Adjust the final payment to achieve perfect amortization,
which will then result in an ending balance of zero.

* **AdjPrin** - Adjust the final principal reduction amount to achieve
perfect amortization, which will then result in an ending balance of zero.

* **AdjInt** - Adjust the final interest reduction amount to achieve
perfect amortization, which will then result in an ending balance of zero.

Note that using `AdjPrin` or `AdjInt` will cause the final payment to not equal
the sum of the principal reduction amount and amount to interest in the
amortization schedule.

🟦 **BusinessRules.AmortizeOnly** - *Boolean : false { optional }*

If the calling application sets the value of this field to `true`, then instead
of computing a loan with payments that best amortize the principal balance to
zero, the Loan module is instructed to simply amortize the specified loan (all
advances and payments specified by the calling application) and return the
ending balance.

To use this functionality, there are a few requirements that must be met:

* All specified payment streams must have a `PmtType` of other than `CalcPmt`.
In `AmortizeOnly` mode, the Loan Builder will not compute "regular" principal
and interest payments.

* No protection products may be requested.

* All advances must be specified, and not computed.

* `AmError` (see above) must be set to `Allow`.

Furthermore, the output for an `"AmortizeOnly" : true` Loan request will omit
the `FedBox` and `Moneys` response elements.

🟦 **BusinessRules.CanSkipFirst** - *Boolean : false {optional}*

Set this field to `true` to allow the first payment of a loan to be skipped.
The default value of `false` does not allow a skipped first payment.

🟦 **BusinessRules.CanSkipLast** - *Boolean : false {optional}*

Set this attribute to `true` to allow the last payment of a loan to be skipped.
The default value of `false` does not allow a skipped final payment.

🟦 **BusinessRules.ClosedFormEqn** - *Boolean : true {optional}*

When searching for the payment that best amortizes the loan to zero, this setting
determines whether or not interest is rounded during the payment search algorithm.
The default value of `true` indicates that the closed form equation should be matched,
and interest will be left unrounded during the payment search algorithm. Setting this
attribute value to `false` will cause interest to be rounded during the payment search
algorithm.

🟦 **BusinessRules.CurrencyDP** - *String (0, 2) : see below' {optional}*

The number of decimal places allowed for currency values is defined by this
attribute. If this attribute is not set up, the default value will be 
determined by the value of the `Country` fiwld. For most countries,
the default value is `"2"`. If no country code is specified, then the
default value for this attribute is `"2"`.

---
</details>

🟦 **Country** - *String : (Alpha-2 or Numeric-3 code) : "US" { optional }*

This field allows the calling application to specify a two (2) character or
three (3) digit country code. If none is provided, then a default value of `US`
will be used. Please see the [Countries Appendix](appendix-countries.md) for
the list of supported countries and their associated codes.

Specifying the `Country` will also set the default value for the `APR.Code`
and `BusinessRules.CurrencyDP` fields, as appropriate for the country specified.

🟦 **Construction** - *Object { optional }*

If the requested loan calculation features a construction period, during which
time interest only payments are made by the borrower, then the request need to
include this object and set it's fields appropriately. Note that construction
periods always begin on the date of the first advance.

Generally speaking, there are two methods used to disclose construction loans.
The first method computes the total estimated interest due during the
construction period, and treats it as an unfinanced prepaid fee. The second
method discloses discrete interest only payments in the amortization schedule
during the construction period.

To emulate the first method, simply include the `Construction` object and
ensure that no payment events are set up to occur during the construction
period. The construction interest will be computed and returned in the
`ConInterest` response object with the `IsPrepaid` field set to `true`.

To emulate the second method, set up an interest only payment stream which
begins during the construction period, with the final interest only payment
occurring on the construction period's ending date. The interest only payments
will then appear in the resulting amortization schedule, and the total
construction interest will be returned in the `ConInterest` response object
with the `IsPrepaid` field set to `false`.

<details>
<summary><b>Construction fields</b></summary>

---

🟦 **Construction.HalfCommitment** - *Boolean : false {optional}*

During the construction period, if estimated interest should be disclosed based
upon half of the commitment amount, then set the value of this field to `true`.
If interest is due on the entire commitment amount during the construction
period, then set this value to `false`. Multiple advance construction loans are
computed with this attribute set as `true`.

🟥 **Construction.EndDate** - *String (Date) {required}*

The date on which the construction period terminates.

All dates must be in the form of "YYYY-MM-DD", and be 10 characters long.
Hence, to end the construction period on July 4, 2021, the attribute would be
specified as `"EndDate" : "2021-07-04"`.

If the calling application sets up discrete interest only payments during the
construction period, then the `EndDate` may also be specified as occurring on
a given payment number (`NNNN`) as `"EndDate" : "NNNN-00-00"`.

---
</details>

🟦 **ODI** - *Object { optional }*

If odd days should be treated as a prepaid finance charge *or* added to the
first payment in a manner different from how interest is accruing (see 
`BusinessRules.OddFirstPmt`), then the request needs to define how odd days
interest is computed and handled using the fields of this object.

<details>
<summary><b>ODI fields</b></summary>

---

---
</details>

## Loan Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Loan Module** *(hosted on AWS)*

```json
{
  "Result" : 200,
  "Module" : "Loan",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Notes" : [
    ],
    "FedBox" : {
      "AmtFin" : "950.00",
      "FinChg" : "104.75",
      "TotPmts" : "1054.75",
      "APR" : {
        "Value" : "19.766",
        "Type" : "Actuarial"
      }
    },
    "Moneys" : {
      "Proceeds" : "1000.00",
      "Principal" : "1000.00",
      "Interest" : "54.75",
      "Prepaid" : "50.00"
    },
    "Accrual" : {
      "Methods" : [
        "Actual/365 USRule"
      ],
      "Days1Pmt" : "31",
      "DayCount" : "Actual",
      "Maturity" : "2020-01-01"
    },
    "PmtStreams" : [
      {
        "Term" : "11",
        "Pmt" : "87.90",
        "Rate" : "10.000",
        "Begin" : "2019-02-01",
        "PPY" : "12"
      },
      {
        "Term" : "1",
        "Pmt" : "87.85",
        "Rate" : "10.000",
        "Begin" : "2020-01-01"
      }
    ],
    "AmTable" : {
      "AmLines" : [...]
    }
  }
}```

The `Data` object for a response from the loan module is defined below:

🟥 **Errors[]** - *Array of String : { required }*

The `Errors[]` field contains an array of Strings which describe any errors encountered
while handling the request. If the length of the `Errors[]` Array is zero (0), then the
module processed the request successfully, and the Data object can be further processed
by the calling application for the returned data.

On the other hand, if the length of the `Errors[]` Array is greater than zero (0), then
this indicates that an error condition has been detected, and the calling application
should *not* process the respons Data object further. In this case, the contents of the
`Errors[]` array will describe the error(s) encountered.

Typical errors include the omission of *🟥 required* fields, invalid field values, etc.

🟥 **Warnings[]** - *Array of String : { required }*

The `Warnings[]` field contains an array of Strings which describe any warnings generated
by the module handling the request. The most common warnings returned by modules inform
the calling application that the module does not recognize a specified field (which may
help to isolate a field name spelling error in the calling application's code). Note that
field names which start with "//" will bre treated as comment fields by the SCEJSON, and
no warnings will be generated for these unrecognized fields.

**Example - Request and response illustrating warnings when passing unrecognized fields** *(hosted on AWS)*

```json
{
  "Module" : "Loan",
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
  "Module" : "Loan",
  "Data" : {
    "Errors" : [
      "Data.Advances[] (Array) not found.",
      "Data.AccrualConfigs[] (Array) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}```

🟥 **Notes[]** - *Array of String : { required }*

The `Notes[]` field contains an array of Strings which present important
calculation comments to more fully explain the loan calculation results that
may either not be apparant in the inputs or have overriden input assumptions.
These notes are thoroughly detailed in the [Calculation Notes Appendix](appendix-calcnotes.md).


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Version Module](module-version.md) | [SCEJSON Reference Manual](README.md) | [APR Calculation & Verification Module](module-apr.md) |
