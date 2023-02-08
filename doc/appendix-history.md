# Appendix - History

> "History will be kind to me  
>  for I intend to write it."  
>  --- Winston Churchill

This appendix provides a history of modifications made to this reference manual
in a reverse chronological order, grouped by release. By referencing this
chapter when a new release arrives, you may quickly discern any documentation
changes which may or may not be of interest to you.

## Release 2023-04-0
* The [Account](module-account.md) module has been made available in this
  release. This module allows the calling application to query the account
  numbers and titles of all accounts defined in the `clients.set` file. If your
  solution does require setup file configuration services provided by J. L.
  Sherman and Associates, Inc., then this module will be of minimal use to you.
* The `Format` request object has been added to the [High Cost Mortgages
  (HCM)](module-hcm.md), [Higher Priced Mortgage Loans (HPML)](module-hpml.md),
  and all loan calculation modules. Please see the documentation in the chapters
  covering the modules for further information.
* With the inclusion of the new `Format` request object in the
  [Loan](module-loan.md) module, the `BusinessRules.CurrencyDP` field has been
  deprecated and the documentation for this field has been removed from this
  reference manual.
* The default value of the `ODI.ForceUnitPeriod` field has been changed from
  `false` to `true` in all loan calculation modules that support the `ODI`
  request object.

## Release 2023-01-0
* The [Adjustable Rate Mortgages](module-arm.md) module has been made available
  in this release.
* The [Loan](module-loan.md) module's `BusinessRules` request object supports
  two new fields: `MinIntChg` and `MinFinChg`. These fields allow the calling
  application to specify minimum interest and finance charge values for a given
  loan. If adjustments are made to the final payment to satisfy the specified
  minimum interest and finance charge values, then new `MintIntChgAdj` and
  `MinFinChgAdj` fields will be present in the `Moneys` response object. Please
  see the documentation for these new fields in the chapter covering the
  [Loan](module-loan.md) module for further information.
* All specific loan calculation modules (i.e. [Equal Payment
  Loans](module-equalpmt.md), [Balloon Payment Loans](module-balloon.md), etc.)
  now support either a minimum interest charge *or* a minimum finance charge
  through the setup files. This minimum defined in the setup files is then
  active for all specific loan calculation moudle requests using that account.
  If adjustments are made to the final payment to satisfy the specified minimum
  interest and finance charge values, then new `MintIntChgAdj` and
  `MinFinChgAdj` fields will be present in the `Moneys` response object. Please
  see the chapter covering the appropriate specific loan calculation module for
  further information.
* All loan calculation modules now support financed odd days prepaid interest
  via the `AddToPrin` field of the appropriate object. Note that this field will
  be ignored if the odd days interest is set up to be added to the first payment
  instead of being disclosed as a prepaid fee.
* The `OddDaysPrepaid` request object from the [Loan](module-loan.md) module has
  been added to all of the other loan modules. This provides a consistent way to
  specify odd days interest fees in all loan modules. The old way of specifying
  an odd days prepaid fee (via the `Fees[]` array) has been deprecated, and the
  associated documentation has been removed. However, for backwards
  compatability purposes the functionality will remain within the SCE. Please
  see the appropriate loan chapter for further information.
* The [Loan](module-loan.md) module now supports the calculation of fixed principal
  plus interest and pure principal payments where the principal reduction amount
  is defined as a percentage of the computed payment. This allows a calling
  application to request a principal reduction amount equal to the computed target
  payment (e.g. `"Amount" : "100%C"`). Please see the documentation for the
  `PmtStream` object's `Amount` field in the Loan chapter for further information.
* The [Loan](module-loan.md) module now allows `PmtStream` and `Fee` dates to
  skip weekends, with options to move the date to the previous, next, or nearest
  business day. Please see the documentation for the new `Weekends` field in the
  chapter covering the [Loan](module-loan.md) module for further information.
* The [Loan](module-loan.md) module now allows `PmtStream` and `Fee` dates to
  skip holidays, with options to move the date to the previous or next day.
  Please see the documentation for the new `Holidays[]` array and `Holidays`
  field of the `PmtStream` and `Fee` request objects in the chapter covering the
  [Loan](module-loan.md) module for further information.
* The [Loan](module-loan.md) module now supports the new `Capitalization` array
  of `Capitalize` events, each of which allows the calling application to
  specify a stream of interest accrual capitalization events during the term of
  the loan. This is most useful in loans which feature pure principal payments
  and which capitalize interest regurlarly on a different day of the month.
  Please see the documentation for the new `Capitalization[]` array field in the
  chapter covering the [Loan](module-loan.md) module for further information.
* The `Format` request object has been added to the
  [Annuity](module-annuity.md), [Apy](module-apy.md), [Cd](module-cd), and
  [Ira](module-ira.md) modules. Please see the documentation in the chapters
  covering the modules for further information.

## Release 2022-10-0
* The [Balloon Payment Loans](module-construction.md) module has been made
  available in this release.
* The [Construction Loans](module-construction.md) module has been made
  available in this release.
* The [Equal Payment Loans](module-construction.md) module has been made
  available in this release.
* The [Fixed Principal Plus Interest Loans](module-principalplus.md) module has
  been made available in this release.
* The [Interest Only Loans](module-interestonly.md) module has been made
  available in this release.
* The [Single Payment Notes](module-singlepmt.md) module has been made available
  in this release.
* The [Skip, Pickup and Irregular Payment Loans](module-irregular.md) module has
  been made available in this release.

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
