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

---

🟦 **AccrualConfig.Code**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | see table | see below |

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

---

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

---

🟦 **AccrualConfig.ExtraDays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Integer | 0 |

Increase or decrease the number of days between this event and the next event by
the value of this field. e.g. `1` will be considered one more day of
interest.

---

🟦 **AccrualConfig.IntRate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | see below |

Defines the interest rate that applies at and beyond this event. If no `IntRate`
is specified, the previously defined interest rate is used. A value of zero will
be used if no previous `IntRate` has been defined.

---

🟦 **AccrualConfig.IntRound***

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | nearest, up, down | nearest |

Defines how interest is to be rounded.

---

🟦 **AccrualConfig.NewPmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the payment should change to reflect a new interest rate, set the value of
this field to `true`; otherwise, the payment will not change after this event.
If only one `AccrualConfig` object is being specified, then this field
should be omitted altogether.

---

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

---

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
| String | yes | Number | n/a |

Defines the upper bound to which the specified split rate tier will apply.

🟥  **AccrualConfig.Tier.Rate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number | n/a |

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
| String | yes | Number | n/a |

The proceeds to be advanced to the borrower is defined using this field. If the
calling application requests that the advance be computed (see the
`Advance.Compute` field below), then the value of this field will be added
to the computed advance amount, which can be useful in multiple advance loans.

---

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

---

🟥 **Advance.Date** (Date)

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field's value holds the date on which the advance is made. All dates must
be in the form of YYYY-MM-DD, and be 10 characters long. Hence, an advance date
of January 2, 2021 would be specified as `"Date" : "2021-01-02"`.

---

🟦 **Advance.ExtraDays**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Integer | 0 |

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
the `FedBox` and `Moneys` response objects.
  
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

If the value of this field is set to `true`, then payment objects will be
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
`Moneys` response object as child objects and in the amortization table. The
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

### 🟦 Fees

| Type  | Required |
| :---: |   :---:  |
| array of Fee objects | no |

This array of `Feee` objects allows the calling application to specify one ore
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

---

🟦 **Fee.AddToPrin**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this fee should be added to the principal balance (e.g. the fee is financed
along with the advance(s)), then set this field to `true`. If set to `false`,
then the fee is paid up front out of the borrower's pocket.

---

🟦 **Fee.Adjust**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

The optional `Adjust` field allows the calling application
to increase or decrease the base amount on which a fee is calculated.
If a negative value is specified and this adjustment would produce a
negative base amount, then a value of zero is returned for the given fee.
This field has no effect on fees with a `CalcType` of `Dollar`.

As an example, in Tennessee you may need to define a financed, non-APR
affecting fee to be computed by decreasing the amount financed by $2,000,
and then multiplying this reduced amount by 0.115. The way to accomplish
this in the SCEX is as follows:


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

---

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

---

🟥 **Fee.Amount**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number | 0 |

How this field is interpreted depends upon the `Fee.CalcType` field.
Please see the documentation for this field for further information.

---

🟦 **Fee.Blended** (true, false, 1, 0) `false' \{optional\}}

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

The `Blended` field determines how the SCE will include the fee with a payment
or advance.

A value of `true` in an advance tells the SCEX that the amount of the advance
already includes the fee. A value of `false` means to add the fee on top of the
existing advance.

Similarly, if the Fee occurs on a payment date, then `true` means that the
payment has the fee already included in it, whereas a value of `false` means the
fee is to be added on top of the payment.

---

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

---

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
instructs the SCEX to pay the fee on the MM payment of the payment stream. For
example, `0000-06-00` instructs the system to pay this fee on June Payments.

If the `Date` field is not specified, then it will default to the last specified
Advance `Date`.

---

🟦 **Fee.EqualServChg**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When a fee is set up as a service charge of type `ByTerm` (see `Fee.ServChgType`
field), this field determines if all the service charge fee payments should be
forced to be equal (a value of `true`), or if the final service charge fee
payment can be different from the others due to rounding (a value of `false`).

---

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

---

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

---

🟦 **Fee.Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

If a maximum value for the fee is specified and is greater than zero, then if
the computed fee exceeds this maximum value, then the maximum value will be used
instead. If no maximum value is specified or is set to a value of zero, then no
maximum will be enforced. Please note that this field is applied to *all* fee
types supported. Also, note that a specified maximum value is checked *after*
enforcing a specified minimum value, and hence a specified maximum value trumps
a specified minimum.

---

🟦 **Fee.Min**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

If a minimum value for the fee is specified and is greater than zero, then if
the computed fee is less than this minimum value, then the minimum value will be
used instead. If no minimum value is specified or is set to a value of zero,
then no minimum will be enforced. Please note that this field is applied to
*all* fee types supported. Also, note that a specified minimum value is checked
*before* enforcing a specified maximum value, and hence a specified maximum
value trumps a specified minimum.

---

🟦 **Fee.Name**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | any | empty |

This field is for convenience purposes only, and does not affect the calculation
of the fee in any manner. However, the value of this field *will* be used to
identify the fee in the response, and hence it is highly recommended that you
name your fees accordingly.

---

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

---

🟦 **Fee.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and in the case of the Field
object, determines the frequency for the fee stream. If the value of the `Term`
field is `1`, then the value of this field is ignored.

---

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

---

🟦 **Fee.RoundBasis**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0.01 |

When rounding the amount on which the fee is based, the value of this field
determines the precision to which rounding is performed. The default value of
`0.01` indicates that the rounding should be done to the penny, whereas a value
of `100.0` indicates that rounding should take place to the nearest hundred
dollars.

---

🟦 **Fee.SemimonthlyDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0...31 | 0 |

When specifying a semimonthly fee stream, the day number on which the first fee
is added determines the day number for all of the following odd numbered
payments. If you omit this field or specify a value of `0`, then the even
numbered fees will be generated using the default method within the SCEX. If the
value of the `Term` field is `1`, then the value of this field is
ignored.

If you wish to specify the day number on which even numbered fees fall
(overriding the default method used in the SCEX), then set the value of this
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

---

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

---

🟦 **Fee.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Integer > 1 | 1 |

The `Term` field determines the the number of fees to be included at the
specified frequency (see `Fee.PPY` above), after which the fee stream is
completed. The default value is `1`. If the value of the `Date` field is not a
valid date, then the value of this field is ignored.

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

---

🟦 **ODI.AddToPmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the calling application wants the odd days interest to be added to the first
payment, then set the value of this field to `true`. A value of `false`
indicates that the odd days interest will be treated as a prepaid finance
charge.

---

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

---

🟦 **ODI.ForceUnitPeriod**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Some unit period methods will not use a strict unit period interest accrual
factor in the period to the first payment. For example, code `302` will count
the days to the first payment and divide by 365. For a monthly loan, setting
this field to `true` will use a 1/12 factor instead of Days/365.

---

🟦 **ODI.UseDailyCost**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the total odd days prepaid fee is computed by first generating a rounded (to
the nearest penny) daily cost and then multiplying this value by the computed
number of odd days, then set the value of this property to `true`.

A value of `false` means that the daily cost is left unrounded, and the total
prepaid fee is rounded after the computation is complete.

---

🟦 **ODI.UseNegODI**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If there are negative odd days in the loan, then the value of this field
determines if a negative odd days interest fee is computed. If the value of this
field is `false`, then negative odd days fees are not allowed, the SCEX will
return a value of zero in this situation, and the computed payment will be
adjusted to take into account the negative odd days. A value of `true` will
return a negative odd days interest fee (in effect, it then becomes and odd days
interest credit) when there are negative odd days in a loan.

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
