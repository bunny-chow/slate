---
title: Solidus Labs API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>Solidus Labs Support</a>

includes:

search: true

code_clipboard: true
---

# Overview

**Solidus was designed to make the lives of digital asset compliance teams easier.**

The Solidus Cloud is a crypto-native compliance hub, market surveillance and transaction monitoring solution designed to help compliance teams detect and investigate threats, monitor risk holistically and generate regulatory and internal reports - all in one place.

The Solidus Cloud ingests and processes client data to detect manipulation patterns, generates alerts then presents all findings in a unified risk monitoring dashboard.  In order to streamline the compliance workflow, Solidus was designed for utmost connectivity and includes simple interfaces  for clients to provide  trading and transaction data.  

This manual covers specifications for each supported data transfer method in detail, alongside examples, as well as API connectivity guidelines, required data formats and other frequently asked questions. For additional questions or issues not covered, please contact us using the form below.

# Data Requirements


## Order Data Fields

<aside class="notice">
Field names marked with a * are mandatory fields
</aside>

Field Name | Type | Description | Valid Values
--------- | ------- | ----------- | -----------
*`TransactTime` | `timestamp` | Time in UTC when the trading instruction (i.e. new order, order cancel etc.) was created in the trading system, or when a fill happened | 
*`Id` | `String` | Order ID assigned by buy-side system is carried as a value of tag 11 in trading messages | 
`Version` | `int` | Version of the order as the state of the order change with different transactions |
*`Symbol` | `String` | Pair name to trade. Ex: BTC/USD | `<base/counter>`
*`Side` | `String` | Order ID assigned by buy-side system is carried as a value of tag 11 in trading messages | `Buy`<br> `Sell`
*`OrderQty` | `String` | Number of units | 
*`Price` | `String` | Price for Limit orders | 
`PriceType` | `String` | Order ID assigned by buy-side system is carried as a value of tag 11 in trading messages | 1 = Percentage (e.g. percent of par) (often called "dollar price" for fixed income)<br> 2 = PerUnit (i.e. per share or contract)<br> 3 = FixedAmount (absolute value)
*`OrdType` | `String` | Order Type | `Market`<br> `Limit`<br> `Stoploss`
*`OrdStatus` | `String` | Identifies current status of order, note that:<br> 0=Accepted <br> D=New | `Accepted`<br>`undefined`<br>`Filled`<br>`DoneForDay`<br>`Canceled`<br>`Replaced`<br>`PendingCancel`<br>`Stopped`<br>`Rejected`<br>`Suspended`<br>`PendingNew`<br>`Calculated`<br>`Expired`<br>`PendingReplace`<br><br>`New`<br>`OrderCancelRequest`<br>`OrderReplaceRequest`
`ExVenue`| `String` | Venue name which provided the execution |
`AvgPx`| `String` | Broker calculated average price of all fills/executions on the current order. Will be available on part filled/filled orders |
*`CumQty`| `String` | Total quantity filled |
`LeavesQty`| `String` | Remaining, unfilled/uncanceled quantity on the order |
`TimeInForce`| `String` | Specifies how long the order remains in effect | `Day`<br>`GoodTillCancel`<br>`AtTheOpening`<br>`ImmediateOrCancel`<br>`FillOrKill`<br>`GoodTillCrossing`<br>`GoodTillDat`e<br>`AtTheClose`
`ExpireDateTime`| `timestamp` | Date of order expiration |
*`OrderCapacity`| `String` | Designates the capacity of the firm placing the order |`Agency`<br>`Proprietary`<br>`Principal`
*`Account`| `String` | Field used for account mnemonic. Its identification source is mutually agreed between the counterparties |
*`ClientId`| `String` | Unique identifier of the client/beneficiary of the order. Represented in fix as part of PartyRole <452> and PartyID <448> |`Client ID`<Br>`<Client ID>` 
`OriginationFirm`| `String` | Buyside trader id associated with Order Origination Firm which originates/submits the order. | Order Orig. Firm<br>`<Orig. Firm>`
*`OriginationTrader` | `String` | Buyside firm associated with Order Origination Firm which originates/submits the order. |
`DeskId`| `String` | Desk Id of the transaction creator |
`OrigClOrdID`| `String` | Id (such as tag 11) of the previous order (NOT the initial order of the day) as assigned by the institution, used to identify the previous order in cancel and cancel/replace requests |
`Text`| `String` | Free format text String |
*`LastLiquidityInd` | `String` | Indicator to identify whether this fill was a result of a liquidity provider providing or liquidity `taker taking the liquidity` | `AddedLiquidit`<br>`RemovedLiquidity`<br>`LiquidityRoutedOut`

