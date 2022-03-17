# Apr Verification and Calculation Module

> "It is said that love makes the world go 'round -  
>  the announcement lacks verification. It's  
>  wind from the dinner horn that does it."  
>                           --- O. Henry

If a loan is computed outside of the SCE and does not provide the Regulation Z /
EU APR, or if you wish to validate a computed APR, then the APR Verification and
Calculation functionality found in the SCE and described here will be of
interest to you. The SCE supports APR calculation and verification for loans
with a single advance, as well as multiple advance loans. For the United States
of America, all references to Appendix J of Regulation Z are put in italics at
the end of the field's description. For example, *Section (a)(1)* refers to
Appendix J, Section a, Subsection 1 of Regulation Z.

## Apr Data Object Field Definition

**Example - Request Envelope for Apr Module**

The following is a sample SCE request for an APR check calculation with an Amount Financed
of $10,000, and 36 monthly payments of $322.67.

```json
{
  "Module" : "Apr",
  "Data" : {
    "TestApr" : "10.125",
    "TestFinChg" : "1616.12",
    "TestTotPmt" : "11616.12",
    "Advances" : [
      {
        "Date" : "2022-03-16",
        "AmtFin" : "10000.00"
      }
    ],
    "PmtStreams" : [
      {
        "Begin" : "2022-04-16",
        "Term" : "36",
        "Pmt" : "322.67"
      }
    ]
  }
}
```

The `Data` object in the Apr module request is defined below:

### 🟦 ActuarialOpts

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object is only considered if the APR `Method` specified is
`Actuarial`.

<details><summary><b>ActuarialOpts fields</b></summary>

---  

🟦 **ActuarialOpts.ForceSemiMonth**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

The SCE strictly defines a semimonthly unit period as a majority of periods that have "every
other payment a month apart and 15 days between payments" as the most common period of time.
Since Appendix J loosely defines what a semimonthly period is and a reasonable use of the APR
Tool application would allow a looser definition, the SCE now allows for overriding the
strict definition with, for example, payments on the 1st and 15th being classified as having
a unit period of `Semimonth`. If an institution wishes a looser definition of semimonth,
this setting should be set to `true`. Otherwise, the strict rule applies.

---

🟦 **ActuarialOpts.ODI**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Loans that have used an odd days interest prepaid charge, common with mortgage
loans, may use this attribute to adjust the APR to assume one unit period to the
first payment. Otherwise, choose `false`.

---

🟦 **ActuarialOpts.SemiMonthlyFixedFraction**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

This field reflects an unintended consequence of Appendix J to Regulation Z's characterization
of the fraction of a unit period for semimonthly loans. Applying the rules of Appendix J
strictly to each payment in the payment stream results in an oscillating fraction of a unit
period. For a constant fraction of a unit period, choose `true`. To apply the Federal Calendar
to each payment, choose `false`. The latter should typically be selected with an irregular
loan.

*Section (b)(5)(iii)*

---

🟦 **ActuarialOpts.SinglePayFraction**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | InDays, InMonths | InDays |

This field only applies to single advance, single payment transactions of term
less than one year. For these loans, when the term of the loan is a number of
months, Appendix J allows for the term of the loan to be expressed as either a
number of months over twelve, or the number of actual 24-hour days in the loan
over 365. For the former, use `InMonths`. For the latter, use `InDays`.

 *Section (b)(5)(vi)*

---
</details>

### 🟦 AmTableOpts

| Type  | Required |
| :---: |   :---:  |
| Object | no |

The fiedls of this object allow the calculation and disclosure of the APR to be adjusted in 
rarely-needed ways.

<details><summary><b>AmTableOpts fields</b></summary>

---  

🟦 **AmTableOpts.DollarDec**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 3, 4, 5, 6, 7, 8, 9 | 4 |

Currency amounts within lines of the amortization schedule are disclosed as full
floating numbers, which can produce long lines of amortization in the disclosure.
The number of decimals for currency quantities may be trimmed to the number of decimal places
indicated by this attribute. Note that this number does not affect the calculation. It
merely affects to what number of decimals places to which the floating point number is disclosed.
`9` means "do not trim".

