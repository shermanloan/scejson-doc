# Interest Only Loans

> "An election is coming. Universal peace is declared  
>  and the foxes have a sincere interest in  
>  prolonging the lives of the poultry."  
>                           --- T. S. Eliot

Interest only loans are loans which have a payment stream consisting of
one or more interest only payment, with a final payment paying off the
principal balance plus the interest accrued during the final payment
period. The principal balance does not decrease during the term of the
loan, until the final payment when it is scheduled to be fully paid.

## Interest Only Loan Request Data Object Field Definition

**Example - Request Envelope for InterestOnly Module**

The following example is a request for an interest only loan with proceeds of
$10,000, interest will accrue at a 4.5% rate using a unit period / 360 U. S.
Rule accrual method (accrual code `301`), with a financed and APR affecting document
preparation fee of $25 included. There will be 11 monthly interest only payments
with an additional final payment paying off the loan.

```json
{
  "Module" : "InterestOnly",
  "Data" : {
    "LoanDate" : "2022-08-22",
    "PmtDate" : "2022-10-01",
    "IntRate" : "4.500",
    "Term" : "12",
    "PPY" : "12",
    "Proceeds" : "10000.00",
    "Fees" : [
      {
        "Name" : "Doc Prep Fee",
        "AddToFinChg" : true,
        "AddToPrin" : true,
        "CalcType" : "Dollar",
        "Entry" : "25.00"
      }
    ],
    "Settings" : {
      "AccrualCode" : "301"
    }
  }
}
```

The fields of the `Data` object supported by an InterestOnly module request are
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
| String | no | default, actuarial, eu, ca, xirr, xirr360, irr | default |

This field allows the calling application to specify an APR calculation
method which will override the default value found in the setup file for the given
`Account`. If this field is set to a valid value, then the specified method
will be used to compute the APR for this loan calculation.

If the `Country` field has been set by the calling application and
this field is not set, then the APR method used will be determined by the
specified country. For the United States of America, the default APR method is
`actuarial`. For a country in the European Union, the default value is
`eu` (European Union APR). For Canada, the default APR method is
`ca` (Canadian APR).

The `xirr` value implements the internal rate of return method as
implemented in Microsoft\textregistered Excel via the `XIRR()` function,
and is based on an actual day calendar with a 365 day divisor.

The `xirr360` value implements the internal rate of return method based
upon a unit period calendar with a 360 day divisor.

The `irr` value implements a common spreadsheet internal rate of return
method which assumes strict regular periods. Deviating from a strict regular
periodicity may produce unreliable results.

If this field is not specified and the `Country` field is not
specified, then the value of `default` will be used which informs the
SCE to use the APR method defined in the setup file for the specified
`Account`.

🟦 **Apr.UseMAPR**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this field is set to a value of `true`, then the SCE will compute the
Military APR in addition to the Regulation Z APR. The `MAPR` object will be
included in the loan response.

---

</details>

### 🟦 Construction

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Construction` object determines if this loan is a construction to permanent
loan request. If you wish to compute a construction loan without an attached
permanent loan, you will need to use either the
[Construction](module-construction.md) or [Loan](module-loan.md) modules.

<details><summary><b>Construction fields</b></summary>

---

🟦 **Construction.HalfCommitment**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

During the term of the construction loan, the estimated interest for disclosure
purposes may be either computed on the full or half commitment amounts. By
setting this field to `true`, interest will be estimated using half of the
initial commitment amount (i.e. the principal balance). The default value of
`false` will cause interest to be estimated using the entire initial commitment
amount.

🟦 **Construction.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

The PPY field allows the calling application to specify the payment frequency
during the construction period. The default value of `12` will result in a
construction loan with 12 payments per year. If you require a payment frequency
during the construction period other than monthly, then specify it using this
field. Note that the permanent loan's payment frequency *may* differ from the
construction period's payment frequency.

🟥 **Construction.Rate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - % | n/a |

This attribute determines the rate applied to the appropriate commitment
amount during the term of the construction loan.

🟥 **Construction.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Integer | n/a |

The term of the construction loan (in payments) is specified using this field.
Please note that the term may not exceed five years.

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

### 🟥 IntRate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - %| n/a |

Determine the interest rate used for the loan.
The interest rate should be expressed as a percentage. For example,
a loan computed with a rate of 5.125% would be specified as
`"IntRate" : "5.125"`.

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

### 🟦 MI

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `MI` objectt determines if this loan includes one of the supported types of
mortgage insurance (MI): PMI or FHA.

<details>
<summary><b>MI fields</b></summary>

---

🟦 **MI.CashDown**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

The `CashDown` field represents a cash down payment made at closing. If
specified and greater than zero, this amount will be deducted from the
principal balance. If not specified, the cash down payment will default to zero.

🟦 **MI.LoanAmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Currency | 0 |

The `LoanAmt` field represents the amount by which the PMI rates
are multiplied to produce a level PMI premium. If not specified, then
the principal balance will be used to compute the annual premium.

🟦 **MI.Periodic**

| Type  | Required |
| :---: |   :---:  |
| object  | no |

The `Periodic` object supports fields which control when mortgage insurance
should be dropped during the repayment period.

<details>
<summary><b>Periodic fields</b></summary>

---

🟦 **Periodic.DropLTV**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

The value of this field determines the loan to value ratio at which MI should be
removed, and is expressed as a percentage.

🟦 **Periodic.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

The value of this field represents the term beyond which MI is no longer
included.

🟦 **Periodic.WarnLTV**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

The value of this field determines the loan to value ratio at which a warning
should be issued, and is expressed as a percentage of LTV (loan to value).

---

</details>

🟥 **MI.PropertyValue**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number - Currency | n/a |

This field's value represents the appraised property value, and will be
used in the calculation of the loan to value ratio.

🟥 **MI.Rates**

| Type  | Required |
| :---: |   :---:  |
| array of Rate  | yes |

This array of `Rate` objects allows the calling application to specify one ore
more mortgage insurance rate objects.

<details>
<summary><b>Rate fields</b></summary>

---

🟥 **Rate.Rate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Number | n/a |

The value of this field specifies the cost of mortgage insurance per $100 of the
loan amount. Note that there may be more than one Rate object in a loan request
(see the `Switch` field below).

🟦 **Rate.Switch**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

This optional field defines the payment number beyond which the associated
mortgage insurance rate will apply. If not specified, the value will default to
zero.

Thus, if there is only a single mortgage insurance rate, one may omit this
attribute.

---

</details>

🟦 **MI.Type**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | fha, pmi | pmi |

This field determines the type of mortgage insurance to include with the
loan. If the `Type` field is set to `FHA`, then a different
MI premium is computed every twelve months based upon the average outstanding principal
balance during that same term. The MI calculation adheres strictly to the [HUD
regulation](http://portal.hud.gov/hudportal/HUD?src=/program_offices/housing/comp/premiums/sfpcalc).

If the `Type` attribute is set to `PMI`, then each defined MI
rate produces a level MI premium based upon the inital loan amount.

🟦 **MI.UpFront**

| Type  | Required |
| :---: |   :---:  |
| object  | no |

This object determines the up front fee for mortgage insurance, as disclosed in the
`PMI_Fee` element in the results. If this object is not specified, no up
front fee will be computed. Note that ZOMP (zero option monthly premium) products do
not have an up front fee, which means that this element can be omitted or set to zero.

<details>
<summary><b>UpFront fields</b></summary>

---

🟦 **UpFront.Paid**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Financed, AtClosing, ByLender | AtClosing |

If the value of this field is set to `Financed`, then the computed up front fee
will be added to the principal balance and the finance charge. A value of
`AtClosing` will cause the value of the fee to be added to the finance charge
alone. Finally, a value of `ByLender` means that the up front fee is to be paid
by the lender, hence the value of the fee is not added to either the principal
balance nor the finance charge.

🟦 **UpFront.Units**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Months, Years, Points | Months |

If the `Units` field is set to `Months`, then `UpFront.Value` represents the
number of periodic MI premiums to be paid at closing.

If the `Units` attribute is set to `Years`, then `UpFront.Value` represents the
number of annual MI premiums to be paid at closing. Many single premium products
define the up front fee as a number of years of MI to be paid up front.

Finally, if the `Units` attribute is set to `Points`, then `UpFront.Value`
represents the percentage of principal to be paid up front. As of October 3,
2011, FHA loans use points.

🟦 **UpFront.Value**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number | 0 |

The `Value` attribute meaning depends upon the value of `UpFront.Units`. Please
see its documentation above for further information.

---

</details>

---

</details>

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

🟦 **ODI.NoCap**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

When the odd days interest is financed, setting this field to `true` prevents
interest from being computed on interest. The default is `false`, which means
interest will be computed on interest.

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

### 🟦 Protection

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Protection` object is used by the calling application to request one or
more payment protection products (e.g. life, disability, or involuntary
unemployment) be included in the loan calculation. Each of
these products is requested by including its own child object (`Life`,
`Disability`, and `Unemployment`), along with the borrower
birthdays.

