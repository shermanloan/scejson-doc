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

### 🟦 Apr

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Settings related to the computed effective rate returned by the SCE (most
commonly, the Regulation Z APR in the United States of America) are contained
in the fields of this object.

<details><summary><b>Apr fields</b></summary>

---
🟦 **Apr.Code**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 10, 20, 30, 40, 41, 50, 60, 100, 301, 302, 303, 304, 305, 306, 307, 308, 309, 310, 311, 320, 321, 330, 331, 340 | see below |

The method of APR computation is defined by this field. If
this field is not included, the default method depends upon the
value of the `Country` field. For the United States of America,
the default value is `100` (Actuarial). For a country in the European Union, the
default value is `50` (European Union APR). For Canada, the default value
is `60`.

| Apr Code | Description |
| :---:    |   :---      |
|   10  | Account opening disclosure of open-end Loan |
|       | (APR will equal the Interest Rate) |
|   20  | Microsoft\textregistered Excel extended internal rate of return (XIRR) |
|   30  | Unit period 360 internal rate of return (XIRR360) |
|   40  | Unit period internal rate of return (IRR) |
|   41  | Yield unit period internal rate of return (IRR) |
|   50  | European Union APR |
|   60  | Canadian APR |
|  100  | Actuarial |
|  301  | UnitPeriod/360 US Rule |
|  302  | UnitPeriod/365 US Rule |
|  303  | UnitPeriod/DaysPerPeriod US Rule |
|       | (e.g. 91 for quarterly payment frequencies) |
|  304  | UnitPeriod True360/360 US Rule. |
|  305  | UnitPeriod True360/365 US Rule. |
|  306  | UnitPeriod True360/DaysPerPeriod (e.g. 91 for |
|       | quarterly payment frequencies) |
|  307  | UnitPeriod/Federal Calendar US Rule |
|  308  | UnitPeriod/365.25 US Rule |
|  309  | UnitPeriod True360/365.25 US Rule |
|  310  | Actual/360 US Rule |
|  311  | True365/360 US Rule |
|  320  | Actual/365 US Rule |
|  321  | True365/365 US Rule |
|  330  | Actual/Actual US Rule |
|  331  | Midnight366 US Rule |
|  340  | Actual/365.25 US Rule |

---

🟦 **Apr.Decimals**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 3, 4, 5 | see below |

The number of decimal places of accuracy for the disclosed APR is determined by
this field. The default value of this field is `3`.

---

🟦 **Apr.MAPR_Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 36 |

If you are computing the Military APR (see `UseMAPR` below) and wish to override
the default maximum APR value of 36%, then specify the desired maximum as the
value of this field.

---

🟦 **Apr.SinglePayFraction**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | InDays, InMonths | InDays |

 This field only applies to single advance, single payment transactions of term
 less than one year. For these loans, when the term of the loan is a number of months,
 Appendix J allows for the term of the loan to be expressed as either a number of months
 over twelve, or the number of actual 24-hour days in the loan over 365. For the former,
 use "InMonths". For the latter, use "InDays".

---

🟦 **Apr.StrictTime**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When monthly payments are intended for the 29th or 30th of the month, how should
the number of unit periods and fraction be calculated in February, when the day
of the payment cannot be the intended day? (e.g. A non-leap year February where
payments are meant for the 29th, but end up on the 28th). The `StrictTime` field
acknowledges and disambiguates the calculation. Regular payments generally hold
the fraction of a unit period constant, based on the date of the first payment.
The SCE allows for a constant fraction of a unit period or a strict time
accounting, acknowledging that in February, the fraction of a unit period
changes.

When the value of this field is `true`, the SCE will calculate the fraction of a
unit period strictly according to the rules of Appendix J of Regulation Z.

A value of `false` instructs the SCE to keep the fraction of a unit period
constant, equal to the fraction obtained by the time calculated from the date of
the first payment to the transaction date.

---

🟦 **Apr.UseMAPR**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to a value of `true`, then the SCE will compute the
Military APR in addition to the Regulation Z APR. The `MAPR` object will be
included in the loan response.