---

🟦 **AmTableOpts.ForceDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 0, 29, 30, 31 | 0 |

End of month considerations may require that a payment's day was intended to be another day.
For instance, if payments were intended to be made on the end of the month, and
the first payment is in April, making this setting `31` causes the fed calendar to compute
time as if the day of the payment was 31, rather than 30.

---

🟦 **AmTableOpts.RoundInt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | None, Down, Nearest, Up | None |

If interest is rounded within the amortization schedule for US Rule APRs, set the
rounding method here.

---
</details>

### 🟥 Advances

| Type  | Required |
| :---: |   :---:  |
| array of Advance objects | yes |

This array of `Advance` objects must have at least one member, and is used to
specify the advances made to the borrower. The amount of each advance is equal
to the total amount earning interest less any prepaid fees. One may think of
these entries as streams of Amounts Financed (in the RegulationZ sense).

<details><summary><b>Advances fields</b></summary>

---

🟥 **Advance.AmtFin**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

The proceeds to be advanced to the borrower less any prepaid fees for each of
the `Term` advances is defined using this field. 

---

🟥 **Advance.Date** (Date)

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field's value holds the date on which the advance is made. All dates must
be in the form of YYYY-MM-DD, and be 10 characters long. Hence, an advance date
of January 2, 2021 would be specified as `"Date" : "2021-01-02"`.

---

🟦 **Advance.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

This field is used to resolve ambiguitiess in subsequent advance dates when the
`Date` falls on the last day of a month with fewer than 31 days. If the value of
the `Term` field is `1`, then the value of this field is ignored.

Set this field's value to `true` if the intent was to make subsequent advances
occur on the last day of the month. A value of `false` indicates that subsequent
advance dates will fall on the day number specified by the `Date` field.

---

🟦 **Advance.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and in the case of the Advance
object, determines the frequency for the advance stream. If the value of the `Term`
field is `1`, then this field is ignored.

---

🟦 **Advance.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | integer > 1 | 1 |

The `Term` field determines the the number of advances to be included at the
specified frequency (see `Advance.PPY` above), after which the advance stream is
completed. The default value is `1`.

---
</details>

### 🟦 Construction

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Loans which have a construction period require data to be passed using this
object. All of the information used to define the construction period is
contained in the following fields of the object:

<details>
<summary><b>Construction fields</b></summary>

---

🟦 **Construction.AsPrepaid**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | ??? |

This field is only applicable when the `PermanentAttached` field is `true`. If
the permanent loan treats the construction interest as a prepaid finance fee,
then set the value of this attribute to `true`.

On the other had, if the construction interest is not treated as a prepaid
finance fee and is instead included as a stream of interest only payments during
the construction period, then set the value of this attribute to `false`.

---

🟥 **Construction.EndDate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

The date on which the construction period terminates.

All dates must be in the form of "YYYY-MM-DD", and be 10 characters long.
Hence, to end the construction period on July 4, 2021, the field would be
specified as `"EndDate" : "2021-07-04"`.

---

🟦 **Construction.HalfCommitment**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

During the construction period, if estimated interest is disclosed based
upon half of the commitment amount, then set the value of this field to `true`.
If interest is due on the entire commitment amount during the construction
period, then set this value to `false`. Multiple advance construction loans are
computed with this field set as `true`.

---

🟥 **Construction.Interest**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

The total amount of construction interest.

---

🟥 **Construction.PermanentAttached**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | true | true, false | n/a |

Construction loans may be computed on their own, or they may be attached to a
permanent loan which begins at the conclusion of the construction loan. If a
permanent loan is attached to the construction loan, then set this field's
value to `true`. If no permanent loan is attached to the construction loan,
then set the field's value to `false`.

---

🟦 **Construction.Prepaid**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | 0 |

This field holds the total prepaid charge for points and fees, excluding
construction interest. This attribute need only be specified when the
`PermanentAttached` field is `false`.

---
</details>

