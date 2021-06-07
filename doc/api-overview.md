# API Overview

The SCEJSON offers different functionality in distinct *modules* which, are described below:

| Module | Description |
| ------ | ----------- |
| [Version](module-version.md) | A simple module which returns the version of the SCEJSON in use, along with a few other userful data fields. |
| [Loan](module-loan.md)    | The Loan module allows you to build almost any kind of loan you can imagine - one or more advances, adjustable interest rates, overlapping payment schedules, etc.  |
| [Apr](module-apr.md)     | Our APR calculation and verification module. Note that APRs are returned in the Loan module. However, this module is typically used to calculate and/or verify APRs for loan computed by another calculation methodology. |
| [Hcm](module-hcm.md)     | A compliance module which implements the logic necessary to determine if a specified mortgage is considered High Cost, per [§ 1026.32 Requirements for high-cost mortgages](https://www.consumerfinance.gov/rules-policy/regulations/1026/32/). |
| [Hpml](module-hpml.md)    | A compliance module which implements the logic necessary to determine if a specified mortgage is considered a Higher Priced Mortgage Loan, per [§ 1026.35 Requirements for higher-priced mortgage loans](https://www.consumerfinance.gov/rules-policy/regulations/1026/35/).|

Each of the modules listed above require different input requests and output responses, which are detailed in the chapters dedicated to each module.  However, the general (or top-level) format of the data sent to the SCEJSON needs to be consistent for correct parsing, and is called the "request envelope".

## The Request Envelope Object

The request envelope is simply a JSON object which identifies the requested `Module`, along with a `Data` object that contains the module specific data fields. The `Data` object for each module is thoroughly documented in the chapter dedicated to that module.

**Examples**

```json
{
  "Module" : "Version",
  "Data" : {...}
}
```
```json
{
  "Module" : "Loan",
  "Data" : {...}
}
```

## Envelope Object Field Definitions

**Module** - *JString* : {"Apr", "Hcm", "Hpml", "Loan", "Version"} (❗required ) 

The value of the `Module` field determines which module within the SCEJSON is being requested.

**Data** : *JObject* (❗required )

The `Data` field contains the module specific request data, and differs depending upon the desired `Module`. Please reference the chapter dedicated to the module for the required format of the `Data` object.


| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [Introduction](introduction.md) | [SCEJSON Reference Manual](README.md) | [API Overview](module-version.md) |
