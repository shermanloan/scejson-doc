# Appendix - Calculation Notes

> "We all do 'do, re, mi,' but you have  
>  got to find the other notes yourself."  
>                           --- Louis Armstrong

Some of the modules in the SCE may return calculation notes when certain
situations are encountered during the calculation. These are not errors,
but calculation notes which may help to explain something which might not
be expected by the end user.

## Calculation Notes

All of the calculation notes that may be returned by the SCE are documented
below. Each note is associated with a numeric code and text return value. Each
will be further described in the list below:

1. **"Zero Interest Rule in effect. If IntRate=0, round pmts down, set Interest
   to zero. Negative Interest always set to zero."**

   In the Loan module, there is an field of the `BusinessRules` object called
   `ZeroInterestRule`. If enabled and the interest rate is set to zero, then
   payments will *always* be rounded down, no matter what (if any) payment
   rounding method was specified in the input request. Furthermore, if enabled
   and the computed interest amount is less than zero, then the interest amount
   will be set to zero, even if the interest rate is greater than zero.

2. **"Oscillator detected and successfully resolved."**

   The SCE is an iterative calculation engine, and in certain circumstances,
   the computed principal balance (which is made up of the amount requested
   in addition to any amounts which are added to the principal balance) can
   oscillate back and forth between two values. When this situation occurs
   and the difference between both values is small enough, one of the two
   values will be used and the calculation will be completed.

3. **"Currently not used."**

   This note is currently not implemented.

4. **"The Interest Paid, `IntPaid`, of the final amortization event has been
   adjusted to make the `EndBal` zero."**

   When amortizing a loan, often times there is a non-zero ending balance due
   to rounding. The final interest reduction amount has been adjusted to
   achieve perfect amortization, which results in an ending balance of zero.
   Note that this will cause the final payment to not equal the sum of the
   principal reduction amount and amount to interest in the returned
   amortization schedule.

5. **"The Principal Reduction, `Prin`, of the final amortization
    event has been adjusted to make the `EndBal` zero."**

   When amortizing a loan, often times there is a non-zero ending balance due
   to rounding. The final principal reduction amount has been adjusted to
   achieve perfect amortization, which results in an ending balance of zero.
   Note that this will cause the final payment to not equal the sum of the
   principal reduction amount and amount to interest in the returned
   amortization schedule.

6. **"This loan has at least one irregular periods. IRR APR's may not accurately
   reflect the effective rate."**

   When using the IRR APR method, if there is not complete regularity in
   periods between events, the resulting calculated value may not reflect
   the true internal rate of return. Consider using the XIRR method when
   the periodicity is not regular.
   
7. **"The US Rule APR was not computed, but set to the entered rate."**

   As requested, the US Rule APR was not computed, but set equal to the
   entered rate.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Higher Priced Mortgage Loans (HPML)](module-hpml.md) | [SCEJSON Reference Manual](README.md) | [Countries](appendix-countries.md) |