## Execution Or Trade Data Fields

Field Name | Type | Description | Valid Values
--------- | ------- | ----------- | -----------
*`TransactTime` | `timestamp` | Time in UTC when the trading system, or when a fill happened | 
*`Id` | `String` | Unique identifier of execution message as assigned by sell-side (broker, exchange, ECN) |
`Version`| `int` | Version of the order as the state of the order change with different transactions |
*`OrderID` | `String` | Order Id associated with the execution/trade |
*`MatchingOrderID` | `String` | Order ID of the contra/opposite side order that matched with the order in focus |
*`MatchingOrderIDVersion` | `String` | Version of the MatchingOrder associated with the MatchingOrder Execution |
*`Quantity` | `String` | Quantity executed/filled |
*`Price` | `String` | Execution price |
*`Status` | `String` | Identifies current status of execution | `Accepted` <br>`Canceled`
`LastMkt` | `String` | Venue name which provided the execution |


# Data Transfer Modes

### Getting Started
Solidus Labs supports two alerting modes to support client requirements: Batch and Real Time alerting. Data integration implementation depends on the alerting mode chosen by the client. Solidus Labs systems support data integration via REST API or Flat Files. REST API is typically chosen for real time alerting mode while flat files integration is implemented for batch alerting mode.

### REST API
REST API us used when alerting requirements are real time or near real time.
Data is sent to Solidus Labs systems via an API call as the events happen on client side.
Solidus Labs API interface listens for the incoming data, processes it to generate alerts for the Dashboard users.
### Flat Files
Flat files are used when alerting requirements are batch mode.
Data is collected and formatted on client side for a specific amount of time and sent over as a flat file to Solidus Labs.
Solidus Labs systems watch for the file from client, process the data and generate alerts for the Dashboard users.
### FIX API
Setup a FIX engine of your side with SenderCompID and TargetCompID as agreed with Solidus Team.
Convert order and execution data in FIX 4.4 format and send it through the FIX engine.

# Rest API


## Exchanges

> REST API Call Example for Exchanges:

```shell
# New Order

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "0",
 "ClientId": "999010573",
 "OriginationFirm": "401",
 "OriginationTrader": "999010573",
 "CumQty": "0",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "5",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "New",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Partial Fill

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "8556.94",
 "ClientId": "999010573",
 "OriginationFirm": "401",
 "OriginationTrader": "999010573",
 "CumQty": "1",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "4",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "PartiallyFilled",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Cancel

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "8556.94",
 "ClientId": "999010573",
 "OriginationFirm": "401",
 "OriginationTrader": "999010573",
 "CumQty": "1",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "4",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "Canceled",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Execution

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Id": "94945",
 "LastLiquidityInd": "RemovedLiquidity",
 "LastMkt": "GS",
 "MatchingOrderID": "28561690",
 "OrderID": "28561684",
 "Price": "8556.94",
 "Quantity": "1",
 "Status": "New",
 "TimeStamp": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/tradeevents
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```


Credentials are required to authenticate before you can send messages using the REST API

1. Speak with your Solidus representative to receive an credentials for real time data transfer account
2. Authenticate user to receive a security token
3. Use a security token with each REST call. All requests will be directed to HTTPS endpoint

Quick Notes:

* All requests must be sent to https://auth.region.environment.soliduslabs.com/api/v1
* Non-HTTPS requests will be redirected to HTTPS, possibly causing functional or performance issues with your application
* All REST requests will result in a 200 Ok response unless there is a server or infrastructure error. The API result will be wrapped in a JSON Result object.
* 400 BAD_REQUEST - INVALID_FORMAT (with explanation: item number and relevant fields)
* 400 BAD_REQUEST - INVALID_VALUES (with explanation: for example: timestamp exists in the future)
* 403 FORBIDDEN - user does not have permission to manage rest ingestion
* 409 CONFLICT - transaction id already exists in the system

