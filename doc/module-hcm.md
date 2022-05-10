# High Cost Mortgages (HCM)

> "I can make more generals,  
>  but horses cost money."  
>  --- Abraham Lincoln

A loan is considered a high-cost mortgage if it is a consumer credit transaction
that is secured by the consumer's principal dwelling (with a few minor
exclusions) and one or more of the following conditions are met:

1. the APR test fails; 
2. the points and fees test fails; or
3. the prepayment penalty test fails.

The HCM module will determine if an already computed loan is considered a
high-cost mortgage. The calling application must furnish the SCE with the
required information, and then the SCE will perform the three required tests
mentioned above, returning the loan's status in its output.

Please note that the HCM module requires the use of the Average Prime Offer
Rates (APOR) tables, which are published weekly by the federal government. The
two files (one for fixed rates and one for adjustable rates) are available for
download at the following web site: https://ffiec.cfpb.gov/tools/rate-spread.
Please note that it is necessary that these files be updated weekly, or else the
SCE will not be able to determine if a loan is a HCM for loans whose lock in
dates fall outside the range of dates provided in the APOR files. If
you are using a version of the SCE that is hosted by J. L. Sherman and
Associates, Inc., then the APOR files will be automatically updated weekly for
your use.

Also note that there are two values specified in the HCM points and fees test
which will adjust annually. These values are stored in a file named `hcm.ini`.
This file needs to be located in the same directory as the APOR files mentioned
above. Similar to the APOR files, this file will be updated automatically for
your use if you are using a version of the SCE which is hosted by J. L. Sherman
and Associates, Inc.

## Hcm Request Data Object Field Definition

**Example - Request Envelope for Hcm Module**

The following is a sample SCE request for a Hcm calculation:

```json
{
  "Module" : "Hcm",
  "Data" : {
    "LockInDate" : "2022-03-09",
    "LienType" : "first",
    "RateType" : "fixed",
    "Dwelling" : "personal_property",
    "Term" : "360",
    "PPY" : "12",
    "LoanAmount" : "255102.04",
    "AmountFinanced" : "255102.04",
    "Apr" : "9.91",
    "FinanceCharge" : "292688.40",
    "InterestCharge" : "266366.36",
    "FederalStatePremiumsFees" : "0",
    "PMI" : {
      "After" : "20625.00",
      "AtOrBefore" : "625.00",
      "MaxAllowedAtOrBefore" : "7653.06"
    },
    "ThirdPartyCharges" : "0.00",
    "DiscountPoints" : {
      "Points" : "2.0",
      "FullRate" : "6.000",
      "Fee" : "5102.04"
    },
    "LoanOriginatorFees" : "0.00",
    "RealEstate" : {
      "Fees" : "0.00",
      "FinanceAmt" : "0.00"
    },
    "CreditInsurance" : {
      "Premiums" : "0.00",
      "FinanceAmt" : "0.00"
    },
    "PrepaymentPenalty" : {
      "Max" : "0.00",
      "Total" : "0.00",
      "FinanceAmt" : "0.00",
      "After36Months" : false,
      "AmountPrepaid" : "0.00"
    }
  }
}
```

The fields of the `Data` object supported by a Hcm module request are defined
in alphabetical order below:

### 🟥 AmountFinanced

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

As defined in Regulation Z (*Section 1026.18(b)*), the amount of credit provided
to the borrower or on the borrower's behalf. As an example, the SCEJSON's Loan
module returns the loan's Amount Financed in the `Data.FedBox.AmtFin` field.

### 🟥 Apr

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - % | n/a |

This field's value contains the APR applicable to the transaction, which is
*not* the same as the Regulation Z APR. The APR applicable to the transaction is
determined as follows (from *Section 1026.32(a)(3)*):

1. If the rate plan is fixed, then the APR equals the interest rate in effect as
   of the date the interest rate for the transaction is set.
2. If the rate plan varies based on an index, then the APR equals the maximum
   margin allowed during the term of the loan plus the value of the index rate
   in effect as of the date the interest rate for the transaction is set.
3. If the rate plan varies and is \emph{not} based on an index (case 2 above),
   then the APR equals the maximum interest rate tat may be imposed during the
   term of the loan.

### 🟦 CreditInsurance

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If credit life, credit disability, credit unemployment, or credit property
insurance are included in the loan (or any other life, accident, health, or
loss-of-income insurance for which the creditor is a beneficiary), then this
object should be included in the request with its fields set as appropriate. See
*Section 1026.32(b)(1)(iv)*.

<details>
<summary><b>CreditInsurance fields</b></summary>

---

🟦 **CreditInsurance.Premiums**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The total of all credit life, credit disability, credit unemployment, or credit
property insurance premiums.

---

