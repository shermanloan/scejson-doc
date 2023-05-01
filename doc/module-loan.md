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

The fields of the `Data` object supported by a Loan module request are defined
in alphabetical order below:

### 🟥 AccrualConfigs

| Type  | Required |
| :---: |   :---:  |
| array of AccrualConfig objects | yes |

This array of `AccrualConfig` objects must have at least one member, and
is used to specify how interest is calculated.

<details><summary><b>AccrualConfig fields</b></summary>

---

🟦 **AccrualConfig.Capitalize**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If interest due is to be added to principal (capitalized) on the specified
`Date`, then set this field's value to `true`.

🟦 **AccrualConfig.Code**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | see table | Text - See below |

The method of interest accrual is defined by this field. A value of `default`
means one of two things: if no accrual method has beend defined at all, use the
system default of `201`. If an accrual code has been previously defined, then
the SCE will continue to use that method. Please see the following table of
accrual codes and their descriptions for the meanings of each code.

Add-On interest methods 401...406 have restrictions that apply. Only equal
payment loans with one AccrualConfig object may use add-on interest. In
addition, construction loans may not be used with add-on interest. Also, any
protection products requested must also be single premium.

| Code | Description |
| :---: | :--- |
| 100 | UnitPeriod/Federal Calendar Actuarial |
| 101 | UnitPeriod/Toyota Motor Credit Actuarial |
| 201 | UnitPeriod/360 Simple |
| 202 | UnitPeriod/365 Simple |
| 203 | UnitPeriod/DaysPerPeriod Simple (e.g. 91 for quarterly payment frequencies) |
| 204 | UnitPeriod True360/360 Simple |
| 205 | UnitPeriod True360/365 Simple |
| 206 | UnitPeriod True360/DaysPerPeriod Simple (e.g. 91 for quarterly payment frequencies) |
| 207 | UnitPeriod/Federal Calendar Simple |
| 208 | UnitPeriod/365.25 Simple |
| 209 | UnitPeriod True360/365.25 Simple |
| 210 | Actual/360 Simple |
| 211 | True365/360 Simple |
| 220 | Actual/365 Simple |
| 221 | True365/365 Simple |
| 230 | Actual/Actual Simple |
| 231 | Midnight 366 Simple |
| 240 | Actual/365.25 Simple |
| 250 | UnitPeriod/VarDPY Simple |
| 301 | UnitPeriod/360 US Rule |
| 302 | UnitPeriod/365 US Rule |
| 303 | UnitPeriod/DaysPerPeriod US Rule (e.g. 91 for quarterly payment frequencies) |
| 304 | UnitPeriod True360/360 US Rule |
| 305 | UnitPeriod True360/365 US Rule |
| 306 | UnitPeriod True360/DaysPerPeriod US Rule (e.g. 91 for quarterly payment frequencies) |
| 307 | UnitPeriod/Federal Calendar US Rule |
| 308 | UnitPeriod/365.25 US Rule |
| 309 | UnitPeriod True360/365.25 US Rule |
| 310 | Actual/360 US Rule |
| 311 | True365/360 US Rule |
| 320 | Actual/365 US Rule |
| 321 | True365/365 US Rule |
| 330 | Actual/Actual US Rule |
| 331 | Midnight 366 US Rule |
| 340 | Actual/365.25 US Rule |
| 350 | UnitPeriod/VarDPY US Rule |
| 401 | UnitPeriod/360 Add-On |
| 402 | UnitPeriod/365 Add-On |
| 403 | UnitPeriod/DaysPerPeriod Add-On (e.g. 91 for quarterly payment frequencies) |
| 404 | True360/360 Add-On |
| 405 | True360/365 Add-On |
| 406 | True360/DaysPerPeriod Add-On 91 for quarterly payment frequencies) |

🟦 **AccrualConfig.Date**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD or NNNN-00-00 | Date of the last Advance |

Set the date of this event. If the date is omitted, the system will use
the date of the last `Advance` object in the `Advances` array.

