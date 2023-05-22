# Construction Loans

> "The thin line between life and death  
>  is still under construction."  
>                           --- Santosh Kalwar

Construction loans are multiple advance loans to finance construction of a
dwelling that may be permanently financed by the same creditor either as a
single transaction or as more than one transaction. This module should be used
if the creditor chooses to disclose the construction phase separately. If the
creditor chooses to disclose the construction and the permanent financing as one
transaction, please see the `Construction` field for the appropriate Loan module
which supports the permanent loan's repayment structure.

## Construction Loan Request Data Object Field Definition

**Example - Request Envelope for Construction Module**

The following example is a request for a construction loan with no permanent
loan attached, with a proceeds of $100,000, interest will accrue at a 4.5% rate
using a unit period / 360 U. S. Rule accrual method (accrual code `301`). There
will be 11 monthly interest only payments with an additional final payment
paying off the loan. As the schedule of advances are not known at this time and
interest is only due on the outstanding balance, we will assume that half the
commitment amount is outstanding during the term of the loan. Furthermore, we
will ask the SCE to return the individual interest only payments instead of
using the `Simple` method which returs a single lump sum interest amount.

```json
{
  "Module" : "Construction",
  "Data" : {
    "LoanDate" : "2022-08-24",
    "PmtDate" : "2022-10-01",
    "IntRate" : "4.500",
    "Term" : "12",
    "PPY" : "12",
    "Proceeds" : "100000.00",
    "HalfCommitment" : true,
    "Settings" : {
      "AccrualCode" : "301",
      "ConMethod" : "InterestOnly"
    }
  }
}
```

The fields of the `Data` object supported by a Construction module request are
defined in alphabetical order below:

### 🟦 Apr

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Settings related to the computed effective rate returned by the SCE (most
commonly, the Regulation Z APR in the United States of America) are contained
in the fields of this object.

<details><summary><b>APR fields</b></summary>

---

🟦 **Apr.MAPR_Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - % | 36 |

If you are computing the Military APR (see `UseMAPR` below) and wish to override
the default maximum APR value of 36%, then specify the desired maximum as the
value of this field.

🟦 **Apr.Method**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, actuarial | default |