### 🟦 Decimals

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 3, 4, 5 | see below |

The number of decimal places of accuracy for the disclosed APR is determined by
this field. The default value of this field depends upon the selected `Method`.
For all methods other than `EU_APR`, the default value is `3`. For `EU_APR`, the
default value is `1`.

### 🟦 Method

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Actuarial, USRUle, EU_APR, IRR, YieldIRR, XIRR, XIRR360 | Actuarial |

In the United States of American, Regulation Z allows for APRs to be
computed using two different methods: the actuarial method (`Actuarial`)
or the US Rule method (`USRule`). In the European Union, there
is a different method used (`EU_APR`) to compute the APR. The
SCE also supports the calculation of the internal rate of return as
defined by Microsoft&reg; Excel's XIRR() function (`XIRR`),
as well as the internal rate of return based upon a unit period 360 days
calendar (`XIRR360`). The `IRR` method implements a common
spreadsheet internal rate of return method which assumes strict regular
periods. Deviating from a strict regular periodicity may produce
unreliable results. The `YieldIRR` method is similar to the
`IRR` method, except the rate is converted to an annual yield 
equivalent, according to the payment frequency.

Use this attribute to specify the method used to compute the APR. If
a value of `USRule` is specified, make sure to include the
`USRule` object as appropriate.

### 🟦 PmtStreams

| Type  | Required |
| :---: |   :---:  |
| array of PmtStream objects | yes |

One or more `PmtStream` objects are required to specify the loan's
payment stream. For an equal payment loan, a single payment stream element is
all that is required. For a balloon payment stream, you will need two. For
highly irregular loans (skips, pickups, interest only, etc.) you will need
many. All necessary data for each payment stream is contained in the object's
fields, which are described below.

<details>
<summary><b>PmtStream fields</b></summary>

---

🟥 **PmtStream.Begin**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field contains the date on which this payment stream begins. All
dates must be in the form of YYYY-MM-DD, and be 10 characters long.

---

🟦 **PmtStream.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

This field is used to resolve ambiguitiess in subsequent payment
dates when the `Begin` date falls on the last day of a month
with fewer than 31 days. If the value of the `Begin` field
is not a valid date, then the value of this field is ignored.

Set this field's value to `true` if the intent was to make
subsequent payments occur on the last day of the month. A value of
`false` indicates that subsequent payment dates will fall on the
day number specified by `Begin`.

As an example, if the calling application specifies a `Begin`
of `2010-02-281, then subsequent payment dates for a monthly
payment frequency will be:

- **on the 28th of each subsequent month** if `"LastDay" : false`.
- **on the last day of each subsequent month** if "LastDay : true`.

---

🟥 **PmtStream.Pmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

This field defines the principal and interest portion of the payment for
each of the `Term` payments. If the total payment includes any
protection products such as MOB life or debt protection fees, then these
amounts must be removed as they are not a part of the principal and interest
portion of the payment.

---

🟦 **PmtStream.PPY**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and as one might surmise,
determines the payment frequency for the payment stream. If the value of
the `Date`` field is not a valid date, then the value of this
field is ignored.

---

🟦 **PmtStream.SemimonthlyDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 0 |

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
      "Begin" : "2019-01-01",
      "Term" : "24",
      "PPY" : "24",
      "SemimonthlyDay" : "15"
    }
  ]
}
```

---

🟦 **PmtStream.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 1 |

The term field indicates the the number of *payments* to be made at the
specified payment frequency (see `PmtStream.PPY` above), after which the payment
stream is completed. The default value is `1`. If the value of the
`PmtStream.Date` field is not a valid date, then the value of this field is
ignored, unless it is a replacement payment using the `NNNN-00-00` date format.

---
</details>

### 🟦 Premiums

| Type  | Required |
| :---: |   :---:  |
| array of Premium objects | no |

Only loans that have premiums occurring on dates not equal to the payment dates
should use this element for expressing the premium. In the case of CUNA Mutual,
for example, premiums are paid monthly on the same day of the month, regardless
of when payments are made. In this case, please use the Premiums array for
specifying loan premiums.