<details>
<summary><b>Protection fields</b></summary>

---

🟦 **Protection.Birthday1**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD | n/a |

This field holds the date of birth for the secondary borrower. All dates must be
in the form of YYYY-MM-DD, and be 10 characters long. Hence, a birthday of April
9, 1972 would be specified as `"Birthday2" : "1972-04-09"`. Note that this
element *must* be set if the `Covers` attribute of any of the four payment
protection objects is set to `borrower2` or `both`.

🟦 **Protection.Birthday2**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | YYYY-MM-DD | n/a |

This field holds the date of birth for the primary borrower. All dates must be
in the form of YYYY-MM-DD, and be 10 characters long. Hence, a birthday of April
9, 1972 would be specified as `"Birthday1" : "1972-04-09"`. Note that this
element *must* be set if the `Covers` attribute of any of the four payment
protection objects is set to `borrower1` or `both`.

🟦 **Protection.Disability**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Disability` object determines what type of disability coverage is
requested for the loan.

<details>
<summary><b>Disability fields</b></summary>

---

🟦 **Disability.Benefit**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Currency | 0 |

If you wish to specify a benefit amount less than the maximum allowed, then do
so with this field. Omitting this field will ensure that the maximum benefit
amount allowed will be used in the loan calculation. Note that if the specified
account has not been set up to allow for user-specified benefit amounts for the
product in question, then this attribute will be ignored.

🟦 **Disability.Covers**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | none, borrower1, both, borrower2 | borrower1 |

The value of this field determines what type of coverage is being requested on
the loan. The values `borrower1` and `borrower2` represent single coverage on
the appropriate borrower (whose birthdays are contained in appropriate Birthday
objects described above). A request for joint coverage on both borrowers is
indicated by a value of `both`. Finally, if no coverage is desired, simple omit
the `Disability` object altogether or specify a value of `none`.

🟦 **Disability.CovTerm**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 0 |

If you need to specify a coverage term (in months or payments) less than the
maximum allowed, then do so using this field. If this field is omitted, then the
loan will be covered for the maximum term allowed. Note that if the specified
account has not been set up to allow for user specified coverage terms for this
product, then this field will be ignored.

🟦 **Disability.Table**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | 0...19 | 0 |

If the specified account has been set up with multiple disability
or debt cancellation plans, then this field determines which
plan number will be used. If no table number is specified, the
first table (table zero) will be used.

---

</details>

🟦 **Protection.Financed**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

If the computed premiums for single premium debt protection products should be
financed along with the proceeds, then this field should be set to `true` (which
is the default value if not specified). A value of `false` indicates that the
computed premiums will not be financed with the proceeds, and hence will be paid
out of pocket by the borrower. *Note that this applies to single premium
insurance products only!*

🟦 **Protection.Life**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Life` onject determines what type of life coverage is
requested for the loan.

<details>
<summary><b>Life fields</b></summary>

---

🟦 **Life.CovAmount**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Currency | 0 |