Construction loans *only* support the actuarial APR. Please see [Appendix D to
Regulation Z](https://www.consumerfinance.gov/rules-policy/regulations/1026/d/)
for more information.

🟦 **Apr.UseMAPR**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to a value of `true`, then the SCE will compute the
Military APR in addition to the Regulation Z APR. The `MAPR` object will be
included in the loan response.

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

Specifying the `Country` will also set the default value for the `APR.Method`
and `Format.CurrencyDecimals` fields, as appropriate for the country specified.

### 🟦 Fees

| Type  | Required |
| :---: |   :---:  |
| array of Fee objects | no |

This array of `Fee` objects allows the calling application to specify one ore
more Fee objects to be included with the loan.

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
| Boolean | no | true, false | true |

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
      "Entry" : "0.115",
      "AddToPrin" : true,
      "AddToFinChg" : false
    }
  ]
}
```

🟦 **Fee.CalcType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Dollar, OnProceeds, OnPrincipal, OnAmtFin | Dollar |

This field specifies how the fee is to be computed, as described in the following table.

| Fee Calc Type | Description |
| :--- | :--- |
| Dollar        | The `Entry` field is understood as a flat dollar amount. |
| OnProceeds    | The `Entry` field is understood as a percentage value, to be applied to the loan's proceeds, defined as the sum of advances. An `Amount` of `0.25` would represent a fee of 0.25% of the total proceeds. |
| OnPrincipal   | The `Entry` field is understood as a percentage value, to be applied to the loan's principal balance. An `Amount` of `0.125` would represent a fee of 0.125% of the principal balance. |
| OnAmtFin      | The `Entry` field is understood as a percentage value, to be applied to the loan's Regulation Z Amount Financed. An `Amount` of `0.33` would represent a fee of 0.33% of the amount financed. |

🟦 **Fee.Entry**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Floating | 0 |

How this field is interpreted depends upon the `Fee.CalcType` field.
Please see the documentation for this field for further information.

🟦 **Fee.IsLoanCost***

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When requesting TILA RESPA outputs (via the `Settings.TILARESPA2015` field), the
SCE needs to know which fees need to be considered "borrower paid loan costs",
as defined in the rule. Please note that if the fee is paid by a lender or other
third party, then the fee does not affect the loan calculation and should not be
sent to the SCEX. If it is sent, then the value of this attribute should be set
to false.

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

🟦 **Format.StrictDP**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false  | false |

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

### 🟦 HalfCommitment

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

During the term of the construction loan, the estimated interest for disclosure
purposes may be either computed on the full or half commitment amounts. By
setting this field to `true`, interest will be estimated using half of the
initial commitment amount (i.e. the principal balance). The default value of
`false` will cause interest to be estimated using the entire initial commitment
amount.

### 🟥 IntRate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - %| n/a |

Determine the interest rate used for the loan. The interest rate should be
expressed as a percentage. For example, a loan computed with a rate of 5.125%
would be specified as `"IntRate" : "5.125"`.

### 🟦 IntStartDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD | same value as `LoanDate` |

This field contains the date on which interest begins to accrue on the loan's
principal balance. If this field is not specified, then the interest start
date is set equal the loan date. Also note that the interest start date
*must* be equal to the loan date when computing a construction loan.

The interest start date (when specified) must be on or after the loan date,
as well as on or before the first payment date.

### 🟥 LoanDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field holds the date on which the loan amount is disbursed and interest
begins to accrue. All dates must be in the form of YYYY-MM-DD, and be 10
characters long.

### 🟦 ODI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If odd days should be treated as a prepaid finance charge *or* added to the
first payment, then the request needs to define how odd days interest is
computed and handled using the fields of this object.

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

### 🟥 PmtDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field specifies the first (and for single payment notes, only) payment date
in the loan's repayment schedule. All dates must be in the form of YYYY-MM-DD,
and be 10 characters long.

### 🟦 PPY

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

`PPY` is an abbreviation for "payments per year", and determines the payment
frequency for the requested loan. The default value of monthly will result in a
loan with 12 payments per year. If you require a loan with a payment frequency
other than monthly, specify it using this field.

### 🟥 Proceeds

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

The proceeds specified indicate the amount of money the borrower is
requesting, and does not include financed fees, financed insurances,
etc.

### 🟦 ServiceCharges

| Type  | Required |
| :---: |   :---:  |
| array of ServiceCharge objects | no |

This array of `ServiceCharge` objects allows the calling application to specify one ore
more Service Charge objects to be included with the loan.

<details>
<summary><b>ServiceCharge fields</b></summary>

---

🟦 **ServiceCharge.CalcType**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | LumpSum, Periodic | LumpSum |

A service charge of type `LumpSum` allows the calling application to
specify an amount that will be spread out evenly over the loan's payment
stream. On the other hand, a `Periodic` service charge is the amount
which will be added to each payment.

🟥 **ServiceCharge.Entry**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Floating | n/a |

How this field is interpreted depends upon the `ServiceCharge.CalcType` field.
It is the numeric amount defining either the lump sum or periodically paid
service charge.

🟦 **ServiceCharge.Exact**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

The `Exact` field is only considered when the service charge is of type
`LumpSum`. When the calculated periodic amount is rounded, it will most often
times produce a total service charge amount that differs from what was
specified.

If the `Exact` attribute is set to `true`, then the final periodic amount will
be adjusted so that the sum of all periodic amounts is exactly equal to the
entered amount (and will result in an odd final payment). A value of `false`
indicates that the final periodic amount will not be adjusted.

🟦 **ServiceCharge.IsLoanCost***

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When requesting TILA RESPA outputs (via the `Settings.TILARESPA2015` field), the
SCE needs to know which service charges need to be considered "borrower paid loan costs",
as defined in the rule. Please note that if the fee is paid by a lender or other
third party, then the fee does not affect the loan calculation and should not be
sent to the SCEX. If it is sent, then the value of this attribute should be set
to false.

🟦 **ServiceCharge.Name**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Text | empty |

This field is for convenience purposes only, and does not affect the calculation
of the service charge in any manner. However, the value of this field *will* be
used to identify the fee in the response, and hence it is highly recommended
that you name your fees accordingly.

🟦 **ServiceCharge.Round**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | nearest, up, down | nearest |

This field is only considered when the service charge is of type
`LumpSum`. It determines how the calculated periodic amount
is rounded.

---

</details>

### 🟦 Settings

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Settings` object allows the calling application to alter the default
loan calculation options.