🟦 **CreditInsurance.FinanceAmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The value of this field specifies the amount of the above specified 
premiums and/or fees which were financed.

---

</details>

### 🟦 DataPath

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | see below | see below |

If this field is set, the SCE will look for a data folder containing the
APOR and `hcm.ini` files in the path specified. Note that the APOR files must be
named as follows: `YieldTableFixed.txt` and `YieldTableAdjustable.txt`.

If this field is not set, the SCE will attempt to locate the file in the
default data directory. Note that the AWS hosted version of the SCE API server
places the necessary files in the correct default directory, and hence
specifying this field is not required.

Since the APOR and `hcm.ini` files are global in nature, you only need to have
one copy of these files available even if your installation hosts multiple data
paths for different clients. The APOR files are identical to those needed for
the Hpml module, and thus a single location containing the APOR files can be
used for both modules.

### 🟦 DiscountPoints

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If a loan features discount points, pass the appropriate data using the fields
of this object.

<details>
<summary><b>DiscountPoints fields</b></summary>

---

🟦 **DiscountPoints.Fee**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The total fee due for the discounted interest rate.

---

🟦 **DiscountPoints.FullRate**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - % | 0 |

The value of this field specifies the non-discounted rate, or the interest
rate without any discount.

---

🟦 **DiscountPoints.Points**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - % | 0 |

The value of this field specifies the number of bona fide discount points paid
by the borrower. According to *Section 1026.32(b)(3)(i)(D)*, a bona fide
discount point "means an amount equal to 1 percent of the loan amount paid by
the customer that reduces the interest rate".

---

</details>

### 🟥 Dwelling

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Other, Personal_property | n/a |

If this is a first Lien, then if the dwelling type is personal property
and the loan amount is less than fifty thousand, then the rate spread is
8.5%. For all other first lien loans, the rate spread is 6.5%.

### 🟦 FederalStatePremiums

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - currency | 0 |

The value of this element is the sum of all federal and state mortgage insurance
premiums and guarantee fees that are included in the Finance Charge (see
*Section 1026.32(b)(1)(i)(B)*).

### 🟥 FinanceCharge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

As defined in Regulation Z (*Section 1026.4*), the dollar amount that the credit
will cost the borrower. As an example, the SCEJSON's Loan module returns the
loan's Finance Charge in the `Data.FedBox.FinChg` field.

### 🟥 InterestCharge

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

The value of this field holds the total interest accrued during the term of the loan,
assuming the borrower will make all scheduled payments (see *Section 1026.32(b)(1)(i)(A)*).
As an example, the SCEJSON's Loan module returns the loan's
Interest Charge in the `Data.Moneys.Interest` field.

### 🟥 LienType

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | First, Subordinate | n/a |

The type of lien plays a part in determining the amount over the APOR (called
the rate spread) at which time a loan's APR is considered a HCM. For first-lien
loans, that value is 6.5% or 8.5% (depending upon the loan amount and dwelling
type); for subordinate-lien loans, that value is 8.5%.

### 🟥 LoanAmount

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | number - currency | n/a |

The Loan Amount is the face amount of the note (i.e., the principal balance).

### 🟦 LoanOriginatorFees

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - currency | 0 |

The value of this field represents all compensation paid directly or indirectly
by a consumer or creditor to a loan originator. See *Section 1026.32(b)(1)(ii)*.

### 🟥 LockInDate

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | YYYY-MM-DD | n/a |

The value of this field holds the date on which the interest rate of the loan
was locked in. All dates must be in the form of YYYY-MM-DD, and be 10 characters
long.

The lock in date is used to lookup the APOR rate from the appropriate file. Each
row of the file begins with a date, and the row which will be used will have a
date which is either on the lock in date, or does not preceed it by more than 6
days. Since the rates are updated weekly, there should never be more than six
days between the date of the row used and the specified lock in date.

If the SCE can not find a row in the APOR file which is appropriate for use with
the specified lock in date, then an error message will be returned informing the
calling application.

### 🟦 PMI

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Hcm requests which include private mortgage insurance require data to be passed
using this object.

<details>
<summary><b>PMI fields</b></summary>

---

🟦 **PMI.After**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 0 |

This field specifies the private mortgage insurance premiums payable after
consummation. See *Section 1026.32(b)(1)(i)(C)(1)*.

---

🟦 **PMI.AtOrBefore**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 0 |

The value of this field represents private mortgage insurance premiums paid at
or before consummation. See *Section 1026.32(b)(1)(i)(C)(2)*.

---

🟦 **PMI.MaxAllowedAtOrBefore**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | 0 |

The maximum private mortgage insurance premium paid at or before consummation
allowable under *Section 203(c)(2)(A)* of the National Housing Act. See
*Section 1026.32(b)(1)(i)(C)(2)*.

