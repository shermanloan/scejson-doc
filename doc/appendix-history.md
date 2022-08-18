# Appendix - History

> "History will be kind to me  
>  for I intend to write it."  
>  --- Winston Churchill

This appendix provides a history of modifications made to this reference manual
in a reverse chronological order, grouped by release. By referencing this
chapter when a new release arrives, you may quickly discern any documentation
changes which may or may not be of interest to you.

## Release 2022-10-0
* The [Single Payment Notes](module-singlepmt.md) module has been made available
  in this release.

## Release 2022-07-0
* The [APY Calculation](module-apy.md) module has been made available in this
  release.
* The [Certificates of Deposit](module-cd.md) module has been made available in
  this release.
* The [Individual Retirement Accounts](module-ira.md) module has been made
  available in this release.
* The [Retirement Annuities](module-annuity.md) module has been made
  available in this release.
* The [Loan](module-loan.md) `PmtStream` object has added flexibility to the
  interprertation of the `Date` field. For payment frequencies that are monthly
  multiples, the day number specified in the `Date` field specifies the desired
  day number of all future payments, if this day number does not exceed the
  number of days in the computed month. As an example, if the first payment is
  scheduled to be made in February of 2022 and the desired day for all payments
  is the 30th, then the field would be specified as `"Date" : "2022-02-30"`.
  The `Date` field of the `Fee` object behaves in the same manner when defining
  a stream of fees.


## Release 2022-04-0

* The [Loan](module-loan.md) module's `BusinessRules` request object supports a
  new `OddFirstPmt` field. The value of this field determines if the first
  payment should be adjusted for odd days using the same interest accrual
  calendar as in effect for the loan. Please see the documentation on this new
  field in the chapter covering the [Loan](module-loan.md) module for further
  information.
* The [High Cost Mortgages (HCM)](module-hcm.md) module has been made available
  in this release.
* The [Higher Priced Mortgage Loans (HPML)](module-hpml.md) module has been made
  available in this release.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Countries](appendix-countries.md) | [SCEJSON Reference Manual](README.md) | |