<details><summary><b>Settings fields</b></summary>

---

🟦 **Settings.Account**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1..9999 |

The `Account` field specifies the numeric setup file account number that will be
used to compute the requested loan. Each account is numbered from 1 to 9,999,
and each account corresponds to a set of setup file which define numerous
settings which may affect the loan calculation, such as the accrual method,
insurance methods / rates / caps, etc.

🟦 **Settings.AccrualCode**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | see table | default |

The method of interest accrual is defined by this field. A value of `default`
informs the SCE to use the interest accrual method defined in the setup file for
the specified `Account`. All other valid accrual codes are described in the
table below.

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

🟦 **Settings.ConAccrual**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, unitperiod, actual360, actual365 | default |

If the loan request is a construction loan where interest is computed using the
`Simple` method (see `Settings.ConMethod` below), then this field allows the
calling application to specify the simple accrual method used.

A value of `default` indicates that the accrual method used will be that which
is specified in the setup files for the given account. `unitperiod` computes
interest on a pure unit period basis, with odd days optionally taken into
account (see `Settings.ConUnitOddDayDivisor` field below for more information).
`actual360` uses an actual day calendar and divides the day count by 360 to
determine the appropriate interest factor. `actual365` does the same thing, but
uses a 365 divisor instead.

🟦 **Settings.ConMethod**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, simple, interestonly | default |

When computing a construction loan, there are two
supported methods for computing interest during the constrcution
period:

If the value of this field is `simple`, then the estimated construction
interets will be computed and disclosed as a lup-sum prepaid fee (see
the `Moneys.ConstructInterest` response field).  

If the value of this field is `interestonly`, then the construction and
permanent loans are combined into a single loan, with the construction (and
permanent) loan's interest being reflected in the `Moneys.Interest` field, and
both loans reflected in a single, combined amortization schedule.

A value of `default` instructs the SCE to use the construction method specified
in the setup file for the given account.

🟦 **Settings.ConUnitPeriodDivisor**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Ignore, 360, 365, VarDPY | Ignore |

When specifying a `Settings.ConAccrual` field value of `unitperiod` (see above),
this field allows the calling application to specify how odd days between the
loan and first payment dates are taken into account.

🟦 **Settings.DataPath**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
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

🟦 **Settings.Lastday**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | default |

If present, this field overrides the "Last Day" setting in the setup files,
which only applies to loans computed with an actual day accrual calendar where
the first payment date falls at the end of a month with fewer than 31 days. As
an example, if the first payment date is on April 30, should the following
payment dates be made on the 30th of each month, or on the last day of the
month?

If no value is specified for this field, then the setup file setting will be
used. If `true` is specified, then conditions triggering a last day situation
will result in payments which fall at the end of the month. A value of `false`
indicates that when dictated, subsequent payment dates will not be moved to the
last day of the month.

🟦 **Settings.PmtRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, nearest, up, down, best | default |

How are calculated payments to be rounded at and beyond this event? `best` means
to choose the payment that returns a minimal ending balance at the end of
amortization. If two payments result in the same minimal error magnitude, the
smaller payment is chosen. `default` means that the setting should be configured
as specified in the setup file.

🟦 **Settings.ShowAmTable**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

To supress the entire amortization schedule from the response, set value of this
field to `false`; otherwise, the amortization schedule will be returned with the
response.

