# Introduction

> "There is nothing more difficult to take in hand,  
>  more perilous to conduct, or more uncertain  
>  in its success, than to take the lead in the  
>  introduction of a new order of things."  
>                           --- Niccolo Machiavelli

The development and consulting team here at [J. L. Sherman and Associates](https://www.shermanloan.com/company/) (JLS) has accumulated over fifty man-years of experience creating a comprehensive array of programs providing complex, accurate, and compliant loan calculations for our clients. All of this experience and expertise goes into our Sherman Calculation Engine (SCE), which is the most robust loan calculation and compliance library available on the market today.

## The Sherman Calculation Engine

Prior to the development of the [WinLoan32](https://www.shermanloan.com/products/winloan), our desktop based loan quotation software for the 32-bit Windows platform, several of our customers expressed an interest in having access to the calculation routines contained in the WinLoan16. Due to previous design decisions, this was not possible. With Winloan32, however, the powerful calculation engine that is the foundation of our end user loan quotation software (both WinLoan32 and [eWinLoan](https://www.shermanloan.com/products/ewinloan)) was developed as a separate product, which we call it the Sherman Calculation Engine (SCE).

The SCE was released as a product in the form of a 32-bit dynamic link library (DLL) for the Windows environment. Loan and insurance setup information is stored in text files that are created and maintained by Sherman's technical support team. Depending upon the level of support you desire, we can handle all of the steps in setting up a new account for you. This process involves gathering samples from the client and payment protection provider, and thoroughly testing to ensure that the final calculation results match what the client is expecting.

By separating the calculation logic from the user interface, we make available to our partners all of our applied knowledge and expertise. However, our new partners encountered three problems as they worked to interface directly with the SCE:

1. Initially, the user of the SCE needed to implement all of the data structures used by the SCE for input and output, as well as create the interface to the functions found in the DLL.

2. As the SCE was modified to suit customer needs and federal or state regulations additions and changes, these data structures needed to be altered. This required the data structures in our partner's applications to also be altered, and then the applications which call the SCE must be recompiled.

3. Some of our partners wanted to access the power and functionality of the SCE from applications *not* written for the 32-bit Windows platform. There is no simple way to access a 32-bit Windows DLL from another platform.

These three problems prompted Sherman and Associates to begin development on two new satellite projects of the SCE: the SCE with XML interface (hereafter called the [SCEX](https://www.shermanloan.com/products/scex)) and the SCEX Loan Server.

## Introducing the SCEX

The SCEX addresses problems #1 and #2 in the list above. It provides an XML interface wrapped around the SCE. By using the SCEX instead of the SCE, the calling application no longer needs to duplicate the data structures used by the SCE. Instead, all input sent to the SCEX is contained in one character buffer (or character string). Similarly,
the SCEX returns all results in an output character buffer.

The format of the text contained in these character buffers is what the `X' in the SCEX is all about - XML. XML is a simple, flexible text format which is plays an important role in the exchange of data between disparate platforms and systems.

The SCEX alleviates problem \#1 by doing away with complex input and output data structures all together. Instead, the application calling the SCEX provides the necessary input in a character buffer, which contains the XML data. As an example, here is the XML input data for a 36 month equal payment loan with a 10% interest rate and an amount requested of $10,000:

```xml
<inLOAN_BUILDER>
  <EditInterest Date="2019-01-01" IntRate="10.000" AccrualCode="320" />
  <Advance Date="2019-01-01" Amount="10000.00" />
  <PmtStream Begin="2019-02-01" Term="36" />
</inLOAN_BUILDER>
```

The result of the loan calculation is then passed back to the calling application in a buffer. The application then need only parse the returned XML to use the calculation results as appropriate. Here is a sample result returned from the SCEX, using the inputs specified above (with the amortization table omitted for brevity's sake):

```xml
<?xml version="1.0" standalone="no" ?>
<!DOCTYPE outLOAN_BUILDER SYSTEM "outLOAN_BUILDER.dtd">
<outLOAN_BUILDER>
   <Results>
      <Description>Successful Calculation</Description>
      <XMLDetail>XML Input is well formed</XMLDetail>
      <XMLDetail>XML Output is untested</XMLDetail>
   </Results>
   <FedBox>
      <AmtFin>10000.00</AmtFin>
      <FinChg>1615.40</FinChg>
      <TotPmts>11615.40</TotPmts>
      <RegZAPR Type="Actuarial">9.995</RegZAPR>
   </FedBox>
   <Moneys>
      <Proceeds>10000.00</Proceeds>
      <Principal>10000.00</Principal>
      <Interest>1615.40</Interest>
   </Moneys>
   <Accrual>
      <Method>Actual/365 USRule</Method>
      <Days1Pmt DayCount="Actual">31</Days1Pmt>
      <Maturity>2022-01-01</Maturity>
   </Accrual>
   <PmtStream Term="36" Pmt="322.65" Rate="10.000" Begin="2019-02-01" PPY="12"/>
   <AmTable> ... </AmTable>
</outLOAN_BUILDER>
```

Furthermore, problem #2 is also handled. If a new input or output data field is added to the SCE, then the appropriate XML input / output tags will be added to the SCEX. Applications which do not require this new functionality will *not* need to be recompiled. Instead, they will simply not provide the new input field, or not recognize the new output field. So long as your application does not require the use of the new data zfields, *then no recompilation is necessary*!

Problem #3 was solved with the introduction of the SCEX Loan Server. The SCEX Loan Server goes one step beyond the SCEX, turning the SCEX into a multi-platform calculation engine. The SCEX Loan Server provides a TCP/IP communication wrapper around the SCEX. So long as the platform on which your application resides can communicate via TCP/IP, you can use the Sherman Calculation Engine in your product.

## The Rise of the API and SCEJSON

With the advent of cloud computing and the rise of API driven development, a new light weight and flexible data-interchange format arrived and was quickly adopted - [JSON](https://www.json.org/). Learning from our 18+ years of creating, updating, and maintaining the SCEX, we decided to launch the SCE with JSON interface (hereafter called the SCEJSON).

The SCEJSON uses the same Sherman Calculation Engine behind a new JSON interface. The sample XML loan request above translates to the following SCEJSON loan request:

```json
{
  "Module" : "Loan",
  "Data" : {
    "Advances" : [
      {
        "Date" : "2019-01-01",
        "Amount" : "10000.00"
      }
    ],
    "AccrualConfigs" : [
      {
        "Date" : "2019-01-01",
        "IntRate" : "10.000",
        "AccrualCode" : "320"
      }
    ],
    "PmtStreams" : [
      {
        "Date" : "2019-02-01",
        "Term" : "36"
      }
    ]
  }
}
```

Here is the sample result returned from the SCEJSON, using the inputs specified above (with the amortization table omitted for brevity's sake):

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
      "AmtFin" : "10000.00",
      "FinChg" : "1615.40",
      "TotPmts" : "11615.40",
      "APR" : {
        "Value" : "9.995",
        "Type" : "Actuarial"
      }
    },
    "Moneys" : {
      "Proceeds" : "10000.00",
      "Principal" : "10000.00",
      "Interest" : "1615.40"
    },
    "Accrual" : {
      "Methods" : [
        "Actual/365 USRule"
      ],
      "Days1Pmt" : "31",
      "DayCount" : "Actual",
      "Maturity" : "2022-01-01"
    },
    "PmtStreams" : [
      {
        "Term" : "36",
        "Pmt" : "322.65",
        "Rate" : "10.000",
        "Begin" : "2019-02-01",
        "PPY" : "12"
      }
    ],
    "AmTable" : { ... }
  }
}
```

## Hosted vs. Docker Implementations

The SCEJSON may be implemented in the following manners:

1. ![Hosted](https://img.shields.io/badge/Hosted-orange?style=flat&logo=amazon-aws) - The SCEJSON is hosted by JLS on [AWS](https://aws.amazon.com/) for you. You are provided with a URL and API key that will allow you to access the functionality of the SCEJSON, without having to setup and maintain your own microservice. As new releases of the SCEJSON are made available, you will be notified of these changes and we will upgrade the Hosted version at your request. The only requirement for the application calling the hosted instance of the SCEJSON is being able to execute a network API request to the provided URL, and to be able to process the data returned in the JSON format.

2. ![Docker](https://img.shields.io/badge/Docker-green?style=flat&logo=docker) - The SCE API server Docker image is availabe from [Docker Hub](https://hub.docker.com/repository/docker/shermanloan/sce-api-server), ready to be deployed in [Docker](https://www.docker.com/). You are provided with an API key that will allow you to access the functionality of the SCE, which will need to be updated periodically. As new releases of the SCE are made available, you will be notified of the changes and will be able to download the updated versions as they become generally available. The SCE API server Docker container can be hosted on Linux/amd64 and Linux/aarch64 systems which support Docker.

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [SCEJSON Reference Manual](README.md) | [SCEJSON Reference Manual](README.md) | [API Overview](api-overview.md) |
