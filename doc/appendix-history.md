# Appendix - History

> "History will be kind to me  
>  for I intend to write it."  
>  --- Winston Churchill

This appendix provides a history of modifications made to this reference manual
in a reverse chronological order, grouped by release. By referencing this
chapter when a new release arrives, you may quickly discern any documentation
changes which may or may not be of interest to you.

## Release 2025-01-0
* The [Loan](module-loan.md) module's `AccrualConfig.Date` and `Fee.Date`
  request field documentation was corrected when no `Date` field is provided. By
  default, if the `Date` field is not provided, the SCE will use the earliest
  `Date` field of all `Advance` objects specified in the `Advances[]` array.
* The [Loan](module-loan.md) module's `PmtStream.Amount` request field
  documentation has been updated to document the new options available for fixed
  payment streams.

## Release 2024-10-0
* The [Arm](module-arm.md) module's `Settings.TILARESPA2015` field can now be
  passed in as an object with two supported boolean child fields:
  `MinPnIDetails` and `MaxPnIDetails`. These two new fields allow the calling
  application to request detailed information about the `MinPnI` and `MaxPnI`
  payments that may be returned in the amortization schedule for ARM loans with
  TILARESPA2015 disclosures. If set to `true`, then the amounts to interest and
  principal for each of these payments may also be returned in the `AmLine`
  object. Note that previously, the `TILARESPA2015` field was boolean, and this
  is still supported. `"TILARESPA2015" : true` is now equivalent to
  `"TILARESPA2015" : {}`.

## Release 2024-07-0
* The [Loan](module-loan.md) module's `PmtStream` object now supports the new
  `ReplaceIdx` field which is used to apply a replacement payment stream to a
  subset of the payments. Please see the documentation for this new attribute in
  the Loan chapter.

## Release 2024-04-0
* The [Loan](module-loan.md) module's `Advance` request object now supports
  `NewPmt` and `Position` fields which are used when the calling application
  wishes to compute a loan with multiple advances, where each advance generates
  a new payment stream.
* The [Loan](module-loan.md) module now supports a new `BalAdjs` array field
  which allows the calling application to make one or more balance adjustments
  during the amortization of a loan to better support the quotation and
  servicing of open ended lending.

## Release 2024-01-0
* No changes made to the reference manual.

## Release 2023-10-0
* The `ODI.AddToPmt` field is not supported with pure [Construction
  Loans](module-construction.md) and the documentation for this field has been
  removed from the chapter covering pure construction loans.
* The `ODI` request object recognized by many loan calculation modules supports
  a new `NoCap` field. If an odd days interest fee is present and financed, the
  value of this attribute determines if the ODI fee is added to the principal
  balance for the purposes of computing the ODI fee (e.g. if it is capitalized).
  Please see the documentatio for this new field in the `ODI` object
  documentation. 
* The documentation for the `CD_TotPmts` field of the `TILARESPA2015` response
  object has been updated to include the value of any odd days interest that is
  prepaid at loan closing.

## Release 2023-07-0
* The documentation for the [Loan](module-loan.md) module's `Protection.Formula`
  field has been updated to include a table of all possible formula codes and a
  description of each.
* The documentation for the [Loan](module-loan.md) module's `Protection.DropCode`
  and `Protection.DropReason` fields has been updated to include a table of all
  possible drop codes and reasons.
* The `Format` request object recognized by many modules supports a new
  `StrictDP` field. If the value of this field is `true`, it informs the SCE to
  strictly verify the number of decimal places allowed for currency input
  values. Thus, if the calling application sends in a loan with a proceeds
  amount of 1000.005, the SCE will return an error code. If the value of this
  attribute is `false`, then currency values sent in with an invalid number of
  decimal places will be rounded to the correct number of decimal places by the
  SCE (using five/four rounding), and a warning message with this information
  will be returned with the response.
* The [Loan](module-loan.md) module's `APR` request object now supports a `Max`
  field which allows the calling application to specify a maximum APR. If a
  maximum is specified, then this maximum will be included in the `APR` response
  object in the `Max` field, and a `MaxExceeded` field will also be returned to
  inform the calling application whether or not the specified maximum was
  exceeded.
* In the [Loan](module-loan.md) module documentation, drop code 87 has been
  added to the ``Drop Codes and Reasons'' table.
* A new `EndBal` request field has been added to the [Loan](module-loan.md)
  module, which allows for loans to be computed with a specified ending balance
  greater than zero. Please see the documentation covering this new element in
  the for more information.

## Release 2023-04-0
* The [Account](module-account.md) module has been made available in this
  release. This module allows the calling application to query the account
  numbers and titles of all accounts defined in the `clients.set` file. If your
  solution does require setup file configuration services provided by J. L.
  Sherman and Associates, Inc., then this module will be of minimal use to you.
* The [User Interface](module-ui.md) module has been made available in this
  release. This module allows the user to determine how an *account* has been
  set up for the SCE in a given directory path, and indicates which protection
  products are available and what options are available for each active product.
  If your solution does require setup file configuration services provided by J.
  L. Sherman and Associates, Inc., then this module will be of minimal use to
  you.
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
* Added `APR.Code` `0` to the Loan module. APR Code 0 tells the SCE that no
  effective rate should be computed or returned in the `APR` object, so a value
  of `NC` will be returned for `APR.Value`  with the `APR.Type` set to
  `NotComputed`.  Please see the documentation for the `Code` field of the `APR`
  object in the [Loan](module-loan.md) module documentation for further
  information.

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