If you wish to specify a coverage amount less than the maximum allowed, then do
so with this field. Omitting this field will ensure that the maximum coverage
amount allowed will be used in the loan calculation. Note that if the specified
account has not been set up to allow for user specified coverage amounts for the
product in question, then this attribute will be ignored.

🟦 **Life.Covers**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | none, borrower1, both, borrower2 | borrower1 |

The value of this field determines what type of coverage is being requested on
the loan. The values `borrower1` and `borrower2` represent single coverage on
the appropriate borrower (whose birthdays are contained in appropriate Birthday
objects described above). A request for joint coverage on both borrowers is
indicated by a value of `both`. Finally, if no coverage is desired, simple omit
the `Life` object altogether or specify a value of `none`.

🟦 **Life.CovTerm**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 0 |

If you need to specify a coverage term (in months or payments) less than the
maximum allowed, then do so using this field. If this field is omitted, then the
loan will be covered for the maximum term allowed. Note that if the specified
account has not been set up to allow for user specified coverage terms for this
product, then this field will be ignored.

🟦 **Life.Dismemberment**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the account specified has been set up to offer life protection products with
optional dismemberment coverage, and if the optional dismemberment coverage is
desired, then set this field's value to `true`. Otherwise, use the default value
of `false`.

🟦 **Life.Method**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Default, Gross, Net | Default |

Some accounts are configured to offer different types of credit life products
(usually gross and net). In these accounts, this field allows the calling
application to specify which method to use for a given loan. If no method is
present in the request, then the default method will be used.

🟦 **Life.UseLevelRates**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| Boolean | no | true, false | false |

If the account specified has been set up to offer single premium credit life
using level rates instead of the normal decreasing rates, and if you wish to use
level rates instead of decreasing, then set this field's value to `true`.
Otherwise, use the default value of `false`.

---

</details>

🟦 **Protection.LineOfCredit**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If this is an account using TruStage's MOB insurance, and if this loan is a
line of credit where product term caps should be ignored, then set this field to
`true`. Otherwise, omit this field or set it to `false`.

🟦 **Protection.Mandatory**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the value of this field is set to `true`, then any computed payment
protection premium or fee will be considered a mandatory fee which will be
included as a part of the Finance Charge, and hence affect the disclosed APR.

If the `Mandatory` field is set to `false`, then the loan will treat any
premiums / fees as normal.

🟦 **Protection.ShowBorrowerInfo**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the calling application would like to have data returned for each specified
borrower instead of a single `Term` object, then set the value of this field to
`true`. Otherwise, omit this field or set it to `false`. See the `Borrower` and
`Term` objects defined in the response section for more information.

🟦 **Protection.ShowCaps**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If the calling application would like to have cap information (e.g. maximum
terms, coverage amounts, ages, etc.) returned for each product offered, set the
value of this field to `true`. Otherwise, omit this field or set it to `false`.
See the `Caps` objectt defined in the response section for more information.

🟦 **Protection.Unemployment**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The `Unemployment` object determines what type of unemployment coverage is
requested for the loan.

<details>
<summary><b>Unemployment fields</b></summary>

---

🟦 **Unemployment.Benefit**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Currency | 0 |

If you wish to specify a benefit amount less than the maximum allowed, then do
so with this field. Omitting this field will ensure that the maximum benefit
amount allowed will be used in the loan calculation. Note that if the specified
account has not been set up to allow for user-specified benefit amounts for the
product in question, then this attribute will be ignored.

🟦 **Unemployment.Covers**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | none, borrower1, both, borrower2 | borrower1 |

The value of this field determines what type of coverage is being requested on
the loan. The values `borrower1` and `borrower2` represent single coverage on
the appropriate borrower (whose birthdays are contained in appropriate Birthday
objects described above). A request for joint coverage on both borrowers is
indicated by a value of `both`. Finally, if no coverage is desired, simple omit
the `Unemployment` object altogether or specify a value of `none`.

🟦 **Unemployment.CovTerm**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   | :---: |
| String | no | Number - Integer | 0 |

If you need to specify a coverage term (in months or payments) less than the
maximum allowed, then do so using this field. If this field is omitted, then the
loan will be covered for the maximum term allowed. Note that if the specified
account has not been set up to allow for user specified coverage terms for this
product, then this field will be ignored.

</details>

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

🟦 **Settings.ConMethod**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | default, simple, interestonly | default |

When computing a construction loan, there are two supported methods for
computing interest during the constrcution period:

If the value of this field is `simple`, then the estimated construction
interets will be computed and disclosed as a lup-sum prepaid fee (see
the `Moneys.ConstructInterest` response field).  

If the value of this field is `interestonly`, then the construction and
permanent loans are combined into a single loan, with the construction (and
permanent) loan's interest being reflected in the `Moneys.Interest` field, and
both loans reflected in a single, combined amortization schedule.

A value of `default` instructs the SCE to use the construction method specified
in the setup file for the given account.

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

🟦 **Settings.PmtDollarRound**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Payments are normally rounded to the penny, according to the method specified by
in the setup file, or vy the`Settings.PmtRound` field. If the payment should be
rounded to the dollar instead of the penny, then set the value of this field to
`true`.

Note that for some loans (such as those with longer terms or relatively small
proceeds), rounding the payment up or to the nearest dollar may require a
shortened loan term to prevent one or more negative payments at the end of the
loan.

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

🟦 **Settings.YieldPPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0, 2, 4, 6, 12 | 0 |