## Broker Dealers, Market Makers, Hedge Funds

> REST API Call Example for Broker Dealers, Market Makers, Hedge Funds:


```shell
# New Order

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "0",
 "ClientId": "999010573",
 "ContraBroker": "401",
 "ContraTrader": "999010573",
 "CumQty": "0",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "5",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "New",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Partial Fill

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "0",
 "ClientId": "999010573",
 "ContraBroker": "401",
 "ContraTrader": "999010573",
 "CumQty": "0",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "5",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "New",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Cancel

curl -X POST
--header 'Accept: application/json'
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{
 "Account": "cd64368c-3283-40c0-bccb-585b779bba5a",
 "AvgPx": "0",
 "ClientId": "999010573",
 "ContraBroker": "401",
 "ContraTrader": "999010573",
 "CumQty": "0",
 "DeskId": "D19735",
 "ExpireDateTime": "20190602-00:00:00",
 "Id": "28561684",
 "LeavesQty": "5",
 "OrderCapacity": "Proprietary",
 "OrderQty": "5",
 "OrdStatus": "New",
 "OrdType": "Market",
 "OrigClOrdID": "28561684",
 "Price": "8556.94",
 "PriceType": "Price",
 "Side": "Sell",
 "Symbol": "BTC/USD",
 "Text": "Text",
 "TimeInForce": "Day",
 "TransactTime": "20190601-00:00:01.427618005",
 "Version": 1
}' https://auth.region.environment.soliduslabs.com/api/v1/orderevents
```
```shell
# Execution

curl -X POST 
--header 'Accept: application/json' 
--header 'X-Auth-Token: YourTokenHere' -H 'Content-Type: application/json' -d '{  
  "Id": "94945", 
  "LastLiquidityInd": "RemovedLiquidity", 
  "LastMkt": "GS", 
  "MatchingOrderID": "28561690", 
  "OrderID": "28561684", 
  "Price": "8556.94", 
  "Quantity": "1", 
  "Status": "New", 
  "TimeStamp": "20190601-00:00:01.427618005", 
  "Version": 1
  }' https://api-developers.soliduslabs.com/api/v1/tradeevents
```

# Flat Files

> Examples of flat files for exchanges

```shell

#New Order

TransactTime,Id,Version,Symbol,Side,OrderQty,Price,PriceType,OrdType,OrdStatus,AvgPx,CumQty,LeavesQty,TimeInForce,ExpireDateTime,OrderCapacity,Account,ClientId,OriginationFirm,OriginationTrader,DeskId,OrigClOrdID,Text
1600361061141,28561684,1,BTC/USD,Sell,2.1,8556.94,PerUnit,Limit,New,0,0,2.1,GoodTillCancel,,Agency,585b779bba5a,999010573,401,999010573,D19735,28561684,
```

```shell

#Partial Fill

TransactTime,Id,Version,Symbol,Side,OrderQty,Price,PriceType,OrdType,OrdStatus,AvgPx,CumQty,LeavesQty,TimeInForce,ExpireDateTime,OrderCapacity,Account,ClientId,OriginationFirm,OriginationTrader,DeskId,OrigClOrdID,Text

1600361071141,28561684,2,BTC/USD,Sell,2.1,8556.94,PerUnit,Limit,PartiallyFilled,8556.94,1.5,0.6,GoodTillCancel,,Agency,585b779bba5a,999010573,401,999010573,D19735,28561684,
```
```shell

#Cancel

TransactTime,Id,Version,Symbol,Side,OrderQty,Price,PriceType,OrdType,OrdStatus,AvgPx,CumQty,LeavesQty,TimeInForce,ExpireDateTime,OrderCapacity,Account,ClientId,OriginationFirm,OriginationTrader,DeskId,OrigClOrdID,Text
‍
1600361081141,28561684,3,BTC/USD,Sell,2.1,8556.94,PerUnit,Limit,Canceled,8556.94,1.5,0,GoodTillCancel,,Agency,585b779bba5a,999010573,401,999010573,D19735,28561684,
```
```shell

#Execution Record

TransactTime,Id,Version,OrderID,MatchingOrderID,Quantity,Price,Status,LastMkt,LastLiquidityInd
1600361071141,94945,1,28561684,28561690,1.5,8556.94,Accepted,TestExchange,RemovedLiquidity
```