All dates must be in the form of YYYY-MM-DD, and be 10 characters long.
Hence, to change the interest rate on January 2, 2021, the field
would be specified as `"Date" : "2021-01-02"'.

However, to provide power and flexibility to the calling application, the SCE
will also understand `Date` values in the following formats:

- **NNNN-00-00** If you wish to change an interest accrual parameter on the
 date of a specific payment number (N), use a value of NNNN-00-00. Thus, an
 value of `0012-00-00` instructs the SCE to set the date of this
 event equal to that of the 12'th payment. A vale of 0000-00-00 will set the
 date of the event equal to the date of the first advance.

🟦 **AccrualConfig.ExtraDays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

Increase or decrease the number of days between this event and the next event by
the value of this field. e.g. `1` will be considered one more day of
interest.

🟦 **AccrualConfig.IntRate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - % | Text - See below |

Defines the interest rate that applies at and beyond this event. If no `IntRate`
is specified, the previously defined interest rate is used. A value of zero will
be used if no previous `IntRate` has been defined.

🟦 **AccrualConfig.IntRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | nearest, up, down | nearest |

Defines how interest is to be rounded.

🟦 **AccrualConfig.NewPmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the payment should change to reflect a new interest rate, set the value of
this field to `true`; otherwise, the payment will not change after this event.
If only one `AccrualConfig` object is being specified, then this field
should be omitted altogether.

🟦 **AccrualConfig.PmtRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | nearest, up, down, best | nearest |

How are calculated payments to be rounded at and beyond this event? `best` means
to choose the payment that returns a minimal ending balance at the end of
amortization. If two payments result in the same minimal error magnitude, the
smaller payment is chosen.

Note: If the `BusinessRules.ZeroInterestRule` field is set to 'true' and all
interest rates are zero, payments are required to be rounded down. This rule
ensures the total of payments never exceeds the amounts of the loan to pay off
(principal balance).

🟦 **AccrualConfig.Tiers**

| Type  | Required |
| :---: |   :---:  |
| array of Tier objects |

To compute interest using a split rate method, where different rates are applied
to different parts of the balance, the calling application will need to specify
a `Tier` object with its associated `Rate` and `Ceiling`
fields. Note that the interest rate which is used above the final `Ceiling`
is the one specified in the `AccrualConfig` object. Furthermore, the 
`Tier` objects must be ordered in ascending order based upon the
`Ceiling` amount.

As en example, assume that the calling application needs to charge 20% on the first $100,
15% up to $250, and 10% for the remaining balance. The AccrualConfig object needed to implement this split
rate structure is:

```json
{
  "AccrualConfigs" : [
    {
    "AccrualCode" : "201",
    "Date" : "2020-04-01",
    "IntRate" : "10.000",
    "Tiers" : [
      { "Rate" : "20.0", "Ceiling" : "100.00" },
      { "Rate" : "15.0", "Ceiling" : "250.00" }
    ]
    }
  ]
}
```

🟥  **AccrualConfig.Tier.Ceiling**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

Defines the upper bound to which the specified split rate tier will apply.

🟥  **AccrualConfig.Tier.Rate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - % | n/a |

Defines the interest rate that applies to this split rate tier.

---
</details>

### 🟥 Advances

| Type  | Required |
| :---: |   :---:  |
| array of Advance objects | yes |

This array of `Advance` objects must have at least one member, and is used to
specify the cash advances made to the borrower.

<details><summary><b>Advances fields</b></summary>

---

🟥 **Advance.Amount**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

The proceeds to be advanced to the borrower is defined using this field. If the
calling application requests that the advance be computed (see the
`Advance.Compute` field below), then the value of this field will be added
to the computed advance amount, which can be useful in multiple advance loans.

🟦 **Advance.Compute**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Some applications like to provide useful "what if?" calculations, such as "how
much can I afford to borrow given a specified loan term, interest rate, and
payment stream?" Here at J. L. Sherman and Associates, we call this a "Roll to
Advance" calculation.

If the value of this field is `true`, then the calling application is
requesting that the SCE calculate one or more advances given a specified term,
interest rate, and payment stream. With this in mind, all `PmtStream` objects
(which define the specified payment streams in the loan) may not have a
`PmtType` of `CalcPmt`. Usually, all payment streams will be defined using a
`PmtType` of `FixedPmt`.

We have provided several samples which illustrate various "Roll to Advance"
calculations in the `/samples` directory, for your reference.

🟥 **Advance.Date** (Date)

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field's value holds the date on which the advance is made. All dates must
be in the form of YYYY-MM-DD, and be 10 characters long. Hence, an advance date
of January 2, 2021 would be specified as `"Date" : "2021-01-02"`.

🟦 **Advance.ExtraDays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

Increase or decrease the number of days between this event and the next event by
the value of this field. e.g. `1` will be considered one more day of interest.

---
</details>

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
| String | no | 0, 10, 20, 30, 40, 41, 50, 60, 100, 301, 302, 303, 304, 305, 306, 307, 308, 309, 310, 311, 320, 321, 330, 331, 340 | Text - See below |

The method of APR computation is defined by this field. If
this field is not included, the default method depends upon the
value of the `Country` field. For the United States of America,
the default value is `100` (Actuarial). For a country in the European Union, the
default value is `50` (European Union APR). For Canada, the default value
is `60`.

| Apr Code | Description |
| :---:    |   :---      |
|   0  | Not computed |
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

🟦 **Apr.Decimals**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 3, 4, 5 | Text - See below |

The number of decimal places of accuracy for the disclosed APR is determined by
this field. The default value of this field is `3`.

🟦 **Apr.MAPR_Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - % | 36 |

If you are computing the Military APR (see `UseMAPR` below) and wish to override
the default maximum APR value of 36%, then specify the desired maximum as the
value of this field.

🟦 **Apr.SinglePayFraction**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | InDays, InMonths | InDays |

 This field only applies to single advance, single payment transactions of term
 less than one year. For these loans, when the term of the loan is a number of months,
 Appendix J allows for the term of the loan to be expressed as either a number of months
 over twelve, or the number of actual 24-hour days in the loan over 365. For the former,
 use "InMonths". For the latter, use "InDays".

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
the `FedBox` and `Moneys` response objects.
  
🟦 **BusinessRules.CanSkipFirst**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Set this field to `true` to allow the first payment of a loan to be skipped.
The default value of `false` does not allow a skipped first payment.
  
🟦 **BusinessRules.CanSkipLast**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Set this field to `true` to allow the last payment of a loan to be skipped.
The default value of `false` does not allow a skipped final payment.
 
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

🟦 **BusinessRules.MinFinChg**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

This field allows the calling application to specify a minimum allowed
finance charge for the loan. If a value greater than zero is specified, then the
loan will be computed in the normal manner, along with the finance charge. If
the computed finance charge falls below the specified minimum, then the final
payment's interest charge will be adjusted to meet the specified minimum.

Note that the minimum interest charge is checked *before* the minimum
finance charge. If the loan triggered a minimum finance charge adjustment, then
the amount of this adjustment will be disclosed in the `MinFinChgAdj` field
of the `Moneys` response object.

Note that since the minimum finance charge check is done after the loan payment
and protection calculations have been completed, certain protection products
whose calculations may vary due to a modified final payment will not reflect any
minimum finance charge adjustments made to the final payment.

🟦 **BusinessRules.MinIntChg**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

This field allows the calling application to specify a minimum allowed
interest charge for the loan. If a value greater than zero is specified, then the
loan will be computed in the normal manner, along with the interest charge. If
the computed interest charge falls below the specified minimum, then the final
payment's interest charge will be adjusted to meet the specified minimum.

Note that the minimum interest charge is checked *before* the minimum
finance charge. If the loan triggered a minimum interest charge adjustment, then
the amount of this adjustment will be disclosed in the `MinIntChgAdj` field
of the `Moneys` response object.

Note that since the minimum interest charge check is done after the loan payment
and protection calculations have been completed, certain protection products
whose calculations may vary due to a modified final payment will not reflect any
minimum interest charge adjustments made to the final payment.

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

🟦 **BusinessRules.OddFirstPmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | true, false, OnlyPositive | false |

If the first payment should be adjusted (up or down) due to odd days interest
in the same manner that interest is accruing in the loan, then set the value
of this field `true`. A value of `OnlyPositive` instructs
the SCE to only adjust the first payment due to odd days if there are positive
odd days. Finally, a value of `false` instructs the SCE to not adjust
the first payment using this method.

Note that you can not specify an odd first payment using this attribute 
*and* include `ODI` in the same request.

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

🟦 **BusinessRules.ProtMandatory**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is set to `true`, then protection premiums/fees
will be considered to be a part of the Finance Charge, and thus affect the Regulation
Z APR. 

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

### 🟦 Capitalization

| Type  | Required |
| :---: |   :---:  |
| array of Capitalize objects | no |

Some loans require interest to be capitalized on specific dates, irrespective of
any other considerations. For these events, use one or more `Capitalize`
objects.

<details>
<summary><b>Capitalize fields</b></summary>

---

🟦 **Capitalize.AllowFeb29**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

This attribute is used when generating event dates in a capitalization stream
(if any) beyond the specified `Date` date of the `Capitalize` object. If the
value of this attribute is `true`, then February 29th is an allowed date.

On the other hand, if the value of this attribute is set to `false` and a computed
date in the given capitalization stream is scheduled to fall on February 29th,
then the scheduled date will be altered in one of the following manners:

- If the number of payments per year is 1, 2, 4, 6, 12, or 24 then the
 scheduled date of February 29 will be moved back one day to February 28.
 This is the only date that will be adjusted.
- If the number of payments per year is 26 or 52 then the scheduled
 date of February 29 will be moved forward one day to March 1, and all subsequent
 dates will be a multiple of 7 or 14 days from this date (depending upon
 the specified payment frequency).

🟥 **Capitalize.Date**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

The `Date` field for the `Capitalize` object is used to specify the date on
which to capitalize interest. (For a stream of Capitilize events, see the `Term`
field.) As an example, if interest is to be capitalized on Feb. 1, 2008, the
field should be specified as `"Date" : "2008-02-01"`.

🟦 **Capitalize.Holidays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, next | ignore |

If event dates (including the specified start date) are not allowed to fall on
specified holidays (see the `Holidays[]` array for more inforation on
how to specify holidays), then set the value of this field to something other
than `ignore`. The meaning of the other two values are defined below:

- **prev -** The date will be moved to the day before the holiday.
- **next -** The date will be moved to the Monday after the holiday.

Note that two other `Capitalize` fields can cause the computed dates to be
adjusted: `AllowFeb29` and `Weekends`. The rules for how these three fields
interact are described below in detail.

For every date required in our capitalization stream, the SCE goes through the
following steps:

1. Generate the next date in our date stream, called the target date.
2. If February 29th is not allowed and if the target date is 2/29, then
 adjust the target date forward or backward one day based upon the date
 generation frequency. If the frequency is a monthly multiple, then the target
 date is moved backward one day to 2/28. Weekly and biweekly frequencies move the
 target date forward one day to 3/1.
3. If weekend dates are not allowed (e.g. the `Weekends` field holds a
 value other than `ignore`) and the target date falls on a Saturday or Sunday:
    1. then adjust the target date according to the field value.
    2. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 3(a) above.
4. If holidays are not allowed (e.g. the `Holidays` field holds a value
 other than `ignore`) and the target date falls on a specified holiday:
    1. Then adjust the target date according to the `Holidays` field value.
    2. If weekend dates are not allowed (e.g. the `Weekends` field holds a
     value other than `Ignore`) and the target date falls on a Saturday or Sunday,
     then adjust the target date in the same direction as 4(a) above.
    3. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 4(a) above.
    4. Step 4 is repeated until the target date is not adjusted.

🟦 **Capitalize.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

This field is used to resolve ambiguitiess in subsequent event
dates when the `Date` date falls on the last day of a month
with fewer than 31 days.

Set this field's value to `true` if the intent was to make
subsequent events occur on the last day of the month. A value of
`false` indicates that subsequent event dates will fall on the
day number specified by the `Date`.

As an example, if the calling application specifies a `Date`
of "2010-02-28", then subsequent payment dates for a monthly
payment frequency will be:

- **on the 28th of each subsequent month** if `"LastDay" : false`.
- **on the last day of each subsequent month** if `"LastDay" : true`.

🟦 **Capitalize.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

`PPY` is an abbreviation for "payments per year", and as one might surmise,
determines the frequency of capitalization events. If only one capitalization
event occurs or the stream of capitilization events are monthly, this attribute
may be ignored.

🟦 **Capitalize.SemimonthlyDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

When specifying a semimonthly stream, the day number on which the first
capitalization event occurs is the day number for all of the following odd
numbered payments. If you omit this field or specify a value of `0`, then the
even numbered events will be generated using the default method within the SCE.

If you wish to specify the day number on which even numbered events fall
(overriding the default method used in the SCE), then set the value of this
field to the desired day number. Setting the value of this field to `31` will
cause all even events to fall on the last day of the month.

As an example, if you want to specify a semi-monthly capitalization stream
beginning on January 1, 2019 with events that fall on the 1st and 15th of each
month for 24 payments, the object should look something like this:

```json
{
  "Capitalization" : [
    {
      "Date" : "2019-01-01",
      "Term" : "24",
      "PPY" : "24",
      "SemimonthlyDay" : "15"
    }
  ]
}
```

🟦 **Capitalize.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1 |

The `Term` field indicates the the number of capitalization events to be made at
the specified payment frequency (see `Capitalize.PPY` above). The default value
is `1`, and if only one capitalization event is required, this field may be
omitted.

🟦 **Capitalize.Weekends**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, near, next | ignore |

If capitalization stream dates (including the specified start `Date`) are not
allowed to fall on a Saturday or Sunday, then set the value of this field to
something other than `ignore`. The meaning of the other three values are defined
below:

- **prev** - The date will be moved to the Friday before the computed weekend
  date.
- **near** - The date will be moved to the Friday before the computed weekend
  date if the computed weekend date falls on a Saturday, otherwise it will be
  moved to the Monday after.
- **next** - The date will be moved to the Monday after the computed weekend
  date.

Note that two other `Capitalize` fields can cause the computed dates to be
adjusted - `AllowFeb29` and `Holidays`. For a complete description of how these
three fields work together, please see the documentation for the `Holidays`
field.

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
and `Format.CurrencyDecimals` fields, as appropriate for the country specified.

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

🟦 **EditOutput.KeepSlush**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Rounding interest each period numerically eliminates values beyond two decimal places. This amount
is referred to as slush. To keep this amount and add it to next period's unrounded interest, set
the value of this field `true`.

🟦 **EditOutput.Merge**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

If the value of this field is set to `true`, then payment objects will be
merged together whenever possible. For example, if one event is of type
`CalcPmt`, and another on the same day is `PayPrin`, then this field determines
whether or not these two events will be combined into a single payment event or
left as separate and distinct events.

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

🟦 **EditOutput.ShowAmTable**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

To supress the entire amortization schedule from the response, set value of this
field to `false`; otherwise, the amortization schedule will be returned.

🟦 **EditOutput.ShowFees**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to `false`, then fees will not explicitly appear in the amortizaton
schedule, unless they do not occur on the date of an advance or the date of a payment. They will
also not be individually listed under the `Moneys` object of the response.

A value of `true` means that all fees will be explicitly accounted for, both in the
`Moneys` response object as child objects and in the amortization table. The
`Type` field will have the name of the fee as its value.

🟦 **EditOutput.ShowGrandTot**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

To show the amortization schedule grand totals in the response, set this field's value to `true`;
otherwise, the grand totals will not be returned.

🟦 **EditOutput.ShowSubTot**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

To show the amortization schedule annual subtotals in the XML output, set this field's value to `true`;
otherwise, annual subtotals will not be returned.

🟦 **EditOutput.ShowTap**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | Text - See below |

If not specified, the default value of this field is determined by the
`Country` specified. If the `Country` is `GB` (United Kingdom), then the default
value is `true`. All other country values will default this field to
`false`.

The value of this field determines if the total amount payable response field
(i.e. `TAP`) will be computed and disclosed. This quantity is broadly defined as
the total of payments plus fees that do not enter into the amortization table in
any way.

🟦 **EditOutput.ShowType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Each line of the amortization schedule is characterized by a type, which describes the particular
amortization event. An `EditInterest` event is different from a `FixedPmt` event, for example. Set
this field to `true` to report the `Type` of each amortization event. A value of `false`
will suppress output of the `Type` field in the amortization schedule.

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

### 🟦 Fees

| Type  | Required |
| :---: |   :---:  |
| array of Fee objects | no |

This array of `Fee` objects allows the calling application to specify one ore
more Fee objects to be added to the loan.

<details>
<summary><b>Fee fields</b></summary>

---

🟦 **Fee.AddToFinChg**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this fee should be included in the computed Finance Charge (and hence, affect
the APR), then set this field to `true`. The default value of `false` indicates
that the fee does not affect the Finance Charge nor APR.

🟦 **Fee.AddToPrin**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this fee should be added to the principal balance (e.g. the fee is financed
along with the advance(s)), then set this field to `true`. If set to `false`,
then the fee is paid up front out of the borrower's pocket.

🟦 **Fee.Adjust**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

The optional `Adjust` field allows the calling application
to increase or decrease the base amount on which a fee is calculated.
If a negative value is specified and this adjustment would produce a
negative base amount, then a value of zero is returned for the given fee.
This field has no effect on fees with a `CalcType` of `Dollar`.

As an example, in Tennessee you may need to define a financed, non-APR
affecting fee to be computed by decreasing the amount financed by $2,000,
and then multiplying this reduced amount by 0.115. The way to accomplish
this in the SCE is as follows:


```json
{
  "Fees" : [
    {
      "Name" : "TN Fee",
      "CalcType" : "OnAmtFin",
      "Adjust" : "-2000.00",
      "Amount" : "0.115",
      "AddToPrin" : true,
      "AddToFinChg" : false
    }
  ]
}
```

🟦 **Fee.AllowFeb29**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

This field is used when generating fee dates in a fee stream (if any) beyond the
specified `Date` date of the `Fee` object. If the value of this field is
`true`, then February 29th is an allowed fee date.

On the other hand, if the value of this field is set to `false` and a
computed date in the given fee stream is scheduled to fall on February 29th,
then the scheduled fee date will be altered in one of the following manners:

- **If the number of payments per year is 1, 2, 4, 6, 12, or 24** then the
 scheduled fee date of February 29 will be moved back one day to February 28.
 This is the only date that will be adjusted.
- **If the number of payments per year is 26 or 52** then the scheduled fee date
 of February 29 will be moved forward one day to March 1, and all subsequent fee
 dates will be a multiple of 7 or 14 days from this date (depending upon the
 specified payment frequency).

🟥 **Fee.Amount**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | 0 |

How this field is interpreted depends upon the `Fee.CalcType` field.
Please see the documentation for this field for further information.

🟦 **Fee.Blended** (true, false, 1, 0) `false' \{optional\}}

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

The `Blended` field determines how the SCE will include the fee with a payment
or advance.

A value of `true` in an advance tells the SCE that the amount of the advance
already includes the fee. A value of `false` means to add the fee on top of the
existing advance.

Similarly, if the Fee occurs on a payment date, then `true` means that the
payment has the fee already included in it, whereas a value of `false` means the
fee is to be added on top of the payment.

🟦 **Fee.CalcType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Dollar, OnProceeds, OnPrincipal, OnAmtFin, OnBalance, OnBalanceFlat | Dollar |

This field specifies how the fee is to be computed, as described in the following table.

| Fee Calc Type | Description |
| :--- | :--- |
| Dollar        | The `Amount` field is understood as a flat dollar amount. |
| OnProceeds    | The `Amount` field is understood as a percentage value, to be applied to the loan's proceeds, defined as the sum of advances. An `Amount` of `0.25` would represent a fee of 0.25% of the total proceeds. |
| OnPrincipal   | The `Amount` field is understood as a percentage value, to be applied to the loan's principal balance. An `Amount` of `0.125` would represent a fee of 0.125% of the principal balance. |
| OnAmtFin      | The `Amount` field is understood as a percentage value, to be applied to the loan's Regulation Z Amount Financed. An `Amount` of `0.33` would represent a fee of 0.33% of the amount financed. |
| OnBalance     | The `Amount` field is understood as an annual percentage value, to be applied to the current balance of the loan when the fee is assessed, and accrued using the interest accrual calendar in effect. An `Amount` of `12.01` would represent a fee accrued at an annual rate of 12.0% of the current balance. |
| OnBalanceFlat | The `Amount` field is understood as a flat percentage value, to be applied to the current balance of the loan when the fee is assessed. An `Amount` of `1.0` would represent a fee of 1.0% of the current balance.|

🟦 **Fee.Date**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD or 0000-MM-00 | Date of the last Advance |

The date on which a fee occurs. If this object defines a stream of more than one
fee, then this field defines the date on which the first fee occurs. If the date
is the same as an advance or payment, the fee is loaded into the advance or
payment event; otherwise, the fee is a stand-alone event. All dates must be in
the form of YYYY-MM-DD, and be 10 characters long. Hence, a fee date of January
2, 2021 would be specified as `"Date" : "2021-01-02"`.

A special format is accepted for annual fees (`Fee.PPY` value of `1`) paid on a
specific calendar month of the year. Using `0000-MM-00` as a value for `Date`
instructs the SCE to pay the fee on the MM payment of the payment stream. For
example, `0000-06-00` instructs the system to pay this fee on June Payments.

If the `Date` field is not specified, then it will default to the last specified
Advance `Date`.

🟦 **Fee.EqualServChg**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When a fee is set up as a service charge of type `ByTerm` (see `Fee.ServChgType`
field), this field determines if all the service charge fee payments should be
forced to be equal (a value of `true`), or if the final service charge fee
payment can be different from the others due to rounding (a value of `false`).

🟦 **Fee.Holidays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, next | ignore |

If fee dates (including the specified start date) are not allowed to fall on
specified holidays (see the `Holidays[]` array for more inforation on how to
specify holidays), then set the value of this field to something other than
`ignore`. The meaning of the other two values are defined below:

