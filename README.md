# CBS #
INTRODUCTION\
The objective of this document is to define the platform for interaction between the 
Central Billing System (CBS) web service platform and third-party platforms 
for the purpose of carrying out specific processes. The communication mode is via REST 
web service and is accessible via the URL (to be provided)


## Create Invoice ##
This web service call will be used by the integrating system to create an invoice in CBS;

* URL\
https://URL/api/v1/invoice/create

* Method\
`POST`

## Header Parameters ##
* Client ID – to be provided
* Client Secret – to be provided

Property      | Description                                            | Data Format   | Mandatory   |
------------- | -------------------------------------------------------| ------------- | ------------|   
Signature     | HMACSHA256 hash of the concatenation of RevenueHeadID <br/> Amount (2 decimal places), CallBackURL, ClientID | `string` | Y |
Client ID     | Identifier of the integrating system on CBS (To be provided) | `string` | Y |

_Note:_  The signature concatenation is hashed with the client secret 

### Input Parameters [content-type - JSON] ###
Property      | Description                                            | Data Format   | Mandatory (Y/N)   |
------------- | -------------------------------------------------------| ------------- | ------------|   
RevenueHeadId | Identifier of the revenue head you want to generate an invoice for | `Numeric` | Y |
RequestReference | When passed, it must be unique per request. It must also be <br/> deterministic for the calling application to generate i.e. for the same request this value must always be the same | `string` | Y |
CallBackURL | API URL you want payment notifications to be sent to | `string` | N |
ExternalRefNumber | Third party reference number | `string` | N |
TaxEntityInvoice     | Tax entity invoice object                                       | [object](#taxentityinvoice)                | Y           |

<div id="taxentityinvoice"></div>

### TaxEntityInvoice Object Property ###
Property      | Description                                             | Data Format                         | Mandatory   |
------------- | --------------------------------------------------------| ------------------------------------| ------------|   
TaxEntity     | Tax entity object                                       | [object](#taxentity)                | Y           |
Amount        | Amount to be generated. Some revenue heads allow tax payers to <br/> input the amount to be paid for, while some have a fixed amount. For cases where an mount is required, this value must be set to be greater than 0.00. for cases where the amount is fixed, if an amount is supplied, the supplied amount is used for invoice generation                                                                         | `numeric`                           | Y           |
InvoiceDescription        | Invoice Description                         | `string`                            | N           |
CategoryId                | Category Id                                 | `numeric`                           | Y           |

<div id="taxentity"></div>

### TaxEntity Object Property ###
Property      | Description                                            | Data Format   | Mandatory   |
------------- | -------------------------------------------------------| ------------- | ------------|   
Recipient     | Name of the invoice recipient | `string` | Y |
Email         | Email address of the recipient   | `string` | Y |
Address         | Address of the recipient   | `string` | Y |
PhoneNumber         | Phone number of the recipient   | `string` | N |
TaxPayerIdentificationNumber         | TIN   | `string` | N |
RCNumber         | RC Number   | `string` | Y |
PayerId         | Payer Id   | `string` | Y |

### Sample Input parameter JSON payload ### 
```json
{
 "RevenueHeadId": 16,
 "TaxEntityInvoice": {
 "TaxEntity": {
 "Recipient": "Tax Payer",
 "Email": "taxpayer@example.com",
 "Address": "API local",
 "PhoneNumber": "0804832361",
 "TaxPayerIdentificationNumber": "7777711",
 "RCNumber": null,
 "PayerId": null
 },
 "Amount": 8903,
 "InvoiceDescription": "Invoice Description here",
 "CategoryId": 1
 },
 "ExternalRefNumber": null,
 "RequestReference": "Unique ref per request, must be deterministic",
 "CallBackURL": "add a call back URL if payment notification is required"
}
```
### Output Parameters [content-type - JSON] ###
For every API call a standard response object is returned. If there was anything wrong with the request, the Error value 
would be true.
If in the request a PayerId was not specified a new PayerId is generated, if a PayerId was specified an invoice is 
generated against the given PayerId

Property      | Description                                            | Data Format   | Mandatory (Y/N)   |
------------- | -------------------------------------------------------| ------------- | ------------|   
Error | Indicates if request was processed successfully. If this request was processed successfully,the ResponseObject would contain an object detailing the invoice            info. If the error value is true, the ResponseObject would contain a list of ErrorModel | `Bool` | Y |
ResponseObject |This value would either contain the ErrorModel if error is true or the invoice details if Error is false | `object` [true](#trueobjectresponse) or [false](#falseobjectresponse)     | N |


<div id="falseobjectresponse"></div>

### ResponseObject Object Property (False) ###
Property      | Description                                            | Data Format   | Mandatory   |
------------- | -------------------------------------------------------| ------------- | ------------|   
InvoiceNumber     | Unique invoice number generated from CBS | `string` | Y |
InvoicePreviewUrl         | URL to preview the generated invoice   | `string` | Y |
AmountDue         | URL to preview the generated invoice  | `decimal` | Y |
CustomerId         | Customer Id for ref   | `long` | N |
Recipient         | Name of the Taxpayer   | `string` | Y |
PayerId         | Tax payer reference value on CBS. It is unique per tax payer   | `string` | Y |
Email         | Tax payer email value   | `string` | N |
PhoneNumber         | Tax payer phone number   | `string` | Y |
TIN         | Tax payer Identification Number    | `string` | N |
MDAName         | MDA Name   | `string` | Y |
RevenueHeadName         |Revenue Head Name   | `string` | Y |
ExternalRefNumber         | External Reference Number   | `string` | Y |
Description         | Invoice Description   | `string` | N |
RequestReference         | Reference Passed during request | `string` | Y |
IsDuplicateRequestReference  | If the request is a duplicate request (i.e. if multiple requests where sent with the same request reference),this value would be true                                         for subsequent request after the first one | `bool` | N |

### Sample Output parameter JSON payload (False) ### 
```json
{
 "Error": false,
 "ErrorCode": null,
 "ResponseObject": {
 "CustomerPrimaryContactId": 352613,
 "CustomerId": 332619,
 "Recipient": "Tax Payer",
 "PayerId": "HA-000034",
 "Email": "taxpayer@example.com",
 "PhoneNumber": "0804832361",
 "TIN": "7777711",
 "MDAName": "MDA TEST 2018 636788501722459205",
 "RevenueHeadName": "API SELF",
 "ExternalRefNumber": null,
 "PaymentURL": null,
 "Description": "API SELF | CODE TEST 2018 636788501722464167/SELF",
 "InvoiceNumber": "1000372608",
 "InvoicePreviewUrl": "http://127.0.0.1/CashFlow.API/v2/ViewInvoice/1000372608/Html",
 "AmountDue": 8903
 }
}
```

<div id="trueobjectresponse"></div>

### ResponseObject Array Object Property (True) ###
Property      | Description     | Data Format   | Mandatory   |
------------- | ----------------| ------------- | ------------|   
FieldName     | Field Name      | `string`      | Y           |
ErrorMessage  | Error Message   | `string`      | Y           |

### Sample Output parameter JSON payload (True)###

```json
{
 "Error": true,
 "ErrorCode": "PPVE",
 "ResponseObject": [
 {
 "FieldName": "Address",
 "ErrorMessage": "Address field is required"
 }
 ]
}
```

* Code Snippets
    * [C#](https://github.com/parkwayprojects/CBS-CSharp-Snippets "Csharp code snippets")
    * [PHP](https://github.com/parkwayprojects/CBS-CSharp-Snippets "Php code snippets")