Canadian mortgages may compute periodic interest using a fractional power of a
periodic yield. If set to a value other than `0', this field determines the
period. Please contact us for further information if you support mortgage
calculations in Canada. Note that when using this attribute with a value other
than zero, the calling application *must* include an odd days prepaid fee in the
request.

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

## Interest Only Loan Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for InterestOnly Module**

```json
{
  "Result" : 200,
  "Module" : "InterestOnly",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Results" : {
      "Final" : "10062.59",
      "First" : "50.13"
    },
    "FedBox" : {
      "AmtFin" : "10000.00",
      "FinChg" : "488.62",
      "TotPmts" : "10488.62",
      "APR" : {
        "Value" : "4.748",
        "Type" : "Actuarial"
      }
    },
    "Moneys" : {
      "Principal" : "10025.00",
      "Interest" : "463.62",
      "FinFees" : "25.00",
      "FinChgFees" : "25.00",
      "Fees" : [
        {
          "Name" : "Doc Prep Fee",
          "Fee" : "25.00"
        }
      ]
    },
    "Accrual" : {
      "Method" : "Unit Period 360 US Rule",
      "Days1Pmt" : "40",
      "DayCount" : "Actual",
      "Maturity" : "2023-09-01"
    },
    "PmtStreams" : [
      {
        "Term" : "1",
        "Pmt" : "50.13",
        "Rate" : "4.500",
        "Begin" : "2022-10-01"
      },
      {
        "Term" : "10",
        "Pmt" : "37.59",
        "Rate" : "4.500",
        "Begin" : "2022-11-01"
      },
      {
        "Term" : "1",
        "Pmt" : "10062.59",
        "Rate" : "4.500",
        "Begin" : "2023-09-01"
      }
    ],
    "AmTable" : {
      "GrandTotals" : {
        "PmtTot" : "10488.62",
        "IntTot" : "463.62",
        "PrinTot" : "10025.00"
      },
      "SubTotals" : [
        {
          "Year" : "2022",
          "Start" : "1",
          "Events" : "3",
          "PmtSub" : "125.31",
          "IntSub" : "125.31",
          "PrinSub" : "0.00"
        },
        {
          "Year" : "2023",
          "Start" : "4",
          "Events" : "9",
          "PmtSub" : "10363.31",
          "IntSub" : "338.31",
          "PrinSub" : "10025.00"
        }
      ],
      "AmLines" : [
        {
          "Idx" : "1",
          "Date" : "2022-10-01",
          "BegBal" : "10025.00",
          "Pmt" : "50.13",
          "Int" : "50.13",
          "Prin" : "0.00",
          "EndBal" : "10025.00"
        },
        {
          "Idx" : "2",
          "Date" : "2022-11-01",
          "BegBal" : "10025.00",
          "Pmt" : "37.59",
          "Int" : "37.59",
          "Prin" : "0.00",
          "EndBal" : "10025.00"
        },
        ...,
        {
          "Idx" : "12",
          "Date" : "2023-09-01",
          "BegBal" : "10025.00",
          "Pmt" : "10062.59",
          "Int" : "37.59",
          "Prin" : "10025.00",
          "EndBal" : "0.00"
        }
      ]
    }
  }
}
```

The `Data` object for a response from the InterestOnly module is defined below, in the
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
  "Module" : "InterestOnly",
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
  "Module" : "InterestOnly",
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

🟥 **Results.Final**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The computed final payment of the interest only loan.

🟥 **Results.First**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The computed first payment of the interest only loan.

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

For every object in the `Fees[]` array present in the request which has its
`IsLoanCost` field set to `true` (and hence, is a borrower paid loan cost) and
whose amount is greater than zero (except odd days interest fee types, as
explained in the previous documentation of the `Fee` object), there will be a
corresponding `LoanCost` object.

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

This field returns the sum of the total of payments, all borrower paid loan
costs, and any odd days interest that is prepaid at loan closing. This value
should be disclosed on the Closing Disclosure form in the Total of Payments
field.

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

🟦 **Moneys.ConstructInterest**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If the requested loan is a construction loan with a permanent loan attached and
the request is configured to use the "Simple" method, then this field will
contain the construction interest for the requested loan.

Note that if a permanent loan is attached to a construction loan and the
request is set up to use the "Interest Only" method, then the construction
and permanent loans are combined into a single loan, with the construction
(and permanent) loan's interest being reflected in the
`Moneys.Interest` field, and both loans reflected in a single,
combined amortization schedule.

If the requested loan was not a construction loan, or if construction
loans have not been configured for the given account, then this field
will not appear in the response.

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

🟦 **Moneys.Protection**

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object returns summary information on all payment protection products
computed with the loan. If no protection products were included with the loan,
then this object will be omitted from the response.

<details>
<summary><b>Protection fields</b></summary>

---

🟥 **Protection.Cost**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Discloses the total cost of all protection plans included with the
computed loan. For the individual plan costs, see the
`Protection` field described below.

🟥 **Protection.PerPmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of all loan protection products expressed as dollars per payment.

🟥 **Protection.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of all loan protection products expressed as dollars per day.

🟥 **Protection.Category**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | None, SP, MOB, TrueMOB, PaidSP |

Te value of this field specifies the category under which the computed loan
protection falls. The categories are described below:

| Category | Description |
| :---     | :--- |
| None        | No loan protection products were computed with this loan. |
| SP          | Financed single premium coverage. |
| MOB         | Monthly outstanding balance coverage. |
| TrueMOB     | TruStage's monthly outstanding balance method. |
| PaidSP      | Non-financed single premium coverage. |

🟦 **Protection.IsDP**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | no | true, false |

This field indicates if the specified account's protection is set up as debt
protection. If this attribute is not present, then the default value of `false`
should be used (which indicates that the account's protection is set up as
insurance instead).

🟦 **Protection.Mandatory**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | no | true, false |

This field should *only* appear in the response if the value of the is `true`.
If this field does not appear in the output, then the value should default to
`false`.

If the value of the `Mandatory` field is `true`, then the total premium / fee
amount for all insurance / debt protection products has been treated as a part
of the Finance Charge, and hence will affect the effective APR.

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

🟦 **Moneys.PMI_Fee**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If PMI has been requested on the loan, and if a number of `UpFront`
payments have been specified, then this field will return the total PMI
fee for all up front payments.

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

### 🟦 PMI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The PMI object will only appear if PMI has been computed with the loan. Please
note that the PMI premiums are do not reflected in the amount reported in the
`Payment` object, the `PmtStreams[]` array, the `TotPmts` object, nor the `Pmt`
attribute of the `AmLine` object.

<details>
<summary><b>PMI fields</b></summary>

---

🟥 **PMI.Rate**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The percentage rate used in the PMI calculation.

🟥 **PMI.LTV**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - % |

The loan to value ratio of the computed loan, expressed as a percentage.

🟥 **PMI.PremiumPerYear**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The annual PMI premium amount.

🟥 **PMI.PremiumPerPeriod**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The periodic PMI premium amount.

🟦 **PMI.IndexToWarn**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field only appears if the `PercentToWarn` PMI input field
is specified, and indicates that the payment index on which the remaining
principal balance to home value ratio drops below the specified percentage.

🟦 **PMI.IndexToRemove**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Integer |

This field only appears if the `PercentToRemove` PMI input field
is specified, and indicates that the payment index on which the remaining
principal balance to home value ratio drops below the specified percentage.
PMI coverage *stops* after this payment.

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
        "Pmt" : "50.13",
        "Rate" : "4.500",
        "Begin" : "2022-10-01"
      },
      {
        "Term" : "10",
        "Pmt" : "37.59",
        "Rate" : "4.500",
        "Begin" : "2022-11-01"
      },
      {
        "Term" : "1",
        "Pmt" : "10062.59",
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

### 🟦 Protection

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If protection products are requested, then this object will be present in the
response, along with a field for each requested protection product.

<details>
<summary><b>Protection fields</b></summary>

---

🟦 **Protection.Life**

| Type  | Required |
| :---: |   :---:  |
| object | no |

If life protection was requested with the loan and decreasing life was configured for this
loan type, then the `Life` object will be present in the response to return information
on decreasing life coverage.

<details>
<summary><b>Life fields</b></summary>

---

🟥 **Life.Result**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

This field contains the calculation result for the requested protection product.
If it contains a value of "Valid Calculation", then the requested product was
computed and factored into the loan. Parse the other fields and child objects
for further details.

A value *other than* "Valid Calculation" means that the requested product was
not computed with the loan, and the value describes why. In this case, there is
no need to parse through the other fields or child objects, as the product was
not computed.

🟥 **Life.Formula**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Formula` field contains an abbreviated description of the formula used to
compute the desired protection product. The formula codes are for the use of the
J. L. Sherman and Associates, Inc. support team.