---
</details>

### 🟦 BusinessRules

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Many loan calculation *business rules* may be toggled using the fields of this
object.

<details><summary><b>BusinessRules fields</b></summary>

---  
🟦 **BusinessRules.AmError**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Allow, AdjPmt, AdjPrin, AdjInt | Allow |

When amortizing a loan, often times there is a non-zero ending balance due to
rounding. This field determines what to do (if anything) with a non-zero
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
  
---
  
🟦 **BusinessRules.AmortizeOnly**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

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

* `AmError` (see above) must be set to `"Allow"`.

Furthermore, the output for an `"AmortizeOnly" : true` Loan request will omit
the `FedBox` and `Moneys` response elements.
  
---
  
🟦 **BusinessRules.CanSkipFirst**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Set this field to `true` to allow the first payment of a loan to be skipped.
The default value of `false` does not allow a skipped first payment.
  
---
  
🟦 **BusinessRules.CanSkipLast**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Set this field to `true` to allow the last payment of a loan to be skipped.
The default value of `false` does not allow a skipped final payment.
  
---
  
🟦 **BusinessRules.ClosedFormEqn**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

When searching for the payment that best amortizes the loan to zero, this setting
determines whether or not interest is rounded during the payment search algorithm.
The default value of `true` indicates that the closed form equation should be matched,
and interest will be left unrounded during the payment search algorithm. Setting this
field value to `false` will cause interest to be rounded during the payment search
algorithm.
  
---
  
🟦 **BusinessRules.CurrencyDP**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0, 2 | 2 |

The number of decimal places allowed for currency values is defined by this
field. If this field is not included, the default value will be 
determined by the value of the `Country` field. For most countries,
the default value is `"2"`. If no country code is specified, then the
default value for this field is `"2"`.
  
---
  
🟦 **BusinessRules.LeapYearRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is `true` and one of the actual/actual 
interest accrual calendars (including Midnight366) is used in the requested
loan, then the interest accrued on a 365 days basis is rounded separately from
the interest accrued on a 366 days basis. Note that this option is encountered
*very* rarely in the marketplace.

If the value of this field is `false`, then interest computed on a 365 day
basis will be added to the interest computed on a 366 day basis, and then
rounded.
  
---

🟦 **BusinessRules.MYED**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | MM-DD | n/a |

The `MYED` field represents the Mortgage Year End Date when computing an annual
rest loan, a loan which is commonly encountered in the United Kingdom. If you do
not need to compute an annual rest loan, then this field need not be included in
the request.

If included in the request, then the value of the field must be in the
format of `"MM-DD"` when `MM` is a valid month number from 1 to 12, and `DD` is
a valid day number in the month specified. The month and day specified is the
annual date on which escrowed payments and interest are applied to the
outstanding principal balance.

When the `MYED` is specified, you must also specify a `PmtStream` object with a
`PmtType` field value of `PmtEsc`, which indicates an escrowed payment.

---

🟦 **BusinessRules.PmtAccrualCode**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, 201, 202, 203, 204, 205, 206, 210, 211, 220, 221, 230, 231, 250, 301, 302, 303, 304, 305, 306, 308, 309, 310, 311, 320, 321, 330, 331, 350 | default |

In rare instances, the calculated payment for a given loan is based upon a
different accrual calendar than the one used to amortize interest in the
amortization schedule. This is most commonly encountered when a system is not
able to compute the "best" payment, and instead uses a simpler accrual
calendar with a closed form equation (such as unit period simple). An
estimated payment is computed, then used in the amortization schedule where
interest is accrued on a different calendar. Note that this will almost
always generate an ending balance of greater magnitude that the "correct"
payment, unless the `BusinessRules.AmError` field is used to adjust the final
payment.

A value of `default` means that the payment will be computed using the same
accrual calendar as interest. Any other value will cause the computed payment
to be calculated using the accrual code specified.

---