- **prev -** The date will be moved to the day before the holiday.
- **next -** The date will be moved to the Monday after the holiday.

Note that two other `Fee` fields can cause the computed dates to be adjusted:
`AllowFeb29` and `Weekends`. The rules for how these three fields interact are
described below in detail.

For every date required in our payment stream, the SCE goes through the
following steps:

1. Generate the next date in our date stream, called the target date.
2. If February 29th is not allowed and if the target date is 2/29, then
 adjust the target date forward or backward one day based upon the date
 generation frequency. If the frequency is a monthly multiple, then the target
 date is moved backward one day to 2/28. Weekly and biweekly frequencies move the
 target date forward one day to 3/1.
3. If weekend dates are not allowed (e.g. the `Weekends` field holds a
 value other than `ignore`) and the target date falls on a Saturday or Sunday:
    1. then adjust the target date according to the field value.
    2. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 3(a) above.
4. If holidays are not allowed (e.g. the `Holidays` field holds a value
 other than `ignore`) and the target date falls on a specified holiday:
    1. Then adjust the target date according to the `Holidays` field value.
    2. If weekend dates are not allowed (e.g. the `Weekends` field holds a
     value other than `Ignore`) and the target date falls on a Saturday or Sunday,
     then adjust the target date in the same direction as 4(a) above.
    3. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 4(a) above.
    4. Step 4 is repeated until the target date is not adjusted.

🟦 **Fee.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

This field is used to resolve ambiguitiess in subsequent fee
dates when the `Date` falls on the last day of a month
with fewer than 31 days. If the value of the `Date` field
is not a valid date, then the value of this field is ignored. If
the value of the `Term` field is `1`, then the value of
this field is ignored.

Set this field's value to `true` if the intent was to make
subsequent fees occur on the last day of the month. A value of
`false` indicates that subsequent fee dates will fall on the
day number specified by the `Date` field.

As an example, if the calling application specifies a `Date`
of `2010-02-28`, then subsequent fee dates for a monthly
fee frequency will be:

- **on the 28th of each subsequent month** if `"LastDay" : false`.
- **on the last day of each subsequent month** if `"LastDay" : true`.

🟦 **Fee.MAPR***

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If you wish to compute the Military APR, then certain fees may not be considered
Regulation Z APR affecting fees, but are considered Military APR affecting fees.
In this case, you will need to set the value of this field to true.