Sending flat files is a way to get started quickly with data integration with Solidus Labs.  Order and execution data is collected and formatted in the specific schema. All order events related to trading activity are collected for market surveillance algorithms.

File Structure:
Uploaded data-files should include the following elements:

### Header Row

Like most CSV files, the first header should include field names according to the sent schema:

Sending flat files is a way to get started quickly with data integration with Solidus Labs.  Order and execution data is collected and formatted in the specific schema. All order events related to trading activity are collected for market surveillance algorithms.

**Order header row:**

`TransactTime,Id,Version,Symbol,Side,OrderQty,Price,PriceType,OrdType,OrdStatus,AvgPx,CumQty,LeavesQty,TimeInForce,ExpireDateTime,OrderCapacity,Account,ClientId,OriginationFirm,OriginationTrader,DeskId,OrigClOrdID,Text`

**Execution header row:**

`TransactTime,Id,Version,OrderID,MatchingOrderID,Quantity,Price,Status,LastMkt,LastLiquidityInd`

### Data Tuples

This is where the actual data comes in, in comma-separated values.

Example of an order:

`TransactTime,Id,Version,OrderID,MatchingOrderID,Quantity,Price,Status,LastMkt,LastLiquidityInd`

### How to uplaod files:

‍In order to upload a file, you will need to follow these steps:

* Speak to your Solidus Labs representative in order to get the credentials required to use our system.
* Login to our system in order to receive a temporary security-token which will enable you to use our API. Login can be achieved using cURL:


    `curl --request POST 'https://auth.region.environment.soliduslabs.com/api/v1/auth/login' --header 'Content-Type: application/json' --header 'Content-Type: text/plain' --data-raw '{"email": "your@email.here", "password": "yourPasswordHere" }' `


* Use our API to issue a secure link that will allow you to upload your data-file to an internal folder within our system. From there, our processes will periodically scan your workspace in search of new files to process:

    `curl --request POST  'https://auth.region.environment.soliduslabs.com/api/v1/issue-upload-url' \ --header 'Content-Type: application/json' \ --header 'X-Auth-Token: your-issued-security-token' \ --header 'Content-Type: application/json' \ --data-raw '{ "fileName": "your_filename.csv", "fileType": "private_order" }'`

    Possible “fileTypes” are:<br>`private_order`<br>`Private_execution`
‍
* Last stretch - we can now use the link that we received in order to upload our file to Solidus Labs.
Please refrain from accidentally uploading different files using the same link, as the link is tied to a specific file-path, and uploading different files from it will only serve to overwrite your files.

    `curl --request PUT 'your-issued-url-here' \ --header 'Content-Type: application/x-www-form-urlencoded' \ --header 'Content-Type: text/csv' \ --data-binary '@/local/path/to/your/data/file.csv'`

### Checking file status:

```shell

curl --request GET 'https://auth.region.environment.soliduslabs.com/api/v1/list-files'
--header 'Content-Type: application/json'
--header 'X-Auth-Token:  your-issued-security-token'
```

Checking up on your previously uploaded files and their processing status can be achieved by another endpoint in our API: 

    `curl --request GET 'https://auth.region.environment.soliduslabs.com/api/v1/list-files' --header 'Content-Type: application/json' --header 'X-Auth-Token:  your-issued-security-token'`



# FIX 4.4

> FIX 4.4 Call examples for exchanges

```shell

# New Order

8=FIX.4.4|9=293|35=D|34=22|49=CLIENT|52=20190601-00:00:02.427618005|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=0|14=0|432=20190602-00:00:00|11=28561684|151=5|528=G|38=5|40=2|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:01.427618005|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=222|
```
```shell

# Cancel Request

8=FIX.4.4|9=298|35=F|34=123|49=CLIENT|52=20190601 00:00:04.001613021|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=8556.94|14=1|432=20190602-00:00:00|11=28561684|151=4|528=G|38=5|40=1|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:02.798615123|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=091|
```
```shell

# Partial Fill

8=FIX.4.4|9=293|35=D|34=22|49=CLIENT|52=20190601-00:00:02.427618005|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=0|14=0|432=20190602-00:00:00|11=28561684|151=5|528=G|38=5|40=2|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:01.427618005|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=222|

```