🟦 **BusinessRules.ProtMandatory**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is set to `true`, then protection premiums/fees
will be considered to be a part of the Finance Charge, and thus affect the Regulation
Z APR. 

---

🟦 **BusinessRules.UsRuleAprException**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

The US Rule APR Exception is an option used by some financial institutions that
accrue interest in a US Rule manner, and also disclose the US Rule APR. If this
field is omitted or set to `false`, the US Rule APR will always be computed.

If the value of this field is set to `true` and the following conditions are
met: (i) interest is accrued via a US Rule method, (ii) A US Rule APR method
matching the acrual method has been specified, (iii) the Regulation Z Finance
Charge is equal to the interest charge, and (iv) there is only a single
`AccrualConfig` object, then the US Rule APR will be disclosed as the interest
rate.

---

🟦 **BusinessRules.YieldPPY** 

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0, 2, 4, 6, 12 | 0 |

Canadian mortgages may compute periodic interest using a fractional power of a
periodic yield. If set to a value other than "0", this value determines the
period. Please contact us for further information if you support mortgage
calculations in Canada. Note that when using this field with a value other
than zero, the calling application *must* include an odd days prepaid fee
in the request.

---

🟦 **BusinessRules.ZeroInterestRule**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is `true` and (i) the interest rate for the entire
term of the loan is 0%, or (ii) the computed interest charge for the loan is
less than zero, then the interest charge will be set to a value of zero, and the
Regulation Z Finance Charge will be adjusted in an identical manner.

Note that setting the value of this field to `true` will likely cause the
sum of the principal and interest to no longer equal the total of payments
(similarly for the Amount Financed and Finance Charge). J. L. Sherman and
Associates does not recommend that you set the value of this field to `true`
before consulting with your legal counsel in regards to these and other
unforseen consequences.

Lastly, all payment rounding is forced to be rounded down to the nearest penny.
This adjustment ensures that the sum of payments never exceeds the Principal
Balance of the loan.

---
</details>

### 🟦 Construction

| Type  | Required |
| :---: |   :---:  |
| Object | no |

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

🟦 **Construction.HalfCommitment**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

During the construction period, if estimated interest should be disclosed based
upon half of the commitment amount, then set the value of this field to `true`.
If interest is due on the entire commitment amount during the construction
period, then set this value to `false`. Multiple advance construction loans are
computed with this field set as `true`.

---

🟥 **Construction.EndDate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

The date on which the construction period terminates.

All dates must be in the form of "YYYY-MM-DD", and be 10 characters long.
Hence, to end the construction period on July 4, 2021, the field would be
specified as `"EndDate" : "2021-07-04"`.

If the calling application sets up discrete interest only payments during the
construction period, then the `EndDate` may also be specified as occurring on
a given payment number (`NNNN`) as `"EndDate" : "NNNN-00-00"`.

---
</details>

### 🟦 Country

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Alpha-2 or Numeric-3 code | US |

This field allows the calling application to specify a two (2) character or
three (3) digit country code. If none is provided, then a default value of `US`
will be used. Please see the [Countries Appendix](appendix-countries.md) for
the list of supported countries and their associated codes.

Specifying the `Country` will also set the default value for the `APR.Code`
and `BusinessRules.CurrencyDP` fields, as appropriate for the country specified.

### 🟦 EditOutput

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Changes to the presentation of the output data are contained in the
fields of this object.

<details>
<summary><b>EditOutput fields</b></summary>

---

🟦 **EditOutput.EarlyPayoffDate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD or NNNN-00-00 | n/a |

If an early payoff date is specified and is in the format of a valid date (i.e.
`YYYY-MM-DD`), then the loan will first be computed normally, ignoring this
value. Once this first step is done, the loan will then be amortized and the
early payoff date applied by removing all loan events after the early payoff
date, ensuring that a final payment date falls on the specified date.

If the early payoff date is specified and is in the format of `NNNN-00-00`, then
the early payoff date will be set to the date of the N'th payment, and the loan
will be computed as described above.