Fees which are added to the finance charge (e.g. `"AddToFinChg" : true` are
always considered MAPR fees, regardless of the stated value of this field.

Note that debt protection products are automatically included in the calculation
of the Military APR, no matter what method is used for payment (e.g. single
premium vs. monthly outstanding balance).

🟦 **Fee.Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

If a maximum value for the fee is specified and is greater than zero, then if
the computed fee exceeds this maximum value, then the maximum value will be used
instead. If no maximum value is specified or is set to a value of zero, then no
maximum will be enforced. Please note that this field is applied to *all* fee
types supported. Also, note that a specified maximum value is checked *after*
enforcing a specified minimum value, and hence a specified maximum value trumps
a specified minimum.

🟦 **Fee.Min**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

If a minimum value for the fee is specified and is greater than zero, then if
the computed fee is less than this minimum value, then the minimum value will be
used instead. If no minimum value is specified or is set to a value of zero,
then no minimum will be enforced. Please note that this field is applied to
*all* fee types supported. Also, note that a specified minimum value is checked
*before* enforcing a specified maximum value, and hence a specified maximum
value trumps a specified minimum.

🟦 **Fee.Name**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Text | empty |

This field is for convenience purposes only, and does not affect the calculation
of the fee in any manner. However, the value of this field *will* be used to
identify the fee in the response, and hence it is highly recommended that you
name your fees accordingly.

🟦 **Fee.Position**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Default, OnAdvance, BeforePmt, InPmt, AfterPmt, NotInPmt | Default |

Fees inflence a loan depending on when they are applied.

- **OnAdvance** means that the fee is included in the application of an advance.
- **BeforePmt** means that when the fee occurs on the same date as a payment, the
fee is calculated before the payment amortizes the loan.
- **InPmt** means that the fee is included as part of the payment.
- **AfterPmt** means that when the fee occurs on the same date as a payment, the
payment first amortizes the loan, then the fee is calculated and applied.
- **NotInPmt** means that the fee is paid on the same date as a payment but
is not reflected in that payment. If the fee has `"AddToFinChg" : true`,
it is called a "Pocket APR" fee; otherwise, it is a straight "Pocket Fee".
- **Default** means different things, depending on when the fee occurs:
  - A fee on an advance is treated as if `OnAdvance` were selected.
  - A fee on a payment is treated as if `InPmt` were selected.

🟦 **Fee.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and in the case of the Fee
object, determines the frequency for the fee stream. If the value of the `Term`
field is `1`, then the value of this field is ignored.

🟦 **Fee.Round**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | nearest, up, down | nearest |

The value of this field, along with the with the `Fee.RoundBasis`
field described below, determine how the amount on which the fee is
based upon is to be rounded.

For fee calculation types of `OnPrincipal`, `OnProceeds`, `OnAmtFin`, `OnBalance`, and
`OnBalanceFlat`, the amount on which the fee is based is fairly self
evident: the starting principal balance, proceeds, amount financed, or the outstanding balance at the time
that the fee is assessed. Thus, if the calling application requires a fee to be
computed based upon 0.35% of the principal balance rounded to the nearest $100,
then the fee object would look something like this:

```json
{
  "Fees" : [
    {
      "Name" : "Doc Stamp",
      "CalcType" : "OnPrincipal",
      "Amount" : "0.35",
      "Round" : "nearest",
      "RoundBasis" : "100.00",
      "AddToPrin" : true,
      "AddToFinChg" : false
    }
  ]
}
```

For the `Dollar` fee calculation type, the amount on which the fee is
based is defined as the fee amount passed into the engine. This allows the
calling application to pass in a precomputed fee amount and have the SCE round
it as directed. As an example, here is the definition of a dollar fee which is
rounded up to the next dollar (the value of the fee below would then be $25
after rounding):

```json
{
  "Fees" : [
    {
      "Name" : "Doc Stamp",
      "CalcType" : "Dollar",
      "Amount" : "24.25",
      "Round" : "up",
      "RoundBasis" : "1.00",
      "AddToPrin" : true,
      "AddToFinChg" : false
    }
  ]
}
```

For the `ServChg` fee calculation type, the amount on which the fee is
based is defined as the computed service charge amount, as defined by the
`Amount` and `ServChgType` fields.

🟦 **Fee.RoundBasis**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Floating | 0.01 |

When rounding the amount on which the fee is based, the value of this field
determines the precision to which rounding is performed. The default value of
`0.01` indicates that the rounding should be done to the penny, whereas a value
of `100.0` indicates that rounding should take place to the nearest hundred
dollars.

🟦 **Fee.SemimonthlyDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0...31 | 0 |

When specifying a semimonthly fee stream, the day number on which the first fee
is added determines the day number for all of the following odd numbered
payments. If you omit this field or specify a value of `0`, then the even
numbered fees will be generated using the default method within the SCE. If the
value of the `Term` field is `1`, then the value of this field is
ignored.

If you wish to specify the day number on which even numbered fees fall
(overriding the default method used in the SCE), then set the value of this
field to the desired day number. Setting the value of this field to `31`
will cause all even fees to fall on the last day of the month.

As an example, if you want to specify a semi-monthly $10 fee stream beginning on
January 1, 2015 with fees that fall on the 1st and 15th of each month for 48
fees, the object should look something like this:

```json
{
  "Fees" : [
    {
      "Name" : "Test Fee",
      "Date" : "2015-01-01",
      "Amount" : "10.00",
      "Term" : "48",
      "PPY" : "24",
      "SemimonthlyDay" : "15"
    }
  ]
}
```

🟦 **Fee.ServChgType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | None, ByTerm, ByDays | None |

This field dictates whether or not a fee is treated as a service charge, and if so,
determines how the service charge dollar amount is paid off over the term of the loan.
If a fee should not be treated as a service charge, then the default value of `None`
should be specified.

- **ByTerm** means that the service charge is paid off by dividing the amount by the `Term`
field of the same `Fee` object.
- **ByDays** means that the service charge is paid off on payment dates. The amount of the fee
is calculated as the number of days from the previous payment (or advance) divided by the
total number of days in the term of the fee.

🟦 **Fee.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1 |

The `Term` field determines the the number of fees to be included at the
specified frequency (see `Fee.PPY` above), after which the fee stream is
completed. The default value is `1`. If the value of the `Date` field is not a
valid date, then the value of this field is ignored.

🟦 **Fee.Weekends**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, near, next | ignore |

If fee dates (including the specified start `Date`) are not allowed to fall on a
Saturday or Sunday, then set the value of this field to something other than
`ignore`. The meaning of the other three values are defined below:

- **prev** - The date will be moved to the Friday before the computed weekend
  date.
- **near** - The date will be moved to the Friday before the computed weekend
  date if the computed weekend date falls on a Saturday, otherwise it will be
  moved to the Monday after.
- **next** - The date will be moved to the Monday after the computed weekend
  date.

Note that two other `Fee` fields can cause the computed dates to be adjusted -
`AllowFeb29` and `Holidays`. For a complete description of how these three
fields work together, please see the documentation for the `Holidays` field.

---

</details>

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
number of decimal places allowed after the `DecimalSeparator`. If this field is
not included, the default value will be determined by the value of the `Country`
field. For most countries, the default value is `"2"`. If no country code is
specified, then the default value for this field is `"2"`.


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

### 🟦 Holidays

| Type  | Required |
| :---: |   :---:  |
| array of String | no |

If dates for `PmtStream` and `Fee` objects have requested that holidays be
skipped (e.g. the `Holidays` field for the object in question is set to a value
other than `ignore`, then the request must specify each holiday to be
considered, with each holiday requiring at least one entry in this `Holidays[]`
array.

The format of the `Holidays[]` array member must follow one of the three
following descriptions:

- **YYYY-MM-DD -** A holiday can be defined by sending in a valid date, where
 YYYY is greater than or equal to 1900, MM is a valid month, and DD is a valid
 day for the given year and month.
- **0000-MM-DD -** A holiday that occurs annually on the same month and day can
 be defined by sending in a valid month and day, with the year passed in as
 `0000`. As an example, in the United States of America, Christmas is a Federal
 holiday celebrated on December 25th of each year. To pass this holiday in to
 the SCE, the `Holidays[]` array member would look like `"0000-12-25"`.
- **0001-MM-PD -** A holiday that occurs annually in the same month on a given
 day of the week (`D`) in a given position in the month (`P`) can be defined by
 sending in a valid month, position, and day of the week, with the year passed
 in as `0001`. The value of `D` can be from 0 to 6, where 0 = Sunday, 1 =
 Monday, ..., 6 = Saturday. The value of `P` can be from 1 to 6, where 1 = 1st,
 2 = Second, ..., 5 = 5th, 6 = last. As an example, in the United States of
 America, Thanksgiving is a Federal holiday celebrated on the 4th (`P=4`)
 Thursday (`D=4`) of November (`MM=11`) every year. To pass this holiday in to
 the SCE, the `Holiday[]` array member would look like `"0001-11-44"`.
- **0002-NN-NN -** A holiday that occurs annually, but can not be expressed
 using any of the above methods. Currently, the following special holidays are
 recognized by the SCEX:
   - **0001 -** Good Friday
   - **0002 -** Easter Sunday
   - **0003 -** Easter Monday

### 🟦 MI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `MI` objectt determines if this loan includes one of the supported types of
mortgage insurance (MI) -- PMI or FHA. This object contains fields which further
specify mortgage insurance options.

*NOTE:* Mortgage insurance is only supported on equal, balloon, and interest
only loan builder requests at this time.


<details>
<summary><b>MI fields</b></summary>

---

🟦 **MI.CashDown**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

The `CashDown` field represents a cash down payment made at closing. If
specified and greater than zero, this amount will be deducted from the principal
balance. If not specified, the cash down payment will default to zero.

🟦 **MI.LoanAmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

This field represents the amount by which the PMI rates are multiplied to
produce a level PMI premium. If not specified, then the principal balance will
be used to compute the annual premium.

🟦 **MI.Periodic**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The fields of this object determine the conditions when MI is no longer
included.

<details>
<summary><b>Periodic fields</b></summary>

---

🟦 **Periodic.DropLTV**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - % | 0 |

The value of this field determines the loan to value ratio at which
MI should be removed, and is expressed as a percentage. For example,
if you wish to automatically drop MI when the loan to value ratio first
equals or falls below 78%, then you would specify
`"DropLTV" : "78.0"`.

🟦 **Periodic.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

The value of this field represents the the term (in payments) beyond 
which MI will be removed.

As an example, if mortgage insurance must be removed after the
180th payment, then the calling application should specify

```json
{
  "MI" : {
    "Periodic" : { "Term" : "180" }
  }
}
```

🟦 **Periodic.WarnLTV**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - % | 0 |

The value of this field determines the loan to value ratio at which a
warning should be issued, and is expressed as a percentage of LTV (loan to value).
For example,if you wish to know when the loan to value ratio first equals or
falls below 80%, then you would specify `"WarnLTV" : "80.0"`.

---

</details> 

🟥 **MI.PropertyValue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field's value represents the appraised property value, and will be used in
the calculation of the loan to value ratio.

🟥 **MI.Rates[]**

| Type  | Required |
| :---: |   :---:  |
| array of Rate objects | yes |

This array if `Rate` objects defines the rates used to compute MI on the loan. There must
be at least one `Rate` object in the `Rates[]` array, and there can be more than one if
rates are set to switch at a specified payment number.

<details>
<summary><b>Rate fields</b></summary>

---

🟥 **Rate.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The value of this field specifies the cost of mortgage insurance per $100
of the loan amount. As an example, a loan computed with a MI rate of $1.50  per $100 would be
specified as 

```json
{
  "MI" : {
    "Rates" : [
      { "Rate" : "1.50" }
    ]
  }
}
```

🟦 **Rate.Switch**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

This optional attribute defines the payment number beyond which the associated MI rate
will apply. If not specified, the value will default to zero.

Thus, if there is only a single MI rate, one may omit this attribute (see example above).

Example of a MI multiple rate structure (use a rate of $1.50 for the first 10 years, with a rate
of $0.20 for coverage beyond 10 years):

```json
{
  "MI" : {
    "Rates" : [
      { "Rate" : "1.50" },
      { "Rate" : "0.20" , "Switch" : "120" }
    ]
  }
}
```

---

</details>

🟦 **MI.Type**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | fha, pmi | pmi |

This field determines the type of mortgage insurance to include with the loan.
If the value of this field is set to `fha`, then a different MI premium is
computed every twelve months based upon the average outstanding principal
balance during that same term. The MI calculation adheres strictly to the [HUD
regulation](http://portal.hud.gov/hudportal/HUD?src=/program_offices/housing/comp/premiums/sfpcalc).

If the value of this field is set to `pmi`, then each defined MI rate produces a
level MI premium based upon the inital loan amount.

🟦 **MI.UpFront**

| Type  | Required |
| :---: |   :---:  |
| object | no |

This object determines if there is an up front fee for mortgage insurance, and
if so, how it is computer. If this object is not present in the request, then no
up front fee will be computed. Note that ZOMP (zero option monthly premium)
products do not have an up front fee, which means that this object should be
omitted.

<details>
<summary><b>UpFront fields</b></summary>

---

🟦 **UpFront.Paid**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | atclosing, bylender, financed | atclosing |

If the value of this field is set to `financed`, then the computed up front
fee will be added to the principal balance and the finance charge. A value of `atclosing`
will cause the value of the fee to be added to the finance charge alone. Finally, a value
of `bylender` means that the up front fee is to be paid by the lender, hence the value
of the fee is not added to either the principal balance nor the finance charge.

🟦 **UpFront.Reduce**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

If the specified number of periodic premiums to include as an up front fee is
greater than zero, then this attribute determines if the term of coverage for
PMI will be reduced by the same amount.

🟦 **UpFront.Units**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | months, points, years | months |

If the `Units` field is set to `months`, then `UpFront.Value` represents the
number of periodic MI premiums to be paid at closing.

If the `Units` field is set to `Years`, then `UpFront.Value` represents the
number of annual MI premiums to be paid at closing. Many single premium products
define the up front fee as a number of years of MI to be paid up front.

Finally, if the `Units` field is set to `Points`, then `UpFront.Value`
represents the percentage of principal to be paid up front. As of October 3,
2011, FHA loans use points.

🟦 **UpFront.Value**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Floating | 0 |

If part of the MI fees are to be paid up front, then value of this field must be
grater than zero. How the value of this field is interpreted depends upon the
`Units` field (see below).

The default value of zero means that no up front fee will be computed.

</details>

</details>

---

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

🟦 **ODI.AccrualCode**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 204, 205, 210, 211, 220, 221, 230, 231, 250 | default |

The accrual code defines how the odd days interest is computed. The meaning of the
allowed accrual codes is defined below.

Note that accrual code `250` ("Variable Days Per Year") defines the
basis divisor days to be equal to 12 multiplied by the number of days in the month
of the date on which interest begins to accrue. Thus, if the interest start date
falls in November, then the number of basis days is 360. If the interest start
date falls in a month with 31 days, then the number of basis days is 372. For
an interest start date in February, the number of basis days will either be 336
or 348, depending upon whether or not it is a leap year.

| Accrual Code | Description |
| :---: | :---- |
| 204 | True360/360 |
| 205 | True360/365 |
| 210 | Actual/360 |
| 211 | True365/360 |
| 220 | Actual/365 |
| 221 | True365/365 |
| 230 | Actual/Actual |
| 231 | Midnight 366 |
| 250 | Actual/Variable Days Per Year |

🟦 **ODI.AddToPmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the calling application wants the odd days interest to be added to the first
payment, then set the value of this field to `true`. A value of `false`
indicates that the odd days interest will be treated as a prepaid finance
charge.

🟦 **ODI.AddToPrin**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If any odd days interest should be treated as a financed prepaid fee, then set
the value of this field to `true`. Note that if both `AddToPmt` and `AddToPrin`
are set to `true`, then a warning message will be returned by the SCE and the
value of `AddToPrin` will be set to `false`.

🟦 **ODI.AnchorDate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | BackUnitPeriod, BackDaysPerPeriod | BackUnitPeriod |

The computed number of odd days is the number of days between the loan date and
the anchor date. This field determines how to arrive at the anchor date. A value
of `BackUnitPeriod` means that the anchor date is one unit period prior to the
specified first payment date. A value of `BackDaysPerPeriod` means that the
anchor date is the number of days per period prior to the first payment date.
Please note that for both of these methods, the period used will be that
associated with the payment stream in which the first payment occurs.

🟦 **ODI.ForceUnitPeriod**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

Some unit period methods will not use a strict unit period interest accrual
factor in the period to the first payment. For example, code `302` will count
the days to the first payment and divide by 365. For a monthly loan, setting
this field to `true` will use a 1/12 factor instead of Days/365.

🟦 **ODI.UseDailyCost**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the total odd days prepaid fee is computed by first generating a rounded (to
the nearest penny) daily cost and then multiplying this value by the computed
number of odd days, then set the value of this property to `true`.

A value of `false` means that the daily cost is left unrounded, and the total
prepaid fee is rounded after the computation is complete.

🟦 **ODI.UseNegODI**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If there are negative odd days in the loan, then the value of this field
determines if a negative odd days interest fee is computed. If the value of this
field is `false`, then negative odd days fees are not allowed, the SCE will
return a value of zero in this situation, and the computed payment will be
adjusted to take into account the negative odd days. A value of `true` will
return a negative odd days interest fee (in effect, it then becomes and odd days
interest credit) when there are negative odd days in a loan.

---

</details>

### 🟦 PmtStreams

| Type  | Required |
| :---: |   :---:  |
| array of PmtStream objects | no |

Payment streams are defined using the fields of this empty object,
including individually defined payments.

<details>
<summary><b>PmtStream fields</b></summary>

---

🟦 **PmtStream.AllowFeb29**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

This field is used when generating payment dates (if any) beyond the specified
`PmtStream.Date`. If the value of this field is `true`, then February 29th is an
allowed payment date.

On the other hand, if the value of this field is set to `false` and a computed
date in the given payment stream is scheduled to fall on February 29th, then the
scheduled payment date will be altered in one of the following manners:

- **If the number of payments per year is 1, 2, 4, 6, 12, or 24** then the
 scheduled payment date of February 29 will be moved back one day to February 28.
 This is the only date that will be adjusted.
- **If the number of payments per year is 26 or 52** then the scheduled payment
 date of February 29 will be moved forward one day to March 1, and all
 subsequent payment dates will be a multiple of 7 or 14 days from this date
 (depending upon the specified payment frequency).

🟦 **PmtStream.Amount**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency, Number - Floating +"%", or Number - Floating + "%B" | 0 |

The value of the `Amount` field has different meanings depending upon the
`PmtType` selected. Please see the documentation on the `PmtType` field below
for more information on how these two fields work together to define a payment
stream.

If the `PmtType` field is equal to `PayInt` or `PayPrin`, then the value of this
field may be specified as a flat dollar amount (e.g. `"Amount" : "250.00"`), a
percentage of the principal balance (e.g. `"Amount" : "5.25%"`), a percentage of
the outstanding balance at the time the payment is made (e.g. `"Amount" : "8.125%B"`),
or a percentage of the computed target payment (e.g. `"Amount" : "100%C"`).

If the `PmtType` field is equal to `CalcPmt`, then the value of this field may
be specified as a flat dollar amount (e.g. `"Amount" : "500.00"`) which will be
added to the computed payment, or a percentage of the computed payment (e.g.
`"Amount" : "200%"`), which allows the calling application to specify a double
payment, half payment, etc.

🟦 **PmtStream.ComputeTerm**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If set to `true`, then the `ComputeTerm` field is used to inform the Loan module
that the calling application is requesting an equal payment "roll to term" loan
calculation. In this loan scenario, the calling application specifies the
interest rate, amount requested, maximum desired payment amount, and maximum
loan term. The Loan module will then compute the shortest loan term to produce
an equal payment amount less than or equal to the specified maximum payment.

As an example, the following partial JSON request would direct the Loan module
to compute the term for an equal payment loan with an interest rate of 5.00%, an
advance of $5,000, a maximum desired payment amount of $300, and a maximum
allowed loan term of 60 monthly payments:

```json
{
  "Advances" : [
    {
      "Date" : "2019-01-01",
      "Amount" : "5000.00"
    }
  ],
  "AccrualConfigs" : [
    {
      "Date" : "2019-01-01",
      "IntRate" : "5.000",
      "AccrualCode" : "301"
    }
  ],
  "PmtStreams" : [
    {
      "Date" : "2019-02-01",
      "PmtType" : "FixedPmt",
      "Amount" : "300.00",
      "Term" : "60",
      "ComputeTerm" : true
    }
  ]
}
```

Note that in the above partial JSON request, the `PmtType` must be set to
`FixedPmt`, the value of the `Amount` field is set to the maximum desired
payment amount, and the value of the `Term` field is set to the maximum term
allowed.

🟥 **PmtStream.Date**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD, NNNN-00-00, 0000-MM-00, or YYYY-MM-00 | n/a |

The `Date` field for the `PmtStream` object is generally used to specify the
starting date for the payment stream in question. As an example, if the payment
stream should begin on Feb. 1, 2021, then the field should be specified as
`"Date" : "2021-02-01"`.

For payment frequencies that are monthly multiples, the day number specified
in this field specifies the desired day number of all future payments, if
this day number does not exceed the number of days in the computed month.
As an example, if the first payment is scheduled to be made in February of
2022 and the desired day for all payments is the 30th, then the field
would be specified as `"Date" : "2022-02-30"`.

However, to provide power and flexibility to the calling application, the SCE
will also understand `Date` values in the following formats:

- **NNNN-00-00** If you wish to replace a previously defined payment by
 specifying the payment number (N), use a value of NNNN-00-00. An field value of
 `0012-00-00` instructs the engine to replace the 12th payment. If the value of
 the `Term` field (T) is specified and greater than one, then the Nth through
 (N+T-1)th payments will be replaced.

 *Example:* To define a stream of 12 interest only payments followed by 48
 computed payments, the following JSON could be used:

```json
{
  "PmtStreams" : [
    {
    "Date" : "2014-12-20",
    "PmtType" : "CalcPmt",
    "Term" : "60",
    "PPY" : "12"
    },
    {
    "Date" : "0001-00-00",
    "PmtType" : "PayInt",
    "Term" : "12"
    }
  ]
}
```

- **0000-MM-00** If you wish to define monthly replacement payments, use a value
 formatted as 0000-MM-00, where MM is a two digit month number from 01 through 12.
 A value of `0000-06-00` instructs the SCE to use the defined payment for
 every payment occurring in the month of June.

*Example:* To define a stream of 60 payments with every payment in July and
 August skipped, the following JSON could be used:

```json
{
  "PmtStreams" : [
    {
    "Date" : "2014-12-20",
    "PmtType" : "CalcPmt",
    "Term" : "60",
    "PPY" : "12"
    },
    {
    "Date" : "0000-07-00",
    "PmtType" : "FixedPmt",
    "Amount" : "0"
    },
    {
    "Date" : "0000-08-00",
    "PmtType" : "FixedPmt",
    "Amount" : "0"
    }
  ]
}
```

- **YYYY-MM-00** If you wish to define replacement payments occurring in a
 specified year and month, use a value formatted as YYYY-MM-00, where MM is a
 two digit month number from 01 through 12, and YYYY is a four digit year. A
 value of `2021-08-00` instructs the SCE to use the defined payment for every
 payment occurring in August of 2021 (there can be more that one payment if the
 payment frequency is greater than monthly).

*NOTE - if the value of the `Date` field is not a valid date (i.e. it is instead
of one of the three formats mentioned above), then the `LastDay`, `Term`, and
`PPY` fields will be ignored, unless otherwise noted above.*

🟦 **PmtStream.Holidays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, next | ignore |

If payment dates (including the specified start date) are not allowed to fall on
specified holidays (see the `Holidays[]` array for more inforation on
how to specify holidays), then set the value of this field to something other
than `ignore`. The meaning of the other two values are defined below:

- **prev -** The date will be moved to the day before the holiday.
- **next -** The date will be moved to the Monday after the holiday.

Note that two other `PmtStream` fields can cause the computed dates to be
adjusted: `AllowFeb29` and `Weekends`. The rules for how these three fields
interact are described below in detail.

For every date required in our payment stream, the SCE goes through the
following steps:

1. Generate the next date in our date stream, called the target date.
2. If February 29th is not allowed and if the target date is 2/29, then
 adjust the target date forward or backward one day based upon the date
 generation frequency. If the frequency is a monthly multiple, then the target
 date is moved backward one day to 2/28. Weekly and biweekly frequencies move the
 target date forward one day to 3/1.
3. If weekend dates are not allowed (e.g. the `Weekends` field holds a
 value other than `ignore`) and the target date falls on a Saturday or Sunday:
    1. then adjust the target date according to the field value.
    2. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 3(a) above.
4. If holidays are not allowed (e.g. the `Holidays` field holds a value
 other than `ignore`) and the target date falls on a specified holiday:
    1. Then adjust the target date according to the `Holidays` field value.
    2. If weekend dates are not allowed (e.g. the `Weekends` field holds a
     value other than `Ignore`) and the target date falls on a Saturday or Sunday,
     then adjust the target date in the same direction as 4(a) above.
    3. if February 29th is not allowed and if the target date is 2/29, then
     adjust the target date in the same direction as in 4(a) above.
    4. Step 4 is repeated until the target date is not adjusted.

🟦 **PmtStream.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

This field is used to resolve ambiguitiess in subsequent payment
dates when the `Date` date falls on the last day of a month
with fewer than 31 days. If the value of the `Date` field
is not a valid date, then the value of this field is ignored.

Set this field's value to `true` if the intent was to make
subsequent payments occur on the last day of the month. A value of
`false` indicates that subsequent payment dates will fall on the
day number specified by the `Date`.

As an example, if the calling application specifies a `Date`
of `2010-02-28`, then subsequent payment dates for a monthly
payment frequency will be:

- **on the 28th of each subsequent month** if `"LastDay" : false`.
- **on the last day of each subsequent month** if `"LastDay" : true`.

🟦 **PmtStream.MinPmt** (number) \{optional\}}

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

If specified, this field will dictate a minimum payment. If a calculated payment
falls below the specified minimum payment value, then the minimum payment value
will be used instead.

When a minimum payment is used, the principal reduction of the loan will be
accelerated. In some loan scenarios, this will result in an early payoff of the
loan before the specified `Term` of the payment stream. In this situation, the
final payment will always pay off the loan, and will be smaller than the
specified minimum payment.

🟦 **PmtStream.PmtType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | FixedPmt, PayInt, PayPrin, CalcPmt | CalcPmt |

Various types of payment streams may be requested by use of this field. Please
note that the meaning of the `Amount` field changes according to the `PmtType`
selected. Each payment type is detailed below.

- **FixedPmt** - the `Amount` field represents the exact dollar amount of the
 payment.
- **PayInt** - all interest due is to be paid, and the value of the `Amount`
 field determines the principal reduction to be applied in addition to the
 payment of interest. If the value of `Amount` is zero, then this equates to an
 interest only payment.
- **PayPrin** - the value of `Amount` is treated as a pure principal reduction
 amount.
- **CalcPmt** - calculate the payment. If the value of the `Amount` field is
 greater than zero (but not specified as a percentage), then it is treated as a
 pickup amount to be paid in addition to the computed payment. If the `Amount`
 is expressed as a percentage, then the payment amount will be multiplied by the
 desired percentage (which allows for double or triple payments, etc.)
- **PmtEsc** - When computing an annual rest loan (see the `BusinessRules.MYED`
 field), the payment stream needs to specify a `PmtType` field value of `PmtEsc`
 to indicate an escrowed payment.

When two payments occur on the same date, they merge into one payment according
to the following rules:

- **PayInt and PayInt** - Add the two `Amount`s together, so long as one is an
  interest only payment, or if both types use the same method to specify the
  principal payment amount (e.g. both use fixed dollar amounts, or both us
  percentages of principal).
- **PayInt and PayPrin** - If the same conditions mentioned in the previous case
  exist, then the `Amount` of the `PayInt` event will be increased by the
  `PayPrin` `Amount`. Delete the `PayPrin` event.
- **PayInt and CalcPmt** - Replace the `CalcPmt` event with the `PayInt` event.
- **PayInt and FixedPmt** - Keep both events separate.
- **CalcPmt and PayPrin** - If the `PayPrin` `Amount` is specified as a fixed
  dollar amount, then increase the `Amount` of the `CalcPmt` by the `PayPrin`
  `Amount` and delete the `PayPrin` event.
- **CalcPmt and CalcPmt** - Add the `Amount`s together.
- **CalcPmt and FixedPmt** - Replace the `CalcPmt` event with the `FixedPmt`
  event.
- **FixedPmt and PayPrin** - If the `PayPrin` `Amount` is specified as a fixed
  dollar amount, then increase the `Amount` of the `FixedPmt` by the `PayPrin`
  `Amount` and delete the `PayPrin` event.
- **FixedPmt and FixedPmt** - Keep both events separate.
- **PayPrin and PayPrin** - Keep both events separate.

🟦 **PmtStream.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and as one might surmise,
determines the payment frequency for the payment stream. If the value of
the `Date`` field is not a valid date, then the value of this
field is ignored.

🟦 **PmtStream.SemimonthlyDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 0 |

When specifying a semimonthly payment stream, the day number on which the first
payment is made determines the day number for all of the following odd numbered
payments. If you omit this field or specify a value of `0`, then the even
numbered payments will be generated using the default method within the SCE.

If you wish to specify the day number on which even numbered payments fall
(overriding the default method used in the SCE), then set the value of this
field to the desired day number. Setting the value of this field
to `31` will cause all even payments to fall on the last day of the month.

As an example, if you want to specify a semi-monthly payment stream beginning
on January 1, 2019 with payments that fall on the 1st and 15th of each month
for 24 payments, the object should look something like this:

```json
{
  "PmtStreams" : [
    {
      "Date" : "2019-01-01",
      "Term" : "24",
      "PPY" : "24",
      "SemimonthlyDay" : "15"
    }
  ]
}
```

🟦 **PmtStream.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1 |

The term field indicates the the number of *payments* to be made at the
specified payment frequency (see `PmtStream.PPY` above), after which the payment
stream is completed. The default value is `1`. If the value of the
`PmtStream.Date` field is not a valid date, then the value of this field is
ignored, unless it is a replacement payment using the `NNNN-00-00` date format.

🟦 **PmtStream.Weekends**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ignore, prev, near, next | ignore |

If payment stream dates (including the specified start `Date`) are not allowed
to fall on a Saturday or Sunday, then set the value of this field to something
other than `ignore`. The meaning of the other three values are defined below:

- **prev** - The date will be moved to the Friday before the computed weekend
  date.
- **near** - The date will be moved to the Friday before the computed weekend
  date if the computed weekend date falls on a Saturday, otherwise it will be
  moved to the Monday after.
- **next** - The date will be moved to the Monday after the computed weekend
  date.

Note that two other `PmtStream` fields can cause the computed dates to be
adjusted - `AllowFeb29` and `Holidays`. For a complete description of how these
three fields work together, please see the documentation for the `Holidays`
field.

---
</details>

### 🟦 Protection

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If the calling application wishes to add any payment protection products
to the loan (e.g. credit insurance or debt protection), then this object
and at least one `Product` object in the `Products[]` array must be present.

<details>
<summary><b>Protection fields</b></summary>

---

🟦 **Protection.DataPath**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Text | default data path |

If this field is set, the SCE will look for a `/data` folder containing the
setup files in the path specified. Thus, if the `DataPath` is set to `/etc/sce`,
the SCE will look for the setup files in `/etc/sce/data`.

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

🟦 **Protection.Products**

| Type  | Required |
| :---: |   :---:  |
| array of Product objects | no |

For each distinct payment protection product that the calling application wishes
to add to the loan, a `Product` object is required.

As an example, if a loan was requested with single decreasing life, single
disability, single involuntary unemployment, and joint property insurances, then
the input JSON detailing the protection requests would look similar to the
following:

```json
{
  "Protection" : {
    "Products" : [
      {
        "Code" : "LI",
        "Birthdays" : [
          "1966-04-16"
        ]
      },
      {
        "Code" : "AH",
        "Birthdays" : [
          "1966-04-16"
        ]
      },
      {
        "Code" : "IU",
        "Birthdays" : [
          "1966-04-16"
        ]
      },
      {
        "Code" : "PP",
        "Birthdays" : [
          "1966-04-16",
          "1969-11-21"
        ]
      }
    ]
  }
}
```

<details>
<summary><b>Product fields</b></summary>

---

🟦 **Product.Account**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 1 |

This field specifies the account number that will be used to define the
requested product. Each account is numbered from 1 to 9,999, and each account
corresponds to a set of setup file which define numerous settings which define
the product and how it is calculated, such as the insurance rates, caps,
formulas used, etc. If this field is not specified, a default value of `1` will
be used.

🟦 **Product.Benefit**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Currency | 0 |

If you wish to specify a benefit amount less than the maximum allowed, then do
so with this field. Omitting this field will ensure that the maximum benefit
amount allowed will be used in the loan calculation. Note that if the specified
account has not been set up to allow for user-specified benefit amounts for the
product in question, or if the product itself does not have the notion of a
monthly benefit (such as life and property products), then this attribute will
be ignored.

🟥 **Product.Birthdays[]**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| array of String | yes | YYYY-MM-DD |

This array contains the date of birth for one or two borrowers requesting a
payment protection product. The type of coverage provided (e.g. single or joint)
is determined by the number of members in the array. Thus, to request single
accident and health coverage, the `Product` object (with `"Code" : "AH"`)
requires a `Birthdays[]` array with a single birthday element containing the
birthday of the borrower seeking coverage. Similarly, to request joint
decreasing life coverage, the `Product` object (with `"Code" : "CL"`) requires a
`Birthdays[]` array with two elements, each of which contains the birthday of a
borrower seeking coverage.

Since the type of coverage depends upon the number of elements present in the
array, at lease one must appear and no more than two are allowed (as the only
coverage options are single or joint).

All dates must be in the form of YYYY-MM-DD, and be 10 characters long.

🟥 **Product.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | LI, AH, IU, PP |

This field defines the type of payment protection product, and which setup files
are references for calculation, rate, and cap purposes. In the description of
each product below, "NNNN" is the `Account` number (see above, an account number
of 1 would thus be "0001").

- **LI** - The requested product is treated as decreasing credit life and will
  reference the `clNNNN.ini` setup file.
- **AH** - The requested product is treated as accident and health and will
  reference the `ahNNNN.ini` setup file.
- **IU** - The requested product is treated as involuntary unemployment and will
  reference the `iuNNNN.ini` setup file.
- **PP** - The requested product is treated as property insurance and will
  reference the `ppNNNN.ini setup file.

🟦 **Product.Coverage**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Currency | 0 |

If you wish to specify a coverage amount less than the maximum allowed, then do
so with this field. Omitting this field will ensure that the maximum coverage
amount allowed will be used in the loan calculation. Note that if the specified
account has not been set up to allow for user specified coverage amounts for the
product in question, then this attribute will be ignored.

🟦 **Product.Dismemberment**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | false |

If the account specified has been set up to offer life protection products with
optional dismemberment coverage, and if the optional dismemberment coverage is
desired, then set this field's value to `true`. Otherwise, use the default value
of `false`.

🟦 **Product.Financed**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | true |

Single premium protection products are typically financed (added to the
principal balance). If you wish to have the premiums paid up front at loan
cloasing, then set the value of this field to `false`.

🟦 **Product.Method**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Default, Gross, Net, Level, MOB, TrueMOB, CunaSP, OrdinaryLife | Default |

Some accounts are configured to offer different types of credit life products
(usually gross and net). In these accounts, this field allows the calling
application to specify which method to use for a given loan. If no method is
present in the request, then the default method will be used.

🟦 **Product.ShowFactor**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | false |

If this field is set to `true`, then the `Product.Cost` response object (member
of the `Products[]` array) will include a `Factor` field which is useful to J.
L. Sherman and Associates when debugging protection calculations.

🟦 **Product.Table**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 0 |

When requesting the accident and health product (e.g. the `Code` field is set to
`AH`), if the specified account has been set up with multiple disability or debt
cancellation plans, then this attribute determines which plan number will be
used. If no table number is specified, the first table (table zero) will be
used.

If the requested product `Code` is not `AH`, then this attribute is ignored.

🟦 **Product.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 0 |

If you need to specify a coverage term (in months or payments) less than the
maximum allowed, then do so using this field. If this field is omitted, then the
loan will be covered for the maximum term allowed. Note that if the specified
account has not been set up to allow for user specified coverage terms for this
product, then this field will be ignored.

🟦 **Product.TermUnits**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Pmts, Months | 0 |

The specified numeric coverage term (see `Term` field, above) will be
interpreted as a number of payments or months, depending upon the value of this
attribute.

🟦 **Product.UseLevelRates**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | false |

If the account specified has been set up to offer single premium credit life
using level rates instead of the normal decreasing rates, and if you wish to use
level rates instead of decreasing, then set this field's value to `true`.
Otherwise, use the default value of `false`.

---

</details>

🟦 **Protection.ShowDataPath**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | false |

This field determines if the data directory path used in the calculation
of the loan will be disclosed in the response as a field of the
`Protection` object.

---

</details>

## Loan Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Loan Module**

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

The `Data` object for a response from the loan module is defined below, in the
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

### 🟦 Country

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Alpha-2 or Numeric-3 code | US |

If the request specified a two-character or three-digit `Country` code, then
this field will be present and will contain the full name of the country
associated with the specified code. Please see the [Countries
Appendix](appendix-countries.md) for the list of supported countries and their
associated codes.

### 🟦 CurrencyDP

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0, 2 | 2 |

If the number of decimal places allowed for currency values used in the
request is a value other than `2`, then this field will be
present and will inform the application parsing the output of the correct value.
If this field is *not* present, then the number of decimal places
allowed for currency values used in the request is `2`.

### 🟦 FedBox

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object groups together all fields which contain important numerical
information, as defined in the Truth-In-Lending laws (Regulation Z). This object
will not be present if `BusinessRules.AmortizeOnly` is set to `true` in the
request.

<details>
<summary><b>FedBox fields</b></summary>

---

🟥 **FedBox.AmtFin**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The Regulation Z Amount Financed, which is defined as the amount of credit
provided to the borrower or on their behalf.

🟥 **FedBox.FinChg**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This element contains the Regulation Z Finance Charge, described as the dollar
amount the credit extension will cost the borrower.

🟥 **FedBox.TotPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The amount which the borrower will have paid when the borrower has made all
scheduled payments.

🟦 **FedBox.TAP**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `TAP` (total amount payable) object is only returned with the response if the
value of the `EditOutput.ShowTap` request field is `true`.

<details>
<summary><b>TAP fields</b></summary>

---

🟥 **TAP.Value**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The value of this field represents the total amount payable, and is computed as
the sum of: (i) the total of payments, (ii) all non-financed APR affecting fees,
(iii) all out-of-pocket non-APR affecting fees, and (iv) all out-of-pocket APR
affecting fees.

🟦 **TAP.PrepaidNF**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency|

The value of this field is the sum of all non-financed APR prepaid fees
(APR affecting fees paid on the same date as an advance). This field will
only be present if the value is greater than zero.

🟦 **TAP.Pocket**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The value of this field is the sum of all out-of-pocket non-APR affecting
fees. This field will only be present if the value is greater than zero.

🟦 **TAP.PocketAPR**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The value of this field is the sum of all out-of-pocket APR affecting fees
not paid on an advance. This field will only be present if the value is
greater than zero.

---

</details>

🟥 **FedBox.APR**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The `APR` object contains fields which return the value and APR method used.

<details>
<summary><b>APR fields</b></summary>

---

🟥 **APR.Value**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The computed APR, which is the cost of the extension of credit expressed as a
yearly rate.

🟥 **APR.Method**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field returns the APR method used to compute the reported APR.

---

</details>

🟦 **FedBox.MAPR**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `MAPR` (military APR) object is only returned with the response if the
value of the `Apr.UseMAPR` request field is `true`.

<details>
<summary><b>MAPR fields</b></summary>

---

🟥 **MAPR.Value**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The computed military APR.

🟥 **MAPR.Advance**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field returns the equivalent of the Amount Financed for the Military APR.
Specifically, it is the principal balance less any MAPR fees, debt protection,
etc.

🟥 **MAPR.Max**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

This field holds the maximum Military APR as specified in the
input XML (see `Apr.MAPR_Max`). If not specified, a default value
of 36% is assumed. The value of this field should be displayed
as a percentage. As an example, for `"Max" : "36.000"`, you would
disclose a maximum Military APR of 36%.

🟥 **MAPR.MaxExceeded**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| boolean | yes | true, false |

The value of this field indicates whether or not the current
loan exceeds the maximum allowed Military APR. As an example, if the
maximum APR has been set to 36% and the Military APR for the
returned loan was 37.125%, the `MAPR` object
would be:

```json
{
  "MAPR" : {
    "Value" : "37.125",
    "Advance" : "1350.00",
    "Max" : "36.000",
    "MaxExceeded" : true
  }
}
```

</details>

---

</details>

### 🟦 Moneys

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This element groups together those other major cash result amounts not disclosed
under the `FedBox` object, such as the principal balance, interest charge, and
fee amounts. This object will not be present if `BusinessRules.AmortizeOnly` is
set to `true` in the request.

<details>
<summary><b>Moneys fields</b></summary>

---

🟥 **Moneys.Proceeds**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field represents the sum of all Advance amounts.

🟥 **Moneys.Principal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The principal balance is the amount on which interest is accrued. The
principal balance consists of all advances requested by the borrower,
as well as any fees and/or protection products which are financed.

🟥 **Moneys.Interest**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This value of this field holds the total interest accrued during the term of the
loan, assuming the borrower will make all scheduled payments.

🟦 **Moneys.FinFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field contains the sum of all fees having `AddToPrin`
set to `true` and occuring on the date of an advance. If this
value is zero, the field will not appear in the response.

🟦 **Moneys.Prepaid**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field represents all prepaid finance charges and contains the sum of all
fees occurring on an advance and having `AddToFinChg` set to `true`. If this
value is zero, the field will not be found in the response.

🟦 **Moneys.OthNonAprFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field contains the sum of all fees having `AddToPrin`
set to `true` *not* occuring on the date of an advance. If this
value is zero, the field will not be present in the response.

🟦 **Moneys.ServiceChg**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field represents all service charge fees and contains the sum of all fees
not occurring on an advance and having `AddToFinChg` set to `true`. If this
value is zero, the field will be omitted from the response.

🟦 **Moneys.PocketFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field holds the sum of all fees which are neither financed, nor
added to the finance charge. In essence, they are paid out of the
borrower's pocket. If no out of pocket fees were requested, then this
field will not show up in the response.

🟦 **Moneys.MAPRFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field holds the sum of all fees which are Military APR fees (including 
protection products), and will only appear if the Military APR has been
requested.

🟦 **Moneys.ConInterest**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object holds the total estimated interest accrued during the
construction period, and how the construction interest is treated
in the disclosure. If no construction period was specified, then
this object will not be present in the output.

<details>
<summary><b>ConInterest fields</b></summary>

---

🟥 **ConInterest.IsPrepaid**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If the construction interest is disclosed as interest only payments in the amortization
schedule, then the value of this field will be set to `false`. Otherwise,
the value of this field will be set to `true`.

🟥 **ConInterest.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the total amount of estimated interest accrued during the construction
period.

---

</details>

🟦 **Moneys.ODI**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object, if present, contains information regarding odd days interest. If no
odd days interest was requested, then this object will not be present in the
response.

<details>
<summary><b>ODI fields</b></summary>

---

🟥 **ODI.Count**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Discloses the number of odd days computed by the SCE for the requested loan.

🟦 **ODI.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field holds the number of odd months computed by the SCE for the requested
loan when using odd days accrual method `250`. If the odd days accrual method is
a value other than `250`, then this field will not be present in the `ODI`
object of the response.

🟦 **ODI.DailyCost**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the odd days interest fee is computed using a rounded daily cost, then the
value of this field will hold that value. If the odd days interest is *not*
computed using a rounded daily cost, then this field will not be present in the
response.

🟦 **ODI.AddToPmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | no | true, false |

If the odd days interest has been added to the first payment, then this field will be
present in the response with a value of `true`. If the odd days interest has been 
treated as a prepaid finance charge, then this field will not be present and a default
value of `false` should be assumed.

🟥 **ODI.Fee**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the total amount odd days interest charge.

---

</details>

🟦 **Moneys.MinIntChgAdj**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If a minimum interest charge is configured in the account's setup files and the
final payment was adjusted to meet this minimum interest charge, then this
field will be returned in the response and will contain the value of the
minimum interest charge adjustment.

🟦 **Moneys.MinFinChgAdj**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If a minimum finance charge is configured in the account's setup files and the
final payment was adjusted to meet this minimum finance charge, then this
field will be returned in the response and will contain the value of the
minimum finance charge adjustment.

🟦 **Moneys.MortInt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field holds the total mortgage insurance fee, if not single premium. If
mortgage insurance is not requested, or if it is requested but is not treated as
single premium, then this field will not be included in the response.

🟦 **Moneys.Advances[]**

| Type  | Required |
| :---: |   :---:  |
| array of Advance objects | no |

If the requested loan computed any advance amounts (see the `Advance.Compute`
field for more information), then for each advance in the loan there will be an
Advances object in this array containing the date of the advance and the computed advance amount.

If all of the loan's advances were specified and not computed, then the
`Moneys.Advances[]` array will not be included in the response.

<details>
<summary><b>Advance fields</b></summary>

---

🟥 **Advance.Date**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The date on which the advance is made.

🟥 **Advance.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the computed advance amount.

---

</details>

🟦 **Moneys.Fees[]**

| Type  | Required |
| :---: |   :---:  |
| array of Fee objects | no |

If the requested loan computed any requested fees with the loan, and if `EditOutput.ShowFees`
is set to `true`, then for each fee in the loan there will be a
Fees object in this array containing the name of the fee and the computed fee amount.

If there were no fees requested with the loan, or if `EditOutput.ShowFees` is set to `false`,
then the `Moneys.Fees[]` array will not be included in the response.

<details>
<summary><b>Fee fields</b></summary>

---

🟦 **Fee.Name**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

If a name was provided for the fee, then it will be included here in the
disclosure for identification purposes.

🟥 **Fee.Fee**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the computed fee amount.

</details>

---

</details>

### 🟥 Accrual

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

This object groups together interest accrual information, such
as the accrual method(s) used, days to the first payment and the
loan's maturity date.

<details>
<summary><b>Accrual fields</b></summary>

---

🟥 **Accrual.Methods[]**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| array of String | yes | description of accrual method(s) used |

Each array member contains a textual description
of the interest accrual method used to compute the loan (e.g.
"Unit Period 365 Simple").

🟦 **Accrual.YieldPPY**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

If `BusinessRules.YieldPPY` in the loan's request is set to a value other than `0`,
then this field will be returned in the response with the same value passed into
the request.

🟥 **Accrual.Days1Pmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field contains the number of days between the date of the first
advance and the date of first payment, computed by one of three
methods as specified in by `Accrual.DayCount` (below).

🟥 **Accrual.DayCount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | True360, True365, Actual |

This field specifies the method used to compute the number of days from the date
of the first advance until the first payment date. `Actual` means that the
actual number of days between these two dates are used, whereas the `True360`
and `True365` methods use a 360/365 day calendar, respectively.

🟥 **Accrual.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

Holds the maturity date of the loan, which is the date on which the last
payment is scheduled. All dates are in the form of YYYY-MM-DD, and must
be 10 characters long.

---

</details>

### 🟥 PmtStreams[]

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

The `PmtStreams` array is made up of one or more `PmtStream` objects (there will
always be at least one of these elements, and there may be more than one
depending upon the loan type). The `PmtStream` objects describe the scheduled
stream of payments for the computed loan. Instead of disclosing each and every
payment individually (which can be done with the `AmTable` object), the payment
stream groups together consecutive equal payments at the same interest rate to
produce output along the lines of:

```json
{
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
  ]
}
```

Each object describes a single stream of equal payments at the same interest
rate, using the following fields to define the important properties of each
payment stream.

<details>
<summary><b>PmtStream fields</b></summary>

---

🟦 **PmtStream.Idx**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

If the value of `EditOutput.TagPmts` is set to `true` in the loan request, then
this field will appear for each `PmtStream` object. The value of this field
identifies which `PmtStreams[]` array member of the loan request gave rise to
it.

If the value of this field is `-1`, then perfect amortization was enforced (e.g.
the `BusinessRules.AmError` field of is set to `AdjPmt`).

If the value of this field is `-2`, then an early payoff event was triggered,
which is caused by (i) specifying the `EditOutput.EarlyPayoff` field, or (ii)
using whole dollar rounding which can shorten the specified term of the loan, or
(iii) specifying a minimum payment which also may shorten the specified term of
the loan.

🟥 **PmtStream.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The `Term` field holds the number of payments that make up the given payment
stream.

🟥 **PmtStream.Pmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The `Pmt` field holds the computed payment amount for this payment stream.

🟦 **PmtStream.IsSplitRate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | no | true, false |

If this payment stream accrued interest using split-rate tiers, then this field
will be present and set to `true`. Otherwise, this field will not be
returned and has an assumed default value of `false`.

🟦 **PmtStream.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - % |

Contains the interest rate used for the duration of this payment stream. If this
payment stream accrued interest using split-rate tiers, then this field will
*not* be returned.

🟥 **PmtStream.Begin**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

This field identifies the date on which the first payment for this given payment
stream is scheduled to be made. All dates are in the form of `YYYY-MM-DD`, and
must be 10 characters long.

🟦 **PmtStream.PPY**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

If the value of the `Term` field is greater than one, then the periodic payment
frequency for this payment stream is also disclosed.

---

</details>

### 🟦 MI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If mortgage insurance is present on the requested loan, then this
objectt and all required child fieldss (documented below) will
be included in the response.

<details>
<summary><b>MI fields</b></summary>

---

🟥 **MI.Rates[]**

| Type  | Required |
| :---: |   :---:  |
| array of Rate objects | yes |

Fields of each rate object disclose the rate, premium per
year, and premium per period.

<details>
<summary><b>Rate fields</b></summary>

---

🟥 **Rate.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The percentage rate used in the mortgage insurance calculation.

🟥 **Rate.PremPerYear**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The annual mortgage insurance premium amount.

🟥 **Rate.PremPerPeriod**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The periodic mortgage insurance premium amount.

---

</details>

🟦 **MI.UpFront**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The value of this field represents the up front fee paid for
mortgage insurance. If there is no up front fee, then this field
will not be present in the response.

🟦 **MI.Periodic**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Fields of this object provide the loan to value ratio, as well as the payment
indexes when the principal balance falls below the `WarnLTV` and `DropLTV`
percentage values specified in the request. This element is only present if a
corresponding `Periodic` field is found in the request.

<details>
<summary><b>Periodic fields</b></summary>

---

🟥 **Periodic.LTV**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The loan to value ratio of the computed loan, expressed as a percentage.

🟥 **Periodic.IndexToWarn**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The value of this field indicates the payment index on which the remaining
principal balance to home value ratio drops below the specified `WarnLTV`
percentage.

🟥 **Periodic.IndexToDrop**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The value of this field indicates the payment index on which the remaining
principal balance to home value ratio drops below the specified `DropLTV`
percentage. Mortgage insurance coverage *stops* after this payment.

</details>

---

</details>

### 🟦 Protection

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If protection products are requested, then this object will be present
in the response, along with the `Products` array which contains
details on each requested protection product.

<details>
<summary><b>Protection fields</b></summary>

---

🟥 **Protection.LoanType**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Equal, SinglePay, Balloon, IntOnly, PrinPlus, SkipsIrregs, HighlyIrregular |

Protection products are often limited to certain types of loan repayment
schedules, such as regular equal and balloon payment loans. Also, different
formulas and various other settings may be used for different loan repayment
types.

The value of this field returns the type of loan repayment schedule that the SCE
determines after analyzing on the computed loan. Please see the samples provided
with this documentation which illustrates how to create JSON requests which will
fall under these loan repayment classifications.

🟦 **Protection.Path**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

This optional field will hold the value of the data path used by the
SCE to find the required setup files for protection calculations. This
field will only be present if the `Proetcion.ShowDataPath` field of
the request is set to `true`.

🟥 **Protection.Products[]**

| Type  | Required |
| :---: |   :---:  |
| array of Product objects | yes |

For each distinct payment protection product that the calling application
requested with a loan, a `Product` object will be returned.

<details>
<summary><b>Product fields</b></summary>

---

🟥 **Product.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | LI, LL, AH, IU, PP |

This field defines the type of payment protection product that this element, and
all child fields, refers to. It mirrors the `Product.Code` field defined earlier
in the request section, with the addition of the `LL` code which indicates a
level credit life product.

🟥 **Product.Result**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Full, Partial, Drop |

The value of this field informs the calling application if the requested
coverage has been fully provided, partially provided (i.e. one or more coverage
caps have been exceeded and partial coverage has been extended), or completely
dropped. If coverage has been dropped, then the `Product` object will not have
any child objects (e.g. `Product.Notes`, `Product.Notes`, `Product.Cost`, etc.).

🟥 **Product.Abbrev**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field holds the protection product's abbreviation, as configured in the
appropraite setup file.

🟥 **Product.Name**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field holds the protection product's name, as configured in the appropraite
setup file.

🟦 **Product.IsDP**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | no | true, false |

This field indicates if the product has been set up as debt protection. If this
field is not present, then the default value of `false` should be used
(which indicates that the product has been set up as insurance instead).

🟦 **Product.Table**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

The `Table` field will only be present when a product is offered with differing
tables of rates. If present, the value of this field is the table number used in
the calculation.

🟦 **Product.Formula**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

The `Formula` field contains an abbreviated description of the formula used to
compute the desired protection product. The formula codes are for the use of the
J. L. Sherman and Associates, Inc. support team. Please see the table below which
lists each formula code and description.

| Formula Code | Description |
| :--- | :--- |
| None | No coverage requested |
| Unknown | Unknown formula code |
| SP_Gross_Standard              | Single premium standard gross |
| SP_Gross_AllLevelAsDecreasing  | Single premium gross on each individual payment |
| SP_Gross_Discounted            | Single premium discounted gross |
| SP_Gross_RemPmts               | Single premium gross, sum of partial premiums computed on outstanding payments vis a vis actuarial net |
| SP_Gross_Level                 | Single premium gross using level life |
| SP_Gross_Level_EachPmt         | Single premium gross, each payment is covered with level life as if each were a single payment note |
| SP_Gross_Level_AsDecreasing    | Not currently used |
| SP_Gross_Level_RemPmts         | Single premium gross, balance at truncated term covered with level |
| SP_Net_Standard                | Single premium standard net |
| SP_Net_Discounted              | Single premium discounted net |
| SP_Net_StraightLine            | Single premium straight-line net |
| SP_Net_MainTruncation          | Single premium net, Maine truncation formula |
| SP_Net_AmericanBankers         | Single premium net, American Bankers formula |
| SP_Net_PekinTruncation         | Single premium net, Pekin insurance truncation formula |
| SP_Net_WarmSprings             | Single premium net, Warm Springs formula |
| SP_Net_StraightLine_AmerRepub  | Single premium straight-line net, American Republic formula|
| SP_Net_RemBals                 | Single premium net, sum of partial premiums, each computed as rate times balance |
| SP_Net_Level                   | Single premium net with level |
| SP_Net_LevelAtCap              | Single premium net, level at the cap |
| MOB_Net     | Standard MOB net |
| MOB_Gross   | Standard MOB gross |
| MOB_Benefit | Standard MOB on the benefit |
| TrueMOB_Net   | True MOB net |
| TrueMOB_Gross | True MOB grosst |
| Ordinary_ExecTerm_AR   | Southern Pioneer's Executive Term ordinary life product for Arkansas |
| Ordinary_ProtectAll_AL | Protective Life's (now a part of Life of the South) Protect-All ordinary life product for Alabama |
| Ordinary_ProtectAll_GA | Protect-All ordinary life product for Georgia |
| Ordinary_ProtectAll_NC | Protect-All ordinary life product for North Carolina |
| Ordinary_ProtectAll_LA | Protect-All ordinary life product for Louisiana |
| Ordinary_ProtectAll_SC | Protect-All ordinary life product for South Carolina |
| Ordinary_Uniguard_TN   | Uniguard ordinary life product for Tennessee |
| CUNA_SP_Formula1 | CUNA Mutual's single premium formula \#1 |
| CUNA_SP_Formula2 | CUNA Mutual's single premium formula \#2 |
| CUNA_SP_Formula3 | CUNA Mutual's single premium formula \#3 |
| CUNA_SP_Formula6 | CUNA Mutual's single premium formula \#6 |

🟦 **Product.RateType**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Fixed, Variable |

This field will only be present in the output if the protection
product has been configured to allow for coverage to switch from 
joint to single during the term of coverage, should one of the borrowers
exceed an age at termination cap. If the field is not present, then
a value of `Fixed` should be assumed as only one rate has been
used in the protection calculation.

If this field is present, then there is the possibility that the
rate used to compute the protection may have changed (in the case of
coverage for one borrower ending while coverage for the other borrower
continues). If this is the case, then the field will indicate this
rate change with a value of `Variable`.

🟦 **Product.DropCode**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

If the requested protection was dropped by the SCEX for any reason, then the
value of this field will be greater than zero. In this case, no child objects of
`Product` object will be present.

A value of zero indicates that the requested product was included with the loan,
and as such, the child objects of `Product` which describe the coverage details,
should be parsed.

Please see the table below which lists all drop codes and reasons.

🟦 **Product.DropReason**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

If the requested protection product was successfully included in the loan, then
the value of this field will be `Valid Calculation` and corresponds to a
`DropCode` value of `0`.

If the requested protection was dropped by the SCE for any reason (and hence,
`DropCode` > 0, then this field will provide a brief description of why the
protection was dropped. Please see the table below which lists each drop code
and reason.

| Drop Code | Drop Reason |
| :--: | :--- |
| 0	 |  Successful Calculation |
| 1	 |  This insurance must be written with CL |
| 2	 |  No coverage on non-monthly loans |
| 3	 |  No coverage on Equal Payment Loans |
| 4	 |  No coverage on Balloon Loans |
| 5	 |  No coverage on Single Payment Loans |
| 6	 |  No coverage on Interest Only Loans |
| 7	 |  No coverage on Principal Plus Loans |
| 8	 |  No coverage on Skips/Pickups/Irregs |
| 9	 |  Borrower too old at loan inception |
| 10	| Co-Borrower too old at loan inception |
| 11	| Term exceeded cap. |
| 12	| Borrower became too old during loan |
| 13	| Co-Borrower became too old during loan |
| 14	| Loan term is less than minimum allowed |
| 15	| Computed rate was zero |
| 16	| An invalid AH Plan was passed |
| 17	| Truncation term less than the minimum |
| 18	| Benefit cap was exceeded |
| 19	| Coverage cap was exceeded |
| 20	| Windows experienced an error |
| 21	| The computed coverage is zero |
| 22	| Windows error reading the setup file |
| 23	| Equal Payment or Balloons Only |
| 24	| NOT CURRENTLY USED |
| 25	| Coverage is not allowed on this loan |
| 26	| Truncation term isn't a valid multiple |
| 27	| Keyboard truncation not allowed |
| 28	| No keyboard truncation with Gross |
| 29	| No keyboard truncation with Net |
| 30	| No decreasing life. All coverage level |
| 31	| Term too big for insurance coverage |
| 32	| Joint requested, only single allowed |
| 33	| The age is above the maximum in band |
| 34	| The term is above the maximum in band |
| 35	| Balloon is too small for Level Life |
| 36	| All Coverage allocated to decreasing. |
| 37	| No keyboard truncation with MOB |
| 38	| CUNA No Single Pay Terms > 6 Months |
| 39	| Credit Life not allowed on Annual Loans |
| 40	| Below the Minimum Insurance Term |
| 41	| Below the Minimum Loan Term |
| 42	| CUNA Formula specified is invalid |
| 43	| AH Requires use of a table of rates |
| 44	| LR Requires single rates, not tables |
| 45	| Customer not eligible for Insurance |
| 46	| The insurance code (e.g. 1=single) was not valid |
| 47	| Monthly Renewable exception |
| 48	| Borrower is currently older than the termination age |
| 49	| CoBorrower is older than the termination age |
| 50	| Maine Truncation only defined for monthly loans |
| 51	| Converting SP AHRate to MOB caused an exception |
| 52	| Non-Monthly not allowed with Ordinary Life |
| 53	| AH is not allowed with Ordinary Life |
| 54	| Probably a log calc tripped up an exception |
| 55	| Entry for Borrower Birthday is zero |
| 56	| Entry for CoBorrower Birthday is zero |
| 57	| No coverage for loans < Monthly |
| 58	| No coverage on construction loans |
| 59	| The term of coverage must equal CL Term |
| 60	| Loan Setup is a premium type not Single premium |
| 61	| No insurance allowed on ARM Loans |
| 62	| ARM Loans are only covered with MOB Net |
| 63	| Coverage is not allowed on Coborrower |
| 64	| Borrower is less than the minimum age |
| 65	| CoBorrower less than the minimum age |
| 66	| Loan To Value (** Not Implemented ** ) |
| 67	| ARM Loans only use MOB Net or MOB on Benefit |
| 68	| Credit Life does not permit CMOB rollbacks |
| 69	| Credit Life does not permit CMOB IntOnly Pmts or Construct Cov |
| 70	| This insurance must be written with AH |
| 71	| This insurance must be written with Joint AH |
| 72	| Partial Benefit and Truncation are not allowed with CUNA Level Rate (MOB Net) |
| 73	| This insurance must be written with Joint CL |
| 74	| Preceding Interest Only payments do not allow for protection |
| 75	| Coverage not allowed on Open end loans. |
| 76	| PrimaFacie rates must be expressed as $/100/Year |
| 77	| Loans with "Open End=1" must be equal payments |
| 78	| Premium is zero |
| 79	| The loan, itself, has an error and therefore reports no protection information. |
| 80	| Product not available for any loan |
| 81	| Level cannot be offered if decreasing has been removed |
| 82	| An unknown ordinary life method has been chosen |
| 83	| No Keyboard benefit allowed |
| 84	| TrueMOB protection requested, but no events were logged |
| 85	| Keyboard truncation is not allowed with gross truncated level insurance |
| 86	| Needs Loan Setup ini file |

🟦 **Product.Notes[]**

| Type  | Required |
| :---: |   :---:  |
| array of Note objects | no |

The `Notes` array may appear in the response to further clarify the product
calculation methodology. Each note is associated with a numeric code and text
return value. Please see the documentation for the Note object below.

<details>
<summary><b>Note fields</b></summary>

---

🟥 **Note.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The unique integer identifying the protection product `Note` in question.
This `Code` may be useful if the calling application would like to
override the default description or check for a given Note's `Code`. Please
see the list of all codes, notes, and descriptions below.

🟥 **Note.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

A text description of the `Note`. Please see the list of codes, notes, and descriptions below.

0. **Benefit is Minimum Payment over term of coverage.**

  When computing a loan with skipped, pickup, or irregular payments,
  there are a few different ways one can compute a benefit amount.
  This method uses the minimum non-zero payment as the benefit.

1. **Benefit is Average Payment over term of coverage (excludes zero payments).**

  Similar to above, this method uses the average of all non-zero
  payments over the term of coverage as the benefit.

2. **Benefit is the Computed Payment.**

	Similar to above, this method uses the computed payment as the benefit.

3. **Benefit is True Average Payment over term of coverage (includes zero payments).**

  Similar to above, this method uses the average of all payments over the
  term of coverage, including skips, as the benefit.

4. **Protection factor uses days per period conversion.**

	The protection calculation has converted the periodic rate to a daily rate.

5. **Switch to Rate Set two.**

	The protection calculation has switched to the second set of rates.

6. **Switch method to Net.**

  The protection calculation has switched to net coverage.

7. **Switch method to Ordinary Life.**

	The protection calculation has switched to ordinary life coverage.

8. **Benefit is Average Payment over term of coverage (excludes loan principal from final payment).**

	This average benefit calculation method is most commonly seen with interest only loans.
  It uses the average of all payments over the term of coverage (excluding the loan
  principal amount from the final payment, if covered).

---

</details>

🟦 **Product.Cost**

| Type  | Required |
| :---: |   :---:  |
| object | no |

This object's fields detail the cost of the protection product in total, per
payment, and per day. It also provides the rate used to compute the premiums.

<details>
<summary><b>Cost fields</b></summary>

---

🟥 **Cost.Premium**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The total cost for this protection over the term of the loan.

🟥 **Cost.PerPmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as an amoun per payment.

🟥 **Cost.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as an amoun per day.

🟥 **Cost.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The rate used to compute the premium for the requested protection product.

🟦 **Cost.Factor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Floating |

The rate factor used to compute the premium for the requested protection
product. Note that this field is only present if the `Product.ShowFactor` field
of the associated protection product `true`.

---

</details>

🟦 **Product.Coverage**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The aggregate coverage amount, code, and note are provided
in the fields of this object:

<details>
<summary><b>Coverage fields</b></summary>

---

🟥 **Coverage.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The aggregate coverage amount for this protection product.

🟥 **Coverage.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

If the requested protection was capped by the SCE for any reason, then
the value of this field will be greater than zero. A value of zero
indicates that the requested product was included with the loan for the
full coverage amount of the loan.

🟥 **Coverage.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field will describe the type of coverage provided. If
full coverage has been provided on the aggregate coverage, then the note will contain
`Full Coverage`. Otherwise, the note will describe the type of partial
coverage used.

---

</details>

🟦 **Product.Benefit**

| Type  | Required |
| :---: |   :---:  |
| object | no |

Disability and involuntary unemployment provide coverage based upon a computed
benefit amount. This object collects the monthly and periodic benefit amounts,
as well as the benefit coverage code and note.

<details>
<summary><b>Benefit fields</b></summary>

---

🟥 **Benefit.BenMon**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The computed monthly benefit amount.

🟥 **Benefit.BenPer**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The computed periodic benefit amount.

🟥 **Benefit.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

If the benefit for the requested protection was capped by the SCE for any
reason, then the value of this field will be greater than zero. A value of zero
indicates that the requested product was included with the loan for the full
benefit amount of the loan.

🟥 **Benefit.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field will describe the type of benefit coverage provided. If full coverage
has been provided on the benefit, then the note will contain `Full Coverage`.
Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟦 **Product.Borrowers[]**

| Type  | Required |
| :---: |   :---:  |
| array of Borrower objects | no |

This array provides the calling application with data about the term of
coverage for the specified borrower(s), for the requested product.

<details>
<summary><b>Borrower fields</b></summary>

---

🟥 **Borrower.Birthday**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The birthday associated with the borrower, as specified in the request. All
dates are in the form of `YYYY-MM-DD`, and must be 10 characters long.

🟥 **Borrower.Pmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of payments.

🟥 **Borrower.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of months.

🟦 **Borrower.AmMonths**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field is only be returned when the protection product is
configured to use CUNA Mutual's single premium formula #5, and contains
the computed amortization term used in the premium computation. 

🟥 **Borrower.AgeAtIssue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The value of this field represents the age at issue of the borrower in a special
date-like format of `YYYY-MM-DD`, where the borrower is YYYY years, MM months,
and DD days old when coverage begins.

🟥 **Borrower.AgeAtMaturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The value of this field represents the age at maturity of the borrower in a
special date-like format of `YYYY-MM-DD`, where the borrower is YYYY years, MM
months, and DD days old when coverage terminates.

🟥 **Borrower.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The coverage maturity date for this particular borrower.

🟥 **Borrower.Code**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

If the requested protection was truncated by the SCE for any reason, then the
value of this field will be greater than zero. A value of zero indicates that
the requested product was included with the loan for the full term of the loan.

🟥 **Borrower.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

This field will describe the type of coverage provided. If full term coverage
has been provided, then the note will contain `Full Coverage`. Otherwise, the
note will describe the type of truncated coverage used.

---

</details>

🟦 **Product.Caps**

| Type  | Required |
| :---: |   :---:  |
| object | no |

This object provides the calling application with data about the maximum terms,
amounts, and ages associated with the requested product.

<details>
<summary><b>Caps fields</b></summary>

---

🟦 **Caps.Coverage**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

Contains the maximum aggregate coverage amount, expressed in dollars. If no cap
is present or applicable, then this field will not be present.

🟦 **Caps.Benefit**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

Contains the maximum monthly benefit amount. If no cap is present or applicable,
then this field will not be present.

🟦 **Caps.BenPer**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

Contains the maximum periodic benefit amount. If no cap is present or
applicable, or if the payment frequency of the requested loan is monthly, then
this field will not be present.

🟦 **Caps.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

Contains the maximum coverage term, expressed in months. If no cap is present or
applicable, then this field will not be present.

🟦 **Caps.TermPer**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

Contains the maximum coverage term, expressed as a number of payments. If no cap
is present or applicable, or if the payment frequency of the requested loan is
monthly, then this field will not be present.

🟦 **Caps.InceptAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

Contains the maximum age a borrower may be at loan inception, expressed in
years. If no cap is present or applicable, then this field will not be
present.

🟦 **Caps.AttainAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

Contains the maximum age a borrower may attain during the term of the loan,
expressed in years. If no cap is present or applicable, then then this field
will not be present.

</details>

</details>

---

</details>

### 🟦 AmTable

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object contains fields which summarize and describe the loan's amortization
schedule. If `EditOutput.ShowAmTable` is set to `false`, then this object will
not be found in the response.

<details>
<summary><b>AmTable fields</b></summary>

---

🟦 **AmTable.AvgBal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field will only appear if a Canadian APR is disclosed for the computed
loan. The value of this field is the average balance of the loan used in the
Canadian APR calculation.

🟦 **AmTable.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field will only appear if a Canadian APR is disclosed for the computed
loan. The value of this field is the whole number of months in the term of the
loan used in the Canadian APR calculation. Note that the term is expressed in
monthly or weekly units, but never both.

🟦 **AmTable.Weeks**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field will only appear if a Canadian APR is disclosed for the computed
loan. The value of this field is the whole number of weeks in the term of the
loan used in the Canadian APR calculation. Note that the term is expressed in
monthly or weekly units, but never both.

🟦 **AmTable.OddDays**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field will only appear if a Canadian APR is disclosed for the computed
loan. The value of this field is the number of odd days in the term of the loan
used in the Canadian APR calculation. Odd days are computed by moving backwards
from the maturity date the number of disclosed months or weeks, and then
counting the additional number of days required to land on the loan date.

🟦 **AmTable.GrandTotals**

| Type  | Required |
| :---: |   :---:  |
| object | no |

This object describes the total amounts of various categories throughout the life of the loan.
As an example, the total amount paid to interest and principal, as well as the
total of payments are all contained in fields of this object.

<details>
<summary><b>GrandTotals fields</b></summary>

---

🟥 **GrandTotals.PmtTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The total of payments scheduled for the computed loan.

🟥 **GrandTotals.IntTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The total amount paid to interest over the life of the loan, assuming all
payments are made as scheduled.

🟥 **GrandTotals.PrinTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field contains the total amount paid to principal during the loan term,
assuming all payments are made as scheduled.

---

</details>

🟦 **AmTable.SubTotals[]**

| Type  | Required |
| :---: |   :---:  |
| array of SubTotal objects | no |

Describes the total amounts of various categories paid during a given calendar year.
For each year in which the computed loan has scheduled payments, there will be
a `SubTotal` object in the array.

<details>
<summary><b>SubTotal fields</b></summary>

---

🟥 **SubTotal.Year**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The calendar year for which the subtotal data is applicable.

🟥 **SubTotal.Start**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field defines the index of the first `AmLine` event in the `AmLines[]`
array which falls in the specified calendar year.

🟥 **SubTotal.Events**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field defines the number of amortization events which belong to this
calendar year.

🟥 **SubTotal.PmtSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the total of payments scheduled for the calendar year.

🟥 **SubTotal.IntSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Holds the total amount paid to interest over the calendar year, assuming all
payments are made as scheduled.

🟥 **SubTotal.PrinSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the total amount paid to principal during the calendar year, assuming
all payments are made as scheduled.

---

</details>

🟦 **AmTable.AmLines[]**

| Type  | Required |
| :---: |   :---:  |
| array of AmLine objects | no |

There is one `AmLine` objectt for each amortization event which occurs during
the life of a loan. Most of the time, each event will describe a payment, and
detail how that payment is applied (to interest, principal, loan protection
products, etc.). Some events, such as capitalizing interest, will not have
payments but will show how the loan amortizes.

<details>
<summary><b>AmLine fields</b></summary>

---

🟥 **AmLine.Idx**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The index of the amortization event, which is either the payment number, or
zero. A value of zero designates an event that is either not a payment or is a
skipped payment.

🟦 **AmLine.Type**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

The type of event is recorded in this field, such as `Advance` or `FixedPmt`. If
any fees are included in the loan, then the value of this field for those fee
events will be the `Name` of the fee, as specified in the request.

🟥 **AmLine.Date**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The date on which the amortization event is scheduled to occur. All dates are in
the form of `YYYY-MM-DD`, and must be 10 characters long.

🟥 **AmLine.BegBal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The principal balance before the amortization event occurs.

🟥 **AmLine.Pmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The payment amount for this event.

🟥 **AmLine.Int**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The amount of interest paid at this event.

🟥 **AmLine.Prin**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The amount of principal paid at this event.

🟦  **AmLine.FeeTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The total of all fees paid at this event.

🟦  **AmLine.ProtUnpaid**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the loan includes escrowed True MOB protection products, and if a payment is
not sufficient to pay the accrued fees, then any unpaid protection fees will be
carried forward in this field to be paid off as soon as possible in a future
payment.

🟦  **AmLine.PmtEsc**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the computed loan is an annual rest mortgage, then the sum of escrowed
payments for each amortization event will appear in this field. If the computed
loan is *not* an annual rest mortgage, then this field will not be found in the
response.

🟦  **AmLine.MI**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The amount of mortgage insurance paid at this event.

🟦  **AmLine.UnpaidInt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Floating |

Any interest not paid after this event is logged in this field.

If `EditOutput.KeepSlush` is set to `true` in the request, then the unpaid
interest will always be displayed to four (4) decimal places.

If `EditOutput.KeepSlush` is set to `false`, then if the interest accrued has
not yet been rounded, then the unpaid interest will be displayed to four (4)
decimal placed. If rounded, then the unpaid interest is displayed to two (2)
decimal places.

🟥 **AmLine.EndBal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The principal balance amount, after the amortization event has taken place.

</details>

---

</details>

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Version Module](module-version.md) | [SCEJSON Reference Manual](README.md) | [Adjustable Rate Mortgages](module-arm.md) |