🟦 **Settings.TILARESPA2015**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is `true`, then the SCE will include data for the
Integrated Mortgage Disclosures under the Real Estate Settlement Procedures Act
(Regulation X) and the Truth In Lending Act (Regulation Z) rule, which is
required as of August 1st, 2015. If the field is omitted or set to `false`, then
the TILA RESPA outputs will not be generated.

Note that this field is supported for equal payment loans, balloon payment
loans, single payment notes, interest only loans, construction loans, fixed
principal plus interest loans, skips, pickups and irregulars, and ARMs.

---

</details>

### 🟦 TotalDown

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | n/a |

This optional field represents the total down payment that the borrower
has applied to reduce the requested proceeds. It may consist of a cash down
payment, net trade-in value, or rebate. You only need to specify a total down
payment if the loan needs to disclose a total sale price. Typical examples of
these loan types are auto and boat loans.

### 🟥 Term

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The `Term` field indicates the the number of *payments* to be made at the
specified payment frequency (see the `PPY` field above), after which the loan is
scheduled to be paid off.

## Construction Loan Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Construction Module**

```json
{
  "Result" : 200,
  "Module" : "Construction",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Results" : {
      "Payment" : "100000.00"
    },
    "FedBox" : {
      "AmtFin" : "100000.00",
      "FinChg" : "2306.25",
      "TotPmts" : "102306.25",
      "APR" : {
        "Value" : "4.509",
        "Type" : "Actuarial"
      }
    },
    "Moneys" : {
      "Principal" : "100000.00",
      "Interest" : "2306.25"
    },
    "Accrual" : {
      "Method" : "Unit Period 360 US Rule",
      "Days1Pmt" : "38",
      "DayCount" : "Actual",
      "Maturity" : "2023-09-01"
    },
    "PmtStreams" : [
      {
        "Term" : "1",
        "Pmt" : "237.50",
        "Rate" : "4.500",
        "Begin" : "2022-10-01"
      },
      {
        "Term" : "10",
        "Pmt" : "187.50",
        "Rate" : "4.500",
        "Begin" : "2022-11-01"
      },
      {
        "Term" : "1",
        "Pmt" : "100193.75",
        "Rate" : "4.500",
        "Begin" : "2023-09-01"
      }
    ],
    "AmTable" : {
      "GrandTotals" : {
        "PmtTot" : "102306.25",
        "IntTot" : "2306.25",
        "PrinTot" : "100000.00"
      },
      "SubTotals" : [
        {
          "Year" : "2022",
          "Start" : "1",
          "Events" : "3",
          "PmtSub" : "612.50",
          "IntSub" : "612.50",
          "PrinSub" : "0.00"
        },
        {
          "Year" : "2023",
          "Start" : "4",
          "Events" : "9",
          "PmtSub" : "101693.75",
          "IntSub" : "1693.75",
          "PrinSub" : "100000.00"
        }
      ],
      "AmLines" : [
        {
          "Idx" : "1",
          "Date" : "2022-10-01",
          "BegBal" : "50000.00",
          "Pmt" : "237.50",
          "Int" : "237.50",
          "Prin" : "0.00",
          "EndBal" : "50000.00"
        },
        {
          "Idx" : "2",
          "Date" : "2022-11-01",
          "BegBal" : "50000.00",
          "Pmt" : "187.50",
          "Int" : "187.50",
          "Prin" : "0.00",
          "EndBal" : "50000.00"
        },
        ...,
        {
          "Idx" : "12",
          "Date" : "2023-09-01",
          "BegBal" : "100000.00",
          "Pmt" : "100193.75",
          "Int" : "193.75",
          "Prin" : "100000.00",
          "EndBal" : "0.00"
        }
      ]
    }
  }
}
```