🟥 **Life.RateType**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Fixed |

This field will only be present in the response if the protection
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

🟥 **Life.Cost**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

Fields of this object provide the cost of the protection product in total, per
payment, and per day. It also provides the rate factor used to compute the
premiums.

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

The cost of coverage expressed as currency per payment.

🟥 **Cost.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as currency per dey.

🟥 **Cost.Factor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The rate factor used to compute the premium for the requested protection product.

🟦 **Cost.Premium2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The only time the `Premium2` field will be present is when the account has been
setup to use an ordinary life product *and* the user has requested joint
coverage. If this is the case, then the `Premium2` field is provided so that the
calling application can disclose two premiums instead of a single aggregate
joint premium.

🟦 **Cost.PerPmt2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The cost of coverage for the secondary ordinary life borrower, expressed as
currency per payment. Please see `Cost.Premium2` above for further information.

🟦 **Cost.PerDay2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The cost of coverage for the secondary ordinary life borrower, expressed as
currency per dey. Please see `Cost.Premium2` above for further information.

🟦 **Cost.Factor2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Floating |

The rate factor used to compute the premium for the secondary ordinary life
borrower. Please see `Cost.Premium2` above for further information.

---

</details>

🟥 **Life.Coverage**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The aggregate coverage amount and note are provided in the following fields of
this object:

<details>
<summary><b>Coverage fields</b></summary>

---

🟥 **Coverage.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The aggregate coverage amount for this protection product.

🟥 **Coverage.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If full coverage
has been provided on the aggregate coverage, then the note will contain "Full
Coverage". Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟦 **Life.Term**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Term` object provides the calling application with data about the term of
coverage for the requested product. If the input request has specified
`"ShowBorrowerInfo" : true`, then this object will not be present.

<details>
<summary><b>Term fields</b></summary>

---

🟥 **Term.InMonths**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the term of coverage expressed as a number of months.

🟥 **Term.InPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the number of payments covered.

🟥 **Term.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

This field contains the maturity date for the requested coverage.

🟥 **Term.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If
full term coverage has been provided, then the note will contain
"Full Coverage". Otherwise, the note will describe the type of truncated
coverage used.

---

</details>

🟦 **Life.Borrower1**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower1` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

<details>
<summary><b>Borrower1 fields</b></summary>

---

🟥 **Borrower1.Birthday**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The birthday associated with the borrower, as specified in the request.

🟥 **Borrower1.AgeAtIssue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at issue of the borrower in a special date-like format of `YYYY-MM-DD`,
where the borrower is `YYYY` years, `MM` months, and `DD` days old when coverage
begins.

🟥 **Borrower1.AgeAtMaturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at maturity of the borrower in a special date-like format of
`YYYY-MM-DD`, where the borrower is `YYYY` years, `MM` months, and `DD` days old
when coverage terminates.

🟥 **Borrower1.Pmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of payments.

🟥 **Borrower1.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of months.

🟥 **Borrower1.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The coverage maturity date for this particular borrower.

🟥 **Borrower1.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The value of this field will describe the type of coverage provided. If full
term coverage has been provided, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of truncated coverage used.

---

</details>

🟦 **Life.Borrower2**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower2` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

Please note that the fields of the `Borrower2` object are identical to the
`Borrower1` object. See the documentation above for the `Borrower1` object for
further information.