---

</details>

### 🟦 PrepaymentPenalty

| Type  | Required |
| :---: |   :---:  |
| Object | no |

If the loan features a prepayment penalty, include this object and its relevant
fields.

<details>
<summary><b>PrepaymentPenalty fields</b></summary>

---

🟦 **PrepaymentPenalty.After36Months**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| Boolean | no | true, false | false |

If, under the terms of the loan contract, the creditor can charge a prepayment
penalty more than 36 months after consummation, then the calling application
should set the value of this field to `true`.

Setting this value to `true` will trigger the prepayment penalty test, and his
classify the specified loan as a HCM.

---

🟦 **PrepaymentPenalty.AmountPrepaid**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

If the maximum prepayment penalty exceeds 2% of this field's value, then the
prepayment penalty test will be triggered.

---

🟦 **PrepaymentPenalty.FinanceAmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The value of this field specifies the amount of the total prepayment penalty
which was financed by the creditor.

---

🟦 **PrepaymentPenalty.Max**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The maximum prepayment penalty that may be charged or collected under the terms
of the mortgage loan. See *Section 1026.32(b)(1)(v)*.

---

🟦 **PrepaymentPenalty.Total**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The total prepayment penalty incurred by the consumer if the consumer refinances
the existing mortgage loan with the current holder of the existing loan, a
servicer acting on behalf of the current holder, or an affiliate of either. See
*Section 1026.32(b)(1)(vi)*.

---

</details>

### 🟦 PPY

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | 1, 2, 4, 6, 12, 24, 26, 52 | 12 |

PPY is an abbreviation for "payments per year", and specifies the payment
frequency for the initial fixed rate period of the given loan. The default value
of `12` will result in	a loan with an initial fixed rate period of 12 payments
per year. If the loan in question uses a payment frequency for the initial fixed
rate period other than monthly, specify it using this field.

### 🟥 RateType

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | yes | Adjustable, Fixed | n/a |

The APOR is looked up in one of two tables, determined by the loan's rate type.
If the loan uses a fixed rate, then the SCE will look-up the APOR from the
`YieldTableFixed.txt` file. If the loan uses a variable or adjustable rate, then
the SCE will look-up the APOR from the `YieldTableAdjustable.txt` file.

### 🟦 RealEstate

| Type  | Required |
| :---: |   :---:  |
| Object | no |

Please see *Section 1026.32(b)(1)(iii)* for a discussion of the real estate
fees and other charges which should be included here.

<details>
<summary><b>RealEstate fields</b></summary>

---

🟦 **RealEstate.Fee**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The total fee due for all appropriate real estate fees and other charges.

---

🟦 **RealEstate.FinanceAmt**

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - curency | 0 |

The value of this field specifies the amount of the above specified real
estate fees and other charges which were financed.

---

</details>

### 🟦 Term

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

The value of this field holds the number of payments *in the initial fixed rate
period of the loan*. If the loan uses a fixed rate, then this is equal to the
total number of payments in the loan term.

Note that the calling application *must* specify either the `Term` (and `PPY`)
or `TermInYears` field (below).

Note that the calling application *must* specify either the
`Term` and `PPY fields (above) or `TermInYears`.

### 🟦 TermInYears

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number | n/a |

The value of this field holds the term of the initial fixed rate period of the
loan, in whole years. Valid terms range from 1 year to 50 years, inclusive.

### 🟦 ThirdPartyCharges

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | number - currency | 0 |

This field holds the sum of all bona fide third-party charges included in the
Finance Charge, as defined in *Section 1026.32(b)(1)(i)(D)*.

## Hcm Response Data Object Field Definition

The following example is the response returned from the SCEJSON for the request
provided at the beginning of the previous section.

**Example - Response Envelope for Hcm Module** *(hosted on AWS)*

```json
{
    "Result": 200,
    "Module": "Hcm",
    "Data": {
        "Errors": [],
        "Warnings": [],
        "IsHcm": false,
        "Triggers": {
            "Apr": false,
            "PointsFees": false,
            "Prepayment": false
        },
        "AprTrigger": {
            "Date": "2022-03-07",
            "Apor": "3.830",
            "Spread": "6.500",
            "Difference": "-0.420"
        },
        "PointsFeesTrigger": {
            "TotalLoanAmount": "255102.04",
            "TotalPointsFees": "5072.04",
            "MaxPointsFees": "12755.10"
        },
        "PrepaymentTrigger": {
            "After36Months": false,
            "Total": "0.00",
            "Max": "0.00"
        }
    }
}
```

The `Data` object for a response from the Hcm module is defined below, in the
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
  "Module" : "Hcm",
  "Data" : {
    "//" : "This is a comment.",
    "Hello" : "Friend!",
    "How" : "are you?"
  }
}
```