<details><summary><b>Premium fields</b></summary>

---  

🟥 **Premium.Date**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

This field contains the date on which this premium stream begins. All
dates must be in the form of YYYY-MM-DD, and be 10 characters long. Hence,
a payment stream starting date of December 10, 2006 would
be specified as \texttt{Begin="2006-12-10"}.

---

🟦 **Premium.Freq**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12 | 12 |

The frequency of premiums.

---

🟦 **PmtStream.LastDay**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

Was the premium intended for the last day of the month?

---

🟥 **Premium.Prem**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

The Premium amount.

---

🟦 **Premium.Term**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 1 |

The number of premiums in the stream.

---
</details>

### 🟦 TestApr

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - %| n/a |

The SCE will compute an APR for the given loan, then compare the APR it has
computed against this value. The difference between the computed and test APRs
will be compared. The SCE will inform the calling application whether or not the
provided APR is within compliance. If you do not have a test APR to compare,
omit this field. The test APR should be expressed as a percentage.

### 🟦 TestFinChg

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - currency | n/a |

If this field is present and contains a value greater than zero, then the
SCE will compute the Regulation Z Finance Charge for the given loan and
compare it to the value specified in this field. The magnitude of the
difference between the computed and test values will be returned in the
response.

If this field is not present, or if it is present but contains a value
less than or equal to zero, then the Finance Charge will not be returned
in the response.

### 🟦 TestTotPmt 

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - currency | n/a |

If this field is present and contains a value greater than zero, then the SCE
will compute the Regulation Z Total of Payments for the given loan and compare
it to the value specified in this field. The magnitude of the difference between
the computed and test values will be returned in the response.

If this field is not present, or if it is present but contains a value less than
or equal to zero, then the Total of Payments is still returned in the XML
results, however no `Difference` or `TestValue` fields will be present.

### 🟦 TransactionDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | Date of earliest Advance |

The transaction date is generally the date of the earliet `Advance`.
If, for instance, the finance charge begins beyond the date of the first
advance, use this element to define the date the finance charge begins.

### 🟦 UnitPeriod

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If this object is not present in the request, then the SCE will compute the unit
period. Including this element instructs the SCE to use the unit period
specified by the calling application.

As an example, if you wanted to specify a biweekly unit period, then you would
include the following in the request:

```json
{
  "UnitPeriod" : {
    "Period" : "Week",
    "Multiple" : "2"
  }
}
```

Similarly, if you wanted to specify a quarterly unit period, then you would
include the follwing in the request:

```json
{
  "UnitPeriod" : {
    "Period" : "Month",
    "Multiple" : "3"
  }
}
```

<details><summary><b>UnitPeriod fields</b></summary>

---  

🟦 **UnitPeriod.Multiple**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 1 |

This field's value defines the integral number of standard unit periods in the
defined unit period.

---

🟦 **UnitPeriod.Period**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Day'', Week, Month, Year, or Semimonth | n/a |

This field defines the standard unit period. The unit period will be some
integral multiple of this standard unit period, as specified by the `Multiple`
field above.

---
</details>

### 🟦 UsRuleOpts

| Type  | Required |
| :---: |   :---:  |
| Object | no |

This object is only relevant if the APR `Method` specified is
`UsRule`. The fields of this object allow the calling application
to configure the parameters of the US Rule APR calculation.

<details><summary><b>UsRuleOpts fields</b></summary>

---  

🟦 **UsRuleOpts.Calendar**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | ActualDay, UnitPeriod, FedCalendar | ActualDay |

Actual day, unit period, and Federal calendars are supported with US Rule
interest accrual.

---

🟦 **UsRuleOpts.CalendarOption**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | None, True360, True365, Midnight366 | None |

`True360` means that each month is assumed to have 30 days in all respects. Typically, a loan
computes the actual number of days from the Advance to the first payment to compute the
Term Factor to the first payment.

`True365` means that February 29th is not allowed, both in counting days and assessing the days per
year divisor.