🟦 **Life.Caps**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Caps` object provides the calling application with data about the maximum
terms, amounts, and ages associated with the requested product. This objectt
will only be present if the `ShowCaps` field of the `Protection` request objectt
is set to `true`.

<details>
<summary><b>Caps fields</b></summary>

---

🟥 **Caps.Cov**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum aggregate coverage amount. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.Ben**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum monthly benefit amount. If no cap is present or applicable,
then a value of zero is returned.

🟥 **Caps.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum coverage term, expressed in months. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.InceptAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may be at loan inception, expressed in
years. If no cap is present or applicable, then a value of zero is returned.

🟥 **Caps.AttainAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may attain during the term of the loan,
expressed in years. If no cap is present or applicable, then a value of zero is
returned.

</details>

---

</details>

🟦 **Protection.Level**

| Type  | Required |
| :---: |   :---:  |
| object | no |

If life protection was requested with the loan and level life was configured for this
loan type, then the `Level` object will be present in the response to return information
on level life coverage.

<details>
<summary><b>Level fields</b></summary>

---

🟥 **Level.Result**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

This field contains the calculation result for the requested protection product.
If it contains a value of "Valid Calculation", then the requested product was
computed and factored into the loan. Parse the other fields and child objects
for further details.

A value *other than* "Valid Calculation" means that the requested product was
not computed with the loan, and the value describes why. In this case, there is
no need to parse through the other fields or child objects, as the product was
not computed.

🟥 **Level.Formula**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Formula` field contains an abbreviated description of the formula used to
compute the desired protection product. The formula codes are for the use of the
J. L. Sherman and Associates, Inc. support team.

🟥 **Level.Cost**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

Fields of this object provide the cost of the protection product in total, per
payment, and per day. It also provides the rate factor used to compute the
premiums.

<details>
<summary><b>Cost fields</b></summary>

---

🟥 **Cost.Premium**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The total cost for this protection over the term of the loan.

---

🟥 **Cost.PerPmt**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as currency per payment.

🟥 **Cost.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as currency per dey.

🟥 **Cost.Factor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The rate factor used to compute the premium for the requested protection product.

🟦 **Cost.Premium2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The only time the `Premium2` field will be present is when the account has been
setup to use an ordinary life product *and* the user has requested joint
coverage. If this is the case, then the `Premium2` field is provided so that the
calling application can disclose two premiums instead of a single aggregate
joint premium.

🟦 **Cost.PerPmt2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The cost of coverage for the secondary ordinary life borrower, expressed as
currency per payment. Please see `Cost.Premium2` above for further information.

🟦 **Cost.PerDay2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The cost of coverage for the secondary ordinary life borrower, expressed as
currency per dey. Please see `Cost.Premium2` above for further information.

🟦 **Cost.Factor2**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Floating |

The rate factor used to compute the premium for the secondary ordinary life
borrower. Please see `Cost.Premium2` above for further information.

---

</details>

🟥 **Level.Coverage**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The aggregate coverage amount and note are provided in the following fields of
this object:

<details>
<summary><b>Coverage fields</b></summary>

---

🟥 **Coverage.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The aggregate coverage amount for this protection product.

🟥 **Coverage.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If full coverage
has been provided on the aggregate coverage, then the note will contain "Full
Coverage". Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟦 **Level.Term**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Term` object provides the calling application with data about the term of
coverage for the requested product. If the input request has specified
`"ShowBorrowerInfo" : true`, then this object will not be present.

<details>
<summary><b>Term fields</b></summary>

---

🟥 **Term.InMonths**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the term of coverage expressed as a number of months.

---

🟥 **Term.InPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the number of payments covered.

🟥 **Term.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

This field contains the maturity date for the requested coverage.

🟥 **Term.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If
full term coverage has been provided, then the note will contain
"Full Coverage". Otherwise, the note will describe the type of truncated
coverage used.

---

</details>

🟦 **Level.Borrower1**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower1` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

<details>
<summary><b>Borrower1 fields</b></summary>

---

🟥 **Borrower1.Birthday**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The birthday associated with the borrower, as specified in the request.

🟥 **Borrower1.AgeAtIssue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at issue of the borrower in a special date-like format of `YYYY-MM-DD`,
where the borrower is `YYYY` years, `MM` months, and `DD` days old when coverage
begins.

🟥 **Borrower1.AgeAtMaturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at maturity of the borrower in a special date-like format of
`YYYY-MM-DD`, where the borrower is `YYYY` years, `MM` months, and `DD` days old
when coverage terminates.

🟥 **Borrower1.Pmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of payments.

🟥 **Borrower1.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of months.

🟥 **Borrower1.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The coverage maturity date for this particular borrower.

🟥 **Borrower1.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The value of this field will describe the type of coverage provided. If full
term coverage has been provided, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of truncated coverage used.

---

</details>

🟦 **Level.Borrower2**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower2` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

Please note that the fields of the `Borrower2` object are identical to the
`Borrower1` object. See the documentation above for the `Borrower1` object for
further information.

🟦 **Level.Caps**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Caps` object provides the calling application with data about the maximum
terms, amounts, and ages associated with the requested product. This objectt
will only be present if the `ShowCaps` field of the `Protection` request objectt
is set to `true`.

<details>
<summary><b>Caps fields</b></summary>

---

🟥 **Caps.Cov**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum aggregate coverage amount. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.Ben**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum monthly benefit amount. If no cap is present or applicable,
then a value of zero is returned.

🟥 **Caps.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum coverage term, expressed in months. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.InceptAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may be at loan inception, expressed in
years. If no cap is present or applicable, then a value of zero is returned.

🟥 **Caps.AttainAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may attain during the term of the loan,
expressed in years. If no cap is present or applicable, then a value of zero is
returned.

</details>

---

</details>

🟦 **Protection.Disability**

| Type  | Required |
| :---: |   :---:  |
| object | no |

If disability protection was requested with the loan, then the
`Disability` object will be present in the response to return information on
disability coverage.

<details>
<summary><b>Disability fields</b></summary>

---

🟥 **Disability.Result**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

This field contains the calculation result for the requested protection product.
If it contains a value of "Valid Calculation", then the requested product was
computed and factored into the loan. Parse the other fields and child objects
for further details.

A value *other than* "Valid Calculation" means that the requested product was
not computed with the loan, and the value describes why. In this case, there is
no need to parse through the other fields or child objects, as the product was
not computed.

🟥 **Disability.Formula**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Formula` field contains an abbreviated description of the formula used to
compute the desired protection product. The formula codes are for the use of the
J. L. Sherman and Associates, Inc. support team.

🟥 **Disability.RateType**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Fixed |

This field will only be present in the response if the protection
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

🟥 **Disability.Cost**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

Fields of this object provide the cost of the protection
product in total, per payment, and per day. It also provides the
rate factor used to compute the premiums.

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