```json
{
    "Result": 200,
    "Module": "Hcm",
    "Data": {
        "Errors": [
            "Data.LockInDate (StringDate) not found.",
            "Data.LienType (String) not found.",
            "Data.RateType (String) not found.",
            "Data.Dwelling (String) not found.",
            "Data.LoanAmount (StringFloat) not found.",
            "Data.AmountFinanced (StringFloat) not found.",
            "Data.Apr (StringFloat) not found.",
            "Data.FinanceCharge (StringFloat) not found.",
            "Data.InterestCharge (StringFloat) not found."
        ],
        "Warnings": [
            "Request field Data.Hello (String) not recognized.",
            "Request field Data.How (String) not recognized."
        ]
    }
}   
```

### 🟥 IsHcm

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

The value of this field will either be true or false, and will indicated
whether or not the submitted loan is a high cost mortgage.

### 🟥 Triggers

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

This object informs the calling application about that status of the three
possible triggers that can cause a loan to be classified as high cost.

<details>
<summary><b>RealEstate fields</b></summary>

---

🟦 **Triggers.Apr**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If the Apr test was triggered, then the value of this field will be `true`.

---


🟦 **Triggers.PointsFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If the Points and Fees test was triggered, then the value of this field will be
`true`.

---


🟦 **Triggers.Prepayment**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If the Prepayment test was triggered, then the value of this field will be
`true`.

---

</details>

### 🟥 AprTrigger

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

This object provides further information on the high cost mortgage Apr test,
which may be of interest to the calling application.

<details>
<summary><b>AprTrigger fields</b></summary>

---

🟥 **AprTrigger.Date**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | YYYY-MM-DD |

The Date field contains the date on which the APOR data was effective. As an
example, if the lock in date was 2013-05-09, then the date returned should be
2013-05-06 (if the APOR files are up to date).

---

🟥 **AprTrigger.Apor**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The value of this field contains the Average Prime Offer Rate for the submitted
loan. This value is retrieved from the appropriate APOR file (fixed or
adjustable).

---

🟥 **AprTrigger.Spread**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The spread value depends upon the type of lien, loan amount, and dwelling type.
The resulting spread will either be 6.5 or 8.5.

---

🟥 **AprTrigger.Difference**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number |

The value of this field returns the difference between the submitted APR
applicable to the transaction, and the APOR plus the spread. If the difference
is greater than zero (which means that the APR is greater than the APOR plus
spread), then the loan is a HCM.

---

</details>

### 🟥 PointsFeesTrigger

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

This object provides further information on the high cost mortgage points and
fees test, which may be of interest to the calling application.

<details>
<summary><b>PointsFeesTrigger fields</b></summary>

---

🟥 **PointsFeesTrigger.TotalLoanAmount**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The total loan amount is defined in *Section 1026.32(b)(4)*. It is calculated by
taking the Regulation Z Amount Financed and deducting some of the financed fees
and charges which are provided as inputs to the calculation.

---

🟥 **PointsFeesTrigger.TotalPointsFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The total points and fees value is defined in *Section 1026.32(b)(1)*. It is
calculated by taking the Regulation Z Finance Charge and deducting some portions
of the Finance Charge, and then adding other fees and charges which are provided
as inputs to the calculation.

---

🟥 **PointsFeesTrigger.MaxPointsFees**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

If the computed total points and fees value for the submitted loan exceeds this
value, then the loan is a high cost mortgage. The maximum points and fees value
calculation is defined in *Section 1026.32(a)(1)(ii)*.

---

</details>

### 🟥 PrepaymentTrigger

| Type  | Required |
| :---: |   :---:  |
| Object | yes |

This object provides further information on the prepayment test, which may be of
interest to the calling application.

<details>
<summary><b>PrepaymentTrigger fields</b></summary>

---

🟥 **PrepaymentTrigger.After36Months**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| Boolean | yes | true, false |

If the creditor can charge a prepayment penalty  more than 36 months after
consummation, then the loan is a high cost mortgage.

---

🟥 **PrepaymentTrigger.Total**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

This field returns the total potential amount of prepayment penalties.

---

🟥 **PrepaymentTrigger.Max**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | number - currency |

The maximum prepayment penalty is computed acording to *Section
1026.32(a)(1)(iii)*, and is equal to 2% of the specified `AmountPrepaid`. If the
`Total` amount exceeds this value, then the loan is a high cost mortgage.

---

</details>

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [APR Calculation & Verification Module](module-apr.md) | [SCEJSON Reference Manual](README.md) | [Higher Priced Mortgage Loans (HPML)](module-hpml.md) |