> FIX 4.4 Call examples for broker dealers, market makers, hedge funds

```shell

# New Order

8=FIX.4.4|9=293|35=D|34=22|49=CLIENT|52=20190601-00:00:02.427618005|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=0|14=0|432=20190602-00:00:00|11=28561684|151=5|528=G|38=5|40=2|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:01.427618005|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=222|
```

```shell

# Cancel Request

8=FIX.4.4|9=293|35=D|34=22|49=CLIENT|52=20190601-00:00:02.427618005|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=0|14=0|432=20190602-00:00:00|11=28561684|151=5|528=G|38=5|40=2|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:01.427618005|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=222|
```

```shell

# Partial Fill

8=FIX.4.4|9=297|35=8|34=56|49=CLIENT|52=20190601-00:00:03.117612011|56=SOLIDUS|1=cd64368c-3283-40c0-bccb-585b779bba5a|6=8556.94|14=1|432=20190602-00:00:00|5300=28561690|37=28561684|17=94945|30=GS|151=4|528=G|38=5|39=1|40=1|41=28561684|44=8556.94|54=2|55=BTC/USD|58=Text|59=1|60=20190601-00:00:02.123615001|851=2|453=3|448=999010573|447=D|452=3|448=T100001|447=D|452=11|448=DBCA|447=D|452=13|10=026|
```


An API Key is required to authenticate before you can send messages using FIX Engine

* Clients will have TCP connection setup to send messages and heartbeats.
* All FIX requests will result in a Heartbeat incrementing the sequence number on the message.

# Transaction Monitoring

> Example transaction file

```shell

# New Order

Id,Version,TransactTime,ActorId,ExternalAccountId,AccountType,AccountFunction,InternalAccountId,Currency,Quantity,Type,Status,TransactionHash
fb8078a4839f,1,1605493727,act445,0xe5efbdcd9ee2178779cec66150fdb3d0e537eeb7,CRYPTO,TRADING,0x82461e670c212099a9c70e68420a50559a235fdd,ETH,2.23,DEPOSIT,APPROVED,0x5213bd89742634331ea864c3bc75bc78607d50e55828fb8078a4839f942b583e
```


Data requirement from clients differ based on the type of client entity. Following are the different applicable data formats applicable for files and REST.

## Transactions

<aside class="notice">
Field names marked with a * are mandatory fields
</aside>

Field Name | Type | Description | Example
--------- | ------- | ----------- | -----------
*`Id` | `String` | Unique Transaction identifier | 
*`ActorId` | `String` | Transaction Creator Identifier | 
*`ExternalAccountId` | `String` | Digital wallet or bank account. This is the external account related to the ActorId | 
*`Currency` | `String` | Currency symbol. Ex: BTC, USD | 
*`Quantity` | `float` | Total units currency | 
*`Type` | `String` |  | DEPOSIT,<br>WITHDRAWAL
*`Status` | `String` |  | PENDING,<br>ACCEPTED,<br>REJECTED
*`TransactTime` | `UTCTimestamp` | Time of transaction | 
`Version` | `Integer` | Version of the transaction | 
`InternalAccountId` | `String` | Internal account owned by the Solidus client. Digital wallet or bank acocunt. | 
`TransactionHash` | `String` | Blockchain transaction hash. Optional for cash txn. | 
*`ExternalAccountType` | `String` | Type of account (ENUM = FIAT, CRYPTO) | 
*`ExternalAccountFunction` | `String` | Intended function of the account if available (ENUM = DEPOSIT, TRADING, HEDGING) | 


## User Profile

Field Name | Type | Description | Example
--------- | ------- | ----------- | -----------
*`Created` | `String` | Time at which profile was created | 
*`CreatedBy` | `String` | User id creating the profile | 
*`LastUpdated` | `String` | Time at which profile was updated last | 
*`LastUpdatedBy` | `String` | User id updating the profile | 
*`Version` | `int` | Version of the profile | 
*`UserID` | `String` | User id for the profile | 
*`Country` | `String` | Country of residence. | USA
*`IpAddress` | `String` |  | 
*`Nationality` | `String` |  | 
*`SourceOfFunds` | `list` |  | ["SAVINGS","INVESTMENTS"]
*`AnnualIncome` | `String` |  | 
*`AccountPurpose` | `list` | | ["LONG_TERM","DAY_TRADING"]
*`TradingFrequency` | `String` | Trading frequency for the profile | DAILY,WEEKLY,..
*`AverageTradeSize` | `String` | Trading size for the profile | UNDER_10K,10K-50K,..
*`LiquidNetWorth` | `String` | Time at which profile was created | UNDER_100K,100K_500K,..
*`AffiliatedAccounts` | `list` | Time at which profile was created | 
*`KycProvider` | `String` | Time at which profile was created | 
*`KycRiskScore` | `float` | Time at which profile was created | 0, 1, 2
*`KycStatus` | `String` | Time at which profile was created | PASSED