The cost of coverage expressed as currency per payment.

🟥 **Cost.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as currency per dey.

🟥 **Cost.Factor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The rate factor used to compute the premium for the requested protection product.

---

</details>

🟥 **Disability.Coverage**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The aggregate coverage amount and note are provided in the following fields of
this object:

<details>
<summary><b>Coverage fields</b></summary>

---

🟥 **Coverage.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The aggregate coverage amount for this protection product.

🟥 **Coverage.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If
full coverage has been provided on the aggregate coverage, then the note will contain
"Full Coverage". Otherwise, the note will describe the type of partial
coverage used.

---

</details>

🟥 **Disability.Benefit**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The protection product's benefit amount and note are provided in the following
fields of this object:

<details>
<summary><b>Benefit fields</b></summary>

---

🟥 **Benefit.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The monthly benefit amount for this protection product.

🟦 **Benefit.Periodic**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the account has been configured to disclose periodic benefits (as opposed
to monthly benefit amounts, which are returned in the `Amount` field
described above), and if the specified payment frequency is other than monthly,
then this field will be present and will hold the periodic benefit amount.

🟥 **Benefit.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field describes the type of coverage provided. If full coverage has
been provided on the benefit, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟦 **Disability.Term**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Term` object provides the calling application with data about the term of
coverage for the requested product. If the input request has specified
`"ShowBorrowerInfo" : true`, then this object will not be present.

<details>
<summary><b>Term fields</b></summary>

---

🟥 **Term.InMonths**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the term of coverage expressed as a number of months.

🟥 **Term.InPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the number of payments covered.

🟥 **Term.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

This field contains the maturity date for the requested coverage.

🟥 **Term.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If
full term coverage has been provided, then the note will contain
"Full Coverage". Otherwise, the note will describe the type of truncated
coverage used.

---

</details>

🟦 **Disability.Borrower1**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower1` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

<details>
<summary><b>Borrower1 fields</b></summary>

---

🟥 **Borrower1.Birthday**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The birthday associated with the borrower, as specified in the request.

🟥 **Borrower1.AgeAtIssue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at issue of the borrower in a special date-like format of `YYYY-MM-DD`,
where the borrower is `YYYY` years, `MM` months, and `DD` days old when coverage
begins.

🟥 **Borrower1.AgeAtMaturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at maturity of the borrower in a special date-like format of
`YYYY-MM-DD`, where the borrower is `YYYY` years, `MM` months, and `DD` days old
when coverage terminates.

🟥 **Borrower1.Pmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of payments.

🟥 **Borrower1.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of months.

🟥 **Borrower1.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The coverage maturity date for this particular borrower.

🟥 **Borrower1.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The value of this field will describe the type of coverage provided. If full
term coverage has been provided, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of truncated coverage used.

---

</details>

🟦 **Disability.Borrower2**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower2` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

Please note that the fields of the `Borrower2` object are identical to the
`Borrower1` object. See the documentation above for the `Borrower1` object for
further information.

🟦 **Disability.Caps**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Caps` object provides the calling application with data about the maximum
terms, amounts, and ages associated with the requested product. This objectt
will only be present if the `ShowCaps` field of the `Protection` request objectt
is set to `true`.

<details>
<summary><b>Caps fields</b></summary>

---

🟥 **Caps.Cov**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum aggregate coverage amount. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.Ben**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum monthly benefit amount. If no cap is present or applicable,
then a value of zero is returned.

🟥 **Caps.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum coverage term, expressed in months. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.InceptAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may be at loan inception, expressed in
years. If no cap is present or applicable, then a value of zero is returned.

🟥 **Caps.AttainAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may attain during the term of the loan,
expressed in years. If no cap is present or applicable, then a value of zero is
returned.

</details>

---

</details>

🟦 **Protection.Unemployment**

| Type  | Required |
| :---: |   :---:  |
| object | no |

If unemployment protection was requested with the loan, then the `Unemployment`
object will be present in the response to return information on involuntary
unemployment coverage.

<details>
<summary><b>Unemployment fields</b></summary>

---

🟥 **Unemployment.Result**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

This field contains the calculation result for the requested protection product.
If it contains a value of "Valid Calculation", then the requested product was
computed and factored into the loan. Parse the other fields and child objects
for further details.

A value *other than* "Valid Calculation" means that the requested product was
not computed with the loan, and the value describes why. In this case, there is
no need to parse through the other fields or child objects, as the product was
not computed.

🟥 **Unemployment.Formula**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Formula` field contains an abbreviated description of the formula used to
compute the desired protection product. The formula codes are for the use of the
J. L. Sherman and Associates, Inc. support team.

🟥 **Unemployment.RateType**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Fixed |

This field will only be present in the response if the protection
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

🟥 **Unemployment.Cost**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

Fields of this object provide the cost of the protection
product in total, per payment, and per day. It also provides the
rate factor used to compute the premiums.

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

The cost of coverage expressed as currency per payment.

🟥 **Cost.PerDay**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The cost of coverage expressed as currency per dey.

🟥 **Cost.Factor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Floating |

The rate factor used to compute the premium for the requested protection product.

---

</details>

🟥 **Unemployment.Coverage**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The aggregate coverage amount and note are provided in the following fields of
this object:

<details>
<summary><b>Coverage fields</b></summary>

---

🟥 **Coverage.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The aggregate coverage amount for this protection product.

🟥 **Coverage.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If full coverage
has been provided on the aggregate coverage, then the note will contain "Full
Coverage". Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟥 **Unemployment.Benefit**

| Type  | Required |
| :---: |   :---:  |
| object | yes |

The protection product's benefit amount and note are provided in the following
fields of this object:

<details>
<summary><b>Benefit fields</b></summary>

---

🟥 **Benefit.Amount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

The monthly benefit amount for this protection product.

🟦 **Benefit.Periodic**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

If the account has been configured to disclose periodic benefits (as opposed
to monthly benefit amounts, which are returned in the `Amount` field
described above), and if the specified payment frequency is other than monthly,
then this field will be present and will hold the periodic benefit amount.

🟥 **Benefit.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field describes the type of coverage provided. If full coverage has
been provided on the benefit, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of partial coverage used.

---

</details>

🟦 **Unemployment.Term**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Term` object provides the calling application with data about the term of
coverage for the requested product. If the input request has specified
`"ShowBorrowerInfo" : true`, then this object will not be present.