The `Data` object for a response from the Construction module is defined below,
in the order the fields are returned:

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
  "Module" : "Construction",
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
  "Module" : "Construction",
  "Data" : {
    "Errors" : [
      "Data.LoanDate (StringDate) not found.",
      "Data.PmtDate (StringDate) not found.",
      "Data.Term (StringInt) not found.",
      "Data.IntRate (StringFloat) not found.",
      "Data.Proceeds (StringFloat) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}
```

### 🟦 Country

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Alpha-2 or Numeric-3 code | US |

If the request specified a two-character or three-digit `Country` code, then
this field will be present and will contain the full name of the country
associated with the specified code. Please see the [Countries
Appendix](appendix-countries.md) for the list of supported countries and their
associated codes.

### 🟦 Results

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This field(s) of this object represent the most important numerical results for
the loan request.

<details>
<summary><b>Results fields</b></summary>

---

🟥 **Results.Payment**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The final principal payment of the construction loan.

---

</details>

### 🟦 FedBox

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object groups together all fields which contain important numerical
information, as defined in the Truth-In-Lending laws (Regulation Z).

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

🟦 **FedBox.TotalSalePrice**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The sum of the total of payments plus the total down payment. Please note that
if no `TotalDown` field was included in the request, then this element will not
be present.

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

🟦 **APR.Max**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - % |

This field returns the maximum APR as configured in the account's setup files.
If no maximum APR has been specified, then this field will not be present in the
response. The value of this field should be displayed as a percentage.

🟦 **APR.MaxExceeded**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| boolean | no | true, false |

If a maximum APR is configured in the account's setup files, then the value of
this field indicates whether or not the current loan exceeds the maximum APR. If
no maximum APR has been specified, then this field will not be present in the
response.

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

### 🟦 TILARESPA2015

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object contains fields which are of interest to fulfilling the 2015 TILA
RESPA rule. It will only be present if the `Settings.TILARESPA2015` field in the
request is set to `true`.

<details>
<summary><b>TILARESPA2015 fields</b></summary>

---

🟥 **TILARESPA2015.LoanCosts**

| Type  | Required |
| :---: |   :---:  |
| array of LoanCost objects | yes |

For every object in the `Fees[]` and `ServiceCharges[]` array present in the
request which has its `IsLoanCost` field set to `true` (and hence, is a borrower
paid loan cost) and whose amount is greater than zero (except odd days interest
fee types, as explained in the previous documentation of the `Fee` and
`ServiceCharge` objects), there will be a corresponding `LoanCost` object.

<details>
<summary><b>LoanCost fields</b></summary>

---

🟦 **LoanCost.Name**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

If a name was provided for the fee, then it will be included here in the
disclosure for identification purposes.

🟦 **LoanCost.In5Years**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the entire amount of the fee is different from the amount collected over the
first five years (for example, a service charge), then this attribute will be
present and disclose the portion of this loan coast that is accrued during the
first five years.

🟥 **LoanCost.Value**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The value of this field is the numerical value of the fee, rounded to the
nearest dollar.

---

</details>

🟥 **TILARESPA2015.TotalLoanCost**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The value of this field is the sum of all borrower paid loan costs. Since all
`LoanCost` values are rounded dollar amounts, the value of this element will
also be a rounded dollar amount.

🟥 **TILARESPA2015.CD_TotPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field returns the sum of the total of payments and all borrower paid loan
costs. This value should be disclosed on the Closing Disclosure form in the
Total of Payments field.

🟥 **TILARESPA2015.In5Years**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

This object contains all important values required for the new "In 5 Years"
section of the disclosure.

<details>
<summary><b>In5Years fields</b></summary>

---

🟥 **In5Years.PaidTotal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field holds the sum of all "principal, interest, mortgage insurance, and
borrower paid loan costs scheduled to be paid through the end of the 60th month
after the due date of the first periodic payment". Note that this value is
rounded to the nearest whole dollar.

🟥 **In5Years.PaidPrincipal**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This attribute holds "the principal scheduled to be paid through the end of the
60th month after the due date of the first periodic payment". Note that this
value is rounded to the nearest whole dollar.

---

</details>

🟥 **TILARESPA2015.TIP**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The total interest percentage, rounded to three or fewer decimal places - as
required.

🟥 **TILARESPA2015.MaxPnIPmt**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The fields of this object hold the maximum sceduled principal and interest
payment during the term of the loan, as well as the date on which that payment
is made.

<details>
<summary><b>MaxPnIPmt fields</b></summary>

---

🟥 **MaxPnIPmt.Date**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The value of this field returns the date on which the maximum scheduled
principal and interest payment is made. If the maximum scheduled payment occurs
more than once, then the date returned is that of the first instance.

🟥 **MaxPnIPmt.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The maximum sceduled principal and interest payment during the term of the loan.

---

</details>

🟥 **TILARESPA2015.MinRate**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The fields of this object hold information regarding the minimum possible
interest rate applied during the term of the loan.

<details>
<summary><b>MinRate fields</b></summary>

---

🟥 **MinRate.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The value of the minimum interest rate applied during the term of the loan.

🟥 **MinRate.Idx**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field returns the payment number for which the minimum rate is
first applicable.

---

</details>

🟥 **TILARESPA2015.MaxRate**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The fields of this object hold information regarding the maximum possible
interest rate applied during the term of the loan.

<details>
<summary><b>MaxRate fields</b></summary>

---

🟥 **MaxRate.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The value of the maximum interest rate applied during the term of the loan.

🟥 **MaxRate.Idx**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field returns the payment number for which the maximum rate is
first applicable.

---

</details>

🟥 **TILARESPA2015.ProjectedPaymentsTable**

| Type  | Required |
| :---: |   :---:  |
| array of PPCols | yes |

This field returns an array of projected payment table columns (`PPCol`), with
each array member detailing a single column. Per the regulation, there will be a
minimum of one column and a maximum of four columns, depending upon the
parameters of the loan.

<details>
<summary><b>PPCol fields</b></summary>

---

🟥 **PPCol.Num**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The value of this field identifies the number of the column to which the
following fields apply. The value will be from 1 to 4.

🟥 **PPCol.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

The value of this field is the title for the column. Most of the time, it
will be in the form of "Years X - Y", or "Year X", or "Final Payment" in
the case of a final balloon payment.

🟥 **PPCol.YearStart**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The beginning year number for which this column data applies.

🟥 **PPCol.YearEnd**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The ending year number for which this column data applies.

🟥 **PPCol.PnIPmtMin**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The minimum principal and interest payment for this column.

🟥 **PPCol.PnIPmtMax**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The maximum principal and interest payment for this column.

🟥 **PPCol.IntOnly**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | none, some, all |

If none of the payments associated with this column are interest only payments,
then the value of this field will be `none`. A value of `some` means that some
of the payments (but not all) associated with this columnt are interest only.
Finally, a value of `all` indicates that *all* payments associated with this
columnt are interest only.

Note that for the purposes of this attribute, a scheduled payment is considered
an interest only payment if the payment amount pays off all interest due at the
time of the payment, with no reduction in the principal balance.

🟥 **PPCol.Balloon**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If any of the payments associated with this column are balloon payments (e.g.
isolated payments that are more than twice the value of a regular periodic
payment), then the value of this attribute will be `true`.

🟥 **PPCol.MIPmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The value of this field holds the mortgage insurance premium associated with
this column, rounded to the nearest dollar. If no mortgage insurance is present
or coverage has been dropped, a value of zero will be present.

🟥 **PPCol.TotalPmtMin**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field returns the minimum estimated total payment for this column. Note
that this value does not include any estimated escrow, as the SCEX does not
support escrow calculations. The calling application will need to increase these
values by the estimated escrow amounts if any are present.

🟥 **PPCol.TotalPmtMax**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

This field returns the maximum estimated total payment for this column. Note
that this value does not include any estimated escrow, as the SCEX does not
support escrow calculations. The calling application will need to increase these
values by the estimated escrow amounts if any are present.

</details>

---

</details>

### 🟦 Moneys

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This element groups together those other major cash result amounts not disclosed
under the `FedBox` object, such as the principal balance, interest charge, and
fee amounts.

<details>
<summary><b>Moneys fields</b></summary>

---

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

🟦 **Moneys.Fees[]**

| Type  | Required |
| :---: |   :---:  |
| array of Fee objects | no |

If the requested loan included fees, then for each fee in the loan there will be
a Fee object in this array containing the name of the fee and the computed fee
amount.

If there were no fees requested with the loan, then the `Moneys.Fees[]` array
will not be included in the response.

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

---

</details>

🟦 **Moneys.ServiceCharges[]**

| Type  | Required |
| :---: |   :---:  |
| array of ServiceCharge objects | no |

If the requested loan included service charges, then for each service charge in
the loan there will be a ServiceCharge object in this array containing the name
of the service charge and the computed service charge amount.

If there were no service charges requested with the loan, then the
`Moneys.ServiceCharges[]` array will not be included in the response.

<details>
<summary><b>ServiceCharge fields</b></summary>

---

🟦 **ServiceCharge.Name**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Text |

If a name was provided for the service charge, then it will be included here in
the disclosure for identification purposes.

🟥 **ServiceCharge.Fee**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the computed service charge amount.

</details>

---

</details>

### 🟦 Accrual

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object groups together interest accrual information, such
as the accrual method(s) used, days to the first payment and the
loan's maturity date.

<details>
<summary><b>Accrual fields</b></summary>

---

🟥 **Accrual.Method**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

The `Method` field contains a textual description of the interest accrual method
used to compute the loan (e.g. "Unit Period 365 Simple".

🟥 **Accrual.Days1Pmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

This field contains the number of days between the date of the first
advance and the date of first payment, computed by one of two
methods as specified in by `Accrual.DayCount` (below).

🟥 **Accrual.DayCount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | True360, Actual |

This field specifies the method used to compute the number of days from the date
of the first advance until the first payment date. `Actual` means that the
actual number of days between these two dates are used, whereas the `True360`
method use a 360 day calendar.

🟥 **Accrual.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

Holds the maturity date of the loan, which is the date on which the last
payment is scheduled. All dates are in the form of YYYY-MM-DD, and must
be 10 characters long.

---

</details>

### 🟦 PmtStreams[]

| Type  | Required |
| :---: |   :---:  |
| Object | no |

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
        "Term" : "1",
        "Pmt" : "475.00",
        "Rate" : "4.500",
        "Begin" : "2022-10-01"
      },
      {
        "Term" : "10",
        "Pmt" : "375.00",
        "Rate" : "4.500",
        "Begin" : "2022-11-01"
      },
      {
        "Term" : "1",
        "Pmt" : "100387.50",
        "Rate" : "4.500",
        "Begin" : "2023-09-01"
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

🟥 **PmtStream.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

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

---

</details>

### 🟦 AmTable

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object contains fields which summarize and describe the loan's amortization
schedule. If `Settings.ShowAmTable` is set to `false`, then this object will not
be found in the response.

<details>
<summary><b>AmTable fields</b></summary>

---

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

🟦 **GrandTotals.SCTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `SCTot` attribute will only appear on loans with service charges. It
contains the total service charge amount paid over the duration of the loan.

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

This field defines the first amortization event which falls in the specified
calendar year. To find the `AmLine` object which corresponds to this value,
match the `Idx` field of the `AmLine` object.

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

🟦 **SubTotal.SCSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `SCSub` field will only appear on loans with service charges. It contains
the total of service charges paid during the year.

---

</details>

🟥 **AmTable.AmLines[]**

| Type  | Required |
| :---: |   :---:  |
| array of AmLine objects | yes |

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

🟦  **AmLine.SC**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field contains the total service charge for this payment, and will only be
present if one or more service charges were requested with the loan.

🟦  **AmLine.UnpaidInt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field will only appear on an amortization line when interest has been accrued,
but has not yet been paid or added to the principal balance. If the interest accrued
has not yet been rounded, then the unpaid interest will be displayed to four (4)
decimal placed. If rounded, then the unpaid interest is displayed to two (2) decimal
places.

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
| [Balloon Payment Loans](module-balloon.md) | [SCEJSON Reference Manual](README.md) | [Equal Payment Loans](module-equalpmt.md) |