## Institutional Profile

Field Name | Type | Description | Example
--------- | ------- | ----------- | -----------
*`Created` | `UTCTimestamp` | Time at which profile was created | 
*`CreatedBy` | `String` | user id creating the profile | 
*`LastUpdated` | `UTCTimestamp` | Time at which profile was updated last | 
*`LastUpdatedBy` | `String` | user id updating the profile | 
*`Version` | `int` | Version of the profile | 
*`UserID` | `String` | User id for the profile | 
*`Country` | `String` | Country of residence. | USA, HKG
*`IpAddress` | `String` |  | 
*`CountryOfIncorporation` | `String` |  | 
*`BusinessType` | `String` |  | 
*`KybRiskScore` | `String` |  | 
*`TradingPurpose` | `String` |  | 
*`SourceOfFunds` | `String` |  | 
*`TradingFrequency` | `String` |  | DAILY,WEEKLY,..
*`AverageTradeSize` | `String` | | 50K_250K,250K_500K,..
*`AffiliatedAccounts` | `list` | Time at which profile was created | 


# FAQ

## Data

**Do I need to send PII data for Solidus Labs algorithms to work?**

No. Solidus Labs algorithms do not require PII.

**Do I need to anonymize data before sending to Solidus Labs?**

Yes, please anonymize the data by removing all sensitive information from it, such as PII.
‍
**How can I ensure that the data is anonymized?**

The easiest way to only provide the data as per data requirement section above.

**What is the best practice of collecting the data on my end?**

It depends on the system architecture of data storage of clients' end. If the data is stored in a database, one of the best approaches would be to define a stored procedure or a function to extract the data. Then call scored procedure from a script to fetch the data for the required time period. This data can be formatted using a scripting or a programming language. For batch alerting, the data is collected in a file. For near real time alerting, the data is converted to JSON formatted and sent via REST API to Solidus Labs. For real time alerting, a messaging bus such as Kafka can be used as a channel for getting the data directly from the order management system.

**What is the fastest way to get my data integration setup?**

Flat files are the easiest interface to get up and running with the data integration. After the data is collected and formatted on clients' side, it takes a couple of commands to send the file over. See flat files sections for details.

**Is there a Solidus Labs QA environment where I can test my data transfer mechanism?**

Yes. You should test the data transmission setup and get the fields checked with Solidus Labs onboarding team before turning on the data flow in production.

## REST API

**Is there a rate limit for REST API?**

Yes. There is an upper limit of 120 messages per minute.

**What happens if I exceed REST API rate limit?**

Rest API calls will fail with response 429 in the status code.

**What is the expected data format for REST API requests?**

JSON. Please see REST API section for details and examples.
‍
## Flat Files

**Is there a size limit on the file?**

Yes, the upper limit for a single file is 5GB. The file can be broken into multiple files with individual sizes less than the upper limit.

**What happens if I try to upload a file bigger that 5GB in a single command?**

Request will fail with the message file too large.

**How often should I send the data via files?**

File transfer frequency depends on client alerting needs. Typically the file transfer frequency ranges from a few hours to a few days.

**Do i need to schedule the files to be sent at the same time?**

No. Solidus Labs watches for the files from the client continuously. So the file will be processed when it is received. However it is a good practice to use a scheduler to send files around the same time to support timely deliver of alerts to the compliance team. Examples of such schedulers are Autosys or Unix cron.

**How can I report an issue with data transmission?**

Please drop an email with issue description to apisupport@soliduslabs.com

**Is there a public API for Solidus?**

No, Solidus product is available to paid customer only. Please reach out at hello@soliduslabs.com to learn more about Solidus compliance product suite.