One common use of this field is to generate computed regular payments based upon
a longer amortization term, with a final balloon payment made on the early
payoff date (commonly called a balloon loan with a long term amortization and
short term call).

All dates must be in the form of YYYY-MM-DD, and be 10 characters long. Hence,
to end an early payoff date on July 1, 2021, the field would be specified as
`"EarlyPayoffDate" : "2021-07-01"`. To specify an early payoff date that
coinsides with the 60th payment, the field would be specified as
`"EarlyPayoffDate" : "0060-00-00"`.

---

🟦 **EditOutput.KeepSlush**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Rounding interest each period numerically eliminates values beyond two decimal places. This amount
is referred to as slush. To keep this amount and add it to next period's unrounded interest, set
the value of this field `true`.

---

🟦 **EditOutput.Merge**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

If the value of this field is set to `true`, then payment elements will be
merged together whenever possible. For example, if one event is of type
`CalcPmt`, and another on the same day is `PayPrin`, then this field determines
whether or not these two events will be combined into a single payment event or
left as separate and distinct events.

---

🟦 **EditOutput.PmtDollarRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Payments are normally rounded to the penny, according to the method specified by
the `EditInterest.PmtRound` field. If the payment should be rounded to the
dollar instead of the penny, then set the value of this field to `true`.

Note that for some loans (such as those with longer terms or relatively small
proceeds), rounding the payment up or to the nearest dollar may require a
shortened loan term to prevent one or more negative payments at the end of the
loan.

---

🟦 **EditOutput.ShowAmTable**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

To supress the entire amortization schedule from the response, set value of this
field to `false`; otherwise, the amortization schedule will be returned.

---

🟦 **EditOutput.ShowFees**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to `false`, then fees will not explicitly appear in the amortizaton
schedule, unless they do not occur on the date of an advance or the date of a payment. They will
also not be individually listed under the `Moneys` object of the response.

A value of `true` means that all fees will be explicitly accounted for, both in the
`Moneys` response object as child elements and in the amortization table. The
`Type` field will have the name of the fee as its value.

---

🟦 **EditOutput.ShowGrandTot**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

To show the amortization schedule grand totals in the response, set this field's value to `true`;
otherwise, the grand totals will not be returned.

---

🟦 **EditOutput.ShowSubTot**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

To show the amortization schedule annual subtotals in the XML output, set this field's value to `true`;
otherwise, annual subtotals will not be returned.

---

🟦 **EditOutput.ShowTap**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | see below |

If not specified, the default value of this field is determined by the
`Country` specified. If the `Country` is `GB` (United Kingdom), then the default
value is `true`. All other country values will default this field to
`false`.

The value of this field determines if the total amount payable response field
(i.e. `TAP`) will be computed and disclosed. This quantity is broadly defined as
the total of payments plus fees that do not enter into the amortization table in
any way.

---

🟦 **EditOutput.ShowType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Each line of the amortization schedule is characterized by a type, which describes the particular
amortization event. An `EditInterest` event is different from a `FixedPmt` event, for example. Set
this field to `true` to report the `Type` of each amortization event. A value of `false`
will suppress output of the `Type` field in the amortization schedule.

---

🟦 **EditOutput.TagPmts**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is set to `true`, then each `PmtStream` response object will
include an `Idx` field which identifies the input payment stream object which gave rise to it.
If this field is set to `true`, then the `EditOutput.Merge` field must be set to `false`. If
both are set to `true`, then an error will be returned.

---
</details>

### 🟦 ODI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

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
}
```

The `Data` object for a response from the loan module is defined below:

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
}
```

### 🟥 Notes

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |

The `Notes[]` field contains an array of Strings which present important
calculation comments to more fully explain the loan calculation results that
may either not be apparant in the inputs or have overriden input assumptions.
These notes are thoroughly detailed in the [Calculation Notes Appendix](appendix-calcnotes.md).


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Version Module](module-version.md) | [SCEJSON Reference Manual](README.md) | [APR Calculation & Verification Module](module-apr.md) |