<details>
<summary><b>Term fields</b></summary>

---

🟥 **Term.InMonths**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the term of coverage expressed as a number of months.

🟥 **Term.InPmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the number of payments covered.

🟥 **Term.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

This field contains the maturity date for the requested coverage.

🟥 **Term.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The `Note` field will describe the type of coverage provided. If
full term coverage has been provided, then the note will contain
"Full Coverage". Otherwise, the note will describe the type of truncated
coverage used.

---

</details>

🟦 **Unemployment.Borrower1**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower1` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

<details>
<summary><b>Borrower1 fields</b></summary>

---

🟥 **Borrower1.Birthday**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The birthday associated with the borrower, as specified in the request.

🟥 **Borrower1.AgeAtIssue**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at issue of the borrower in a special date-like format of `YYYY-MM-DD`,
where the borrower is `YYYY` years, `MM` months, and `DD` days old when coverage
begins.

🟥 **Borrower1.AgeAtMaturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

If coverage is written on this borrower, then the value of this field represents
the age at maturity of the borrower in a special date-like format of
`YYYY-MM-DD`, where the borrower is `YYYY` years, `MM` months, and `DD` days old
when coverage terminates.

🟥 **Borrower1.Pmts**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of payments.

🟥 **Borrower1.Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

The term of coverage expressed as a number of months.

🟥 **Borrower1.Maturity**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The coverage maturity date for this particular borrower.

🟥 **Borrower1.Note**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text - See below |

The value of this field will describe the type of coverage provided. If full
term coverage has been provided, then the note will contain "Full Coverage".
Otherwise, the note will describe the type of truncated coverage used.

---

</details>

🟦 **Unemployment.Borrower2**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Borrower2` object provides the calling application with data about the term
of coverage of a borrower, for the requested product. Note that this object will
only be present if the request has specified `"ShowBorrowerInfo" : true`, and a
valid birthdate was provided.

Please note that the fields of the `Borrower2` object are identical to the
`Borrower1` object. See the documentation above for the `Borrower1` object for
further information.

🟦 **Unemployment.Caps**

| Type  | Required |
| :---: |   :---:  |
| object | no |

The `Caps` object provides the calling application with data about the maximum
terms, amounts, and ages associated with the requested product. This objectt
will only be present if the `ShowCaps` field of the `Protection` request objectt
is set to `true`.

<details>
<summary><b>Caps fields</b></summary>

---

🟥 **Caps.Cov**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum aggregate coverage amount. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.Ben**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Currency |

Contains the maximum monthly benefit amount. If no cap is present or applicable,
then a value of zero is returned.

🟥 **Caps.Term**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum coverage term, expressed in months. If no cap is present or
applicable, then a value of zero is returned.

🟥 **Caps.InceptAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may be at loan inception, expressed in
years. If no cap is present or applicable, then a value of zero is returned.

🟥 **Caps.AttainAge**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Contains the maximum age a borrower may attain during the term of the loan,
expressed in years. If no cap is present or applicable, then a value of zero is
returned.

</details>

</details>

</details>

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

🟦 **GrandTotals.CLTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `CLTot` field will only appear on loans with certain types of life
protection products, such as those based on a monthly outstanding balance. It
contains the total amount paid for life over the duration of the loan.

🟦 **GrandTotals.AHTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `AHTot` field will only appear on loans with certain types of disability or
debt protection products, such as those based on a monthly outstanding balance.
It contains the total amount paid for this protection over the duration of the
loan.

🟦 **GrandTotals.IUTot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `IUTot` field will only appear on loans with certain types of involuntary
unemployment protection products, such as those based on a monthly outstanding
balance. It contains the total amount paid for this protection over the duration
of the loan.

🟦 **GrandTotals.PMITot**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `PMITot` attribute will only appear on loans with PMI insurance where the
PMI premiums were requested in the amortization schedule. It contains the total
PMI amount paid (not including any up front periodic PMI premiums)for PMI over
the duration of the loan.

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

🟦 **SubTotal.CLSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `CLSub` field will only appear on loans with certain types of life
protection products, such as those based on a monthly outstanding balance. It
contains the total amount paid for life during the year.

🟦 **SubTotal.AHSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `AHSub` field will only appear on loans with certain types of accident and
health or debt protection products, such as those based on a monthly outstanding
balance. It contains the total amount paid for this protection during the year.

🟦 **SubTotal.IUSub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `IUSub` field will only appear on loans with certain types of involuntary
unemployment protection products, such as those based on a monthly outstanding
balance. It contains the total amount paid for involuntary unemployment during
the year.

🟦 **SubTotal.PMISub**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `PMISub` field will only appear on loans with PMI insurance where the PMI
premiums were requested in the amortization schedule. It contains the total PMI
amount paid (not including any up front periodic PMI premiums) for PMI during
the year.

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

🟦  **AmLine.CL**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `CL` field will only appear on loans with certain types of life protection
products, such as those based on a monthly outstanding balance. It contains the
amount of the payment which is marked for life coverage.

🟦  **AmLine.AH**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `AH` field will only appear on loans with certain types of accident and
health or debt protection products, such as those based on a monthly outstanding
balance. It contains the amount of the payment which is marked for disability /
debt protection coverage.

🟦  **AmLine.IU**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

The `IU` field will only appear on loans with certain types of involuntary
unemployment protection products, such as those based on a monthly outstanding
balance. It contains the amount of the payment which is marked for this
coverage.

🟦  **AmLine.PMI**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | no | Number - Currency |

This field contains the PMI premium for this payment, and will only show up if
PMI has been computed for this payment and if PMI premiums should be displayed
in the amortization schedule.

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
| [Fixed Principal Plus Interest Loans](module-principalplus.md) | [SCEJSON Reference Manual](README.md) | [Single Payment Notes](module-singlepmt.md) |