`Midnight366` only applies to Actual/366 calendars, and means that a day is the time from midnight
of one day to midnight of the other. The number of days between December 19th 2007 and January
19th 2008 would be 13 Days in a 365 year plus 18 days in a 366 day year, resulting in a Term
Factor of `13/365+18/366`. An Actual/366 calculation not using the Midnight366 setting would
compute the term factor as `12/365+19/366`.

---

🟦 **UsRuleOpts.Divisor**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 360, 365, 366, PerPeriod, 365_25, VarDPY | 365 |

Choose the divisor, or number by which to divide the day count to produce term factors.

`360` means take the computed days to the first payment and divide them by 360
to compute the term factor.

`365` means take the computed days to the first payment and divide them by 365.

`366` is  understood to mean `366` only in a leap year. Any days accruing
interest in a non-leap year use a 365 divisor. This setting may only be used
with `ActualDay` calendars.

`PerPeriod` means 360 for monthly and bimonthly periods, 364 for weekly
multiples, Quarterly, and SemiAnnual repayments, and 365 for annual loans.

`VarDPY` is understood to mean that the computed days to the first payment are
divided by the number of days in the month of the transaction date, multiplied
by twelve.

`365_25` means take the computed days to the first payment and divide them by
365.25 to compute the term factor. This setting may only be used with
`ActualDay` calendars.

---

🟦 **UsRuleOpts.PremBeforePmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | true |

Are premiums processed before or after payments? `true` means premiums are amortized
before payments, `false` means payments are processed before premiums.

---
</details>

## Apr Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for APR Module** *(hosted on AWS)*

```json
{
  "Result" : 200,
  "Module" : "Apr",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "Apr" : {
      "Value" : "10.000",
      "Method" : "Actuarial",
      "UnitPeriod" : "1_Month",
      "UnitPeriodBase" : "Month",
      "UnitPeriodMult" : "1",
      "PeriodsPerYear" : "12"
    },
    "TestResults" : {
      "Apr" : {
        "LoanType" : "Regular",
        "MultAdv" : false,
        "IrregPeriod" : false,
        "IrregPmt" : false,
        "Value" : "10.000",
        "TestValue" : "10.125",
        "Difference" : "0.125",
        "Tolerance" : "0.125",
        "InCompliance" : true,
        "OnCusp" : true
      },
      "FinChg" : {
        "Value" : "1616.12",
        "TestValue" : "1616.12",
        "Difference" : "0.00"
      },
      "TotPmt" : {
        "Value" : "11616.12",
        "TestValue" : "11616.12",
        "Difference" : "0.00"
      }
    },
    "Loan" : {
      "TransactionDate" : "2022-03-16",
      "AmountFinanced" : "10000.00",
      "NumAdvances" : "1",
      "AdvPresVal" : "10000.00",
      "FinChg" : "1616.12",
      "TotPmt" : "11616.12",
      "NumPmts" : "36",
      "TotPmtPresVal" : "9999.942"
    },
    "AmTable" : {
      "Error" : "0.058",
      "ErrorDown" : "-0.0875",
      "ErrorUp" : "0.2035",
      "AmLines" : [
        {
          "Idx" : "0",
          "Date" : "2022-03-16",
          "Unit" : "0",
          "Frac" : "0",
          "Adv" : "10000.00",
          "PresVal" : "10000.00",
          "PresValSum" : "10000.00"
        },
        {
          "Idx" : "1",
          "Date" : "2022-04-16",
          "Unit" : "1",
          "Frac" : "0",
          "Pmt" : "322.67",
          "PresVal" : "320.0033",
          "PresValSum" : "9679.9967"
        },
        ...
      ]
    }
  }
}
```

The `Data` object for a response from the Apr module is defined below, in the
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
  "Module" : "Apr",
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
  "Module" : "Apr",
  "Data" : {
    "Errors" : [
      "Data.Advances[] (Array) not found.",
      "Data.PmtStreams[] (Array) not found."
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ]
  }
}   
```


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Loan Module](module-loan.md) | [SCEJSON Reference Manual](README.md) | |
