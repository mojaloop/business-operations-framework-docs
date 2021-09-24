# Reporting & Auditing Bounded Context
One of the objectives of this workstream project is to provide the ability to trace a transfer end to end. In order to deliver on this objective, part of the reporting and auditing bounded context is to be built in line with the Reference Architecture.

## Design Overview
Here is the overall architectural design.
![Architecture Overview Diagram of Reporting Bounded Context](/Reporting-&-Auditing-Overview.png)

In Mojaloop, all the core services are already pushing events to kafka on a topic ‘topic-event’.

There are two fundamental reporting databases
1. **Reporting DB** 
The reporting database is a relational database that keep track of the latest state of the Mojaloop ojects and makes them available through and efficenty query interface. 
In the implementation of this workstream effort, a reporting dedicated replica of the central ledger database will be used. This doesn't quite fit the architectural model as a database owned by the reporting and auditing bounded context should not have external depencencies. A central ledger replica data is dependent on the schema of the central ledger and therefor has an external dependency. 
::: warning Technical Debt
This should be recognised as **technical debt** that should be paid as more of the reference architecture is built. 
:::
There are two approaches that can be take in paying this technical debt:
  - Changing the replica call to a **oneway data sync** function; which would decouple the schemas of the two databases.
  - Rebuild a new designed **relational database** which is updated based on subscribed kafka topics.
The best approach will depend on the state of the current Mojaloop version at the time that this debt is paid.
2. **Event DB Store** 
The event DB store is a capture of the event details that can provided a more detailed reporting view of what happenned.   
  **Limitations of the event store effort in this workstream**
This design will be implemented on the current Mojaloop version. 
Currently only the data required to provide end to end tracing of a transfer will be collected and made available through the reporting API.
Extensions to this offering can easily be added by extending the event processor to process new use case messages and store them in the Mongo DB, and then configure the generic graphQL resource query to query the new data stores appropriately.

## Alignment to reference architecture
Although the bounded context refers to reporting and auditing, this project only begins to tackle the reporting part of that definition. The current design is independent from other bounded contexts, which is in-line with the reference architecture. 
(There isn't a complete seperation as the current design is using a replica database as the reporting database. The technical debt and next steps is to reslving this are described above.)

It is important to consider how this bounded context will change as more of the reference architecture design is implemented?
### Bounded Contexts will during the Reference Architecture implmentation stop storing data in the central ledger databases
There are three approaches as that can be adopted to acommodate this change. How the reference architecture is build will determine which is best approach:
1. Modifying the sync functionality to accomodate the bounded context new data store
2. Extending the message event processor to capture the required information in the reporting database.
3. Call newly defined bounded context APIs, to retrieve the required data.

## Four use cases defined
In order to effectively trace a transfer end to end, the following four use cases where defined to enable this.
1. **Dashboard view use case**
As a Hub Operator Business Operations Specialist,
I want a high level dashboard summary of the transfers moving through the hub that is derived from a date time range 
So that I can proactively monitor the health of the ecosystem.

:::::: col-wrapper
| Data returned |
| --- |
| Transfer count |
| Transfer amount total per currency |
| Transfer count per error code  |
| Transfer count per payer DFSP |
| Transfer count per payee DFSP |
| Transfer amount per currency per payer DFSP |
| Transfer amount per currency per payee DFSP |
:::::::::

2. **Transfer list view use case**
As a Hub Operator Business Operations Specialist,
I want to view a list of transfers that can be filtered based on one or more of the following 
Always required (must be provided in every call)
- Date time range
Optional filters 
- A specific Payee DFSP
- A specific Payee Id type 
- A specific Payee
- A specific Payer DFSP
- A specific Payer Id type
- A specific Payer
- State of the transfer
- Currency
Nice to have filters (These are not a strict requirement, but should be provided if the design allows for it.)
- A Specific Error Code
- Settlement window
- Settlement Batch Id: The unique identifier of the settlement batch in which the transfer was settled. If the transfer has not been settled yet, it is blank.
Search String on messages

... So that I proactively monitor the health of the ecosystem by having a more detailed view of the transfer data moving through the switch.

:::::: col-wrapper
| Data returned | |
| --- | --- |
| Transfer ID | The unique identifier of the transfer. |
| Transfer State | Indicates if the transfer has succeeded, is pending, or an error has occurred. |
| Transfer Type | (e.g.P2P) |
| Currency | The transfer currency. |
| Amount | The transfer amount. |
| Payer DFSP  | |
| Payer Id Type  | |
| Payer | |
| Payee DFSP | |
| Payee Id Type | |
| Payee | |
| Settlement Batch Id | The unique identifier of the settlement batch in which the transfer was settled. If the transfer has not been settled yet, it is blank. |
| Date Submitted | The date and time when the transfer was initiated. |
:::::::::

3. **Transfer detail view use case**
As a Hub Operator Business Operations Specialist, I want to trace a specific transfer from it’s transfer ID
- So that I can Identify 
- The timing and current state of the transfer
- Any error information that is associated with that transfer
- The associated quoting information and timing for that transfer
- The associated settlement process status and identifiers

:::::: col-wrapper
| Data returned | |
| --- | --- |
| Transfer ID | The unique identifier of the transfer. |
| Transfer State | Indicates if the transfer has succeeded, is pending, or an error has occurred. |
| Transfer Type | (e.g.P2P) |
| Currency | The transfer currency. |
| Amount | The transfer amount. |
| Settlement Batch Id | The unique identifier of the settlement batch in which the transfer was settled. If the transfer has not been settled yet, it is blank. |
| Payer |  |
| Payer Details | The unique identifier of the payer (typically, a MSISDN, that is, a mobile number). |
| Payer DFSP | |
| Payee DFSP | |
| Payee | |
| Payee Details | The unique identifier of the payer (typically, a MSISDN, that is, a mobile number). |
| Transfer State | Indicates if the transfer has succeeded, is pending, or an error has occurred. |
| Date Submitted | The date and time when the transfer was initiated. |
:::::::::

4. **Transfer message view use case**
As a Hub Operator Business Operations Specialist, 
I want to view the detailed messages from it’s transfer ID
So that I can investigate any unexpected problem associated with that transfer

:::::: col-wrapper
| Data returned | |
| --- | --- |
| Scheme Transfer ID | |
| TransferID | |
| QuoteID | |
| Home Transfer ID | |
| Payer and Payee Information | Id Type, Id Value, Display Name, First Name,Middle Name, Last Name, Date of birth, Mechant classification code, FSP Id, Extension List |
| Party Lookup Response | |
| Quote Request | |
| Quote Response | |
| Transfer Prepare | |
| Transfer Fulfill | |
| Error message/s | |
:::::::::

## Business work flow
Here is a business work flow that describes how the use cases are called.
![Business Work Flow](/BusinessFlowView.png)

## Tools Chosen
1. **Event Data Store: MongoDB**
The MongoDB database was chosen because:
   - MongoDB is currently used and deployed in Mojaloop, and is an excepted open sourced tool that optional has standard companies that can provide enterprise support should it be required.
   - MongoDB will meet our requirements for this project.
   - Other tools where considered but were found not to meet all the requirements for and OSS tool in Mojaloop.
 
2. **GraphQL**
The Graph QL technology choice for this API is:
   - Existing reporting solution didn't scale well with the complex reporting requirements being asked of it.
      Exisiting solution simple with backend flexibility
      Increase in complexity of reporting asks resulted in complex difficult to write SQL statements.
      Concern is that longterm this could become difficult to maintain and requires specialists to build.
   - New tools simplifying Graph QL implementation and we have a resident expert. I.e. it has not been a large work effort.
   - Will provide examples so that little or no GraphQL prior knowledge is required.

**Additional advantages on using GraphQL:**
   - Reusable resources/associated RBAC permissions between reports
   - Complex queries are simpler to build because resources are modeled. 
   - Mixing of data sources in a single query (e.g. MySQL with MongoDB ).
   - No requirement for nested fetches.
   - No requirement for multiple fetches.
   - No requirement for API version. API naturally supports backward compatibility between versions. 

**Community Reservations on using GraphQL**
The community has raised a concern regarding the choice of graphQL for the API instead of using a standard rest implmentation. The main argument for this is that the current requirment or use case can be fulfilled with a REST base approach so this doesn't warrant introducing a new technology. 
This can be refrased into this question:
:::tip Crux of the issue
Does the addition of GraphQL add more complexity to the solution than the benefits that it brings?
:::
Although this does have merit and is worth discussion, unfortunately this concern was raised too late in the build process so was not able to adjust the design. Our implementation of GraphQL doesn't add much complexity and fits better with the RBAC requirements, and is developer freindly. 

The implementation of the GraphQL API is only a small part of the work effort, and can easily be replaced or even implemented in parallel. So should we be proved to be wrong this approach can easily be changed at a later date.

## Building the Event data store
The purpose the Event data store, is to provide a persistent storage of events of interest that are easily and efficiently found and queried for reporting.

To achieve this with minimal structure changes from the original message, it was decided to process the message into categories and store these categories as additional meta data inside the message that can be queried later on. Messages that do not fit withing these categories are not stored and are therefore filtered out.

Here is an example of the meta-data that is added to the json message
```json{7-12}
{
    “event”: {
        "id" : {},
        "content" : {},
        "type" : {},
        "metadata" : {}
    },
    “metadata” : {
        “reporting” : {
            “transactionId” : “...”,
            “quoteId”: “...”,
            “eventType” : “Quote”
        }
    }
}
```
Where the eventType can be one of the following:
:::::: col-wrapper
::: col-third
:::

::: col-third
| eventType	|
| ------- |
|Quote |
|Transfer |
|Settlement	|
:::
:::::::::
  
The event stream processor will subscribe to the Kafa Topic **'topic-event'**. This message queue contains all the event messages. A sigificant degree of filtering is therefore necessary.

:::tip Note:
The code delivering this functionality has been structure so that these filters can easily be modified or extended.
The subscribed and classified messages are represented in 'const' files so can easily be added to or amended without detailed knowledge of the code.
:::
### Storing only 'Audit' messages

Only Kafka meesages that are of type 'audit' will be considered for saving.
I.e. Only if:
:::::: col-wrapper
::: col-third
:::
::: col-third
| metadata.event.type |
| ---- |
| audit |
:::
:::::::::

## These are the 'Transfer' messages that are stored
**ml-api-adapter**

:::::: col-wrapper
| metadata.trace.service |
| ---- |
| ml_transfer_prepare |
| ml_transfer_fulfil |
| ml_transfer_abort |
| ml_transfer_getById |
| ml_notification_event |
:::::::::

## These are the 'Quote' messages that are stored
**quoting-service**
:::::: col-wrapper
::: col-third
| metadata.trace.service |
| ---- |
| qs_quote_handleQuoteRequest |
| qs_quote_forwardQuoteRequest |
| qs_quote_forwardQuoteRequestResend |
| qs_quote_handleQuoteUpdate |
| qs_quote_forwardQuoteUpdate |
| qs_quote_forwardQuoteUpdateResend |
| qs_quote_handleQuoteError |
| qs_quote_forwardQuoteGet |
| qs_quote_sendErrorCallback |
:::

::: col-third
| metadata.trace.service |
| ---- |
| qs_bulkquote_forwardBulkQuoteRequest |
| qs_quote_forwardBulkQuoteUpdate |
| qs_quote_forwardBulkQuoteGet |
| qs_quote_forwardBulkQuoteError |
| qs_bulkQuote_sendErrorCallback |
:::

::: col-third
| metadata.trace.service |
| ---- |
| QuotesErrorByIDPut |
| QuotesByIdGet |
| QuotesByIdPut |
| QuotesPost |
| BulkQuotesErrorByIdPut |
| BulkQuotesByIdGet |
| BulkQuotesByIdPut |
| BulkQuotesPost |
:::
:::::::::

## These are the 'Settlement' messages that are stored
**central-settlement**
:::::: col-wrapper
::: col-third
| metadata.trace.service |
| ---- |
| cs_process_transfer_settlement_window |
| cs_close_settlement_window |
| ... |
:::

::: col-third
| metadata.trace.service |
| ---- |
| getSettlementWindowsByParams |
| getSettlementWindowById |
| updateSettlementById |
| getSettlementById |
| createSettlement |
| closeSettlementWindow |
| ... |
:::
:::::::::

## These messages currently will remain unclasified and are filtered out 
**account-lookup-service (not in PI - included as a reference)**
:::::: col-wrapper
::: col-third
| metadata.trace.service |
| ---- | 
| ParticipantsErrorByIDPut |
| ParticipantsByIDPut |
| ParticipantsErrorByTypeAndIDPut |
| ParticipantsErrorBySubIdTypeAndIDPut |
| ParticipantsSubIdByTypeAndIDGet |
| ParticipantsSubIdByTypeAndIDPut |
| ParticipantsSubIdByTypeAndIDPost |
| ParticipantsSubIdByTypeAndIDDelete |
| ParticipantsByTypeAndIDGet |
| ParticipantsByTypeAndIDPut |
| ParticipantsByIDAndTypePost |
| ParticipantsByTypeAndIDDelete |
| ParticipantsPost |
| PartiesByTypeAndIDGet |
| PartiesByTypeAndIDPut |
| PartiesErrorByTypeAndIDPut |
| PartiesBySubIdTypeAndIDGet |
| PartiesSubIdByTypeAndIDPut |
| PartiesErrorBySubIdTypeAndIDPut |
:::

::: col-third
| metadata.trace.service |
| ---- |
| OraclesGet |
| OraclesPost |
| OraclesByIdPut |
| OraclesByIdDelete |
:::

::: col-third
| metadata.trace.service |
| ---- |
| postParticipants |
| getPartiesByTypeAndID |
| ... |
:::
:::::::::

**transaction-requests-service (not in PI - included as a reference)**
:::::: col-wrapper
::: col-third
| metadata.trace.service |
| ----- |
| TransactionRequestsErrorByID |
| TransactionRequestsByID |
| TransactionRequestsByIDPut |
| TransactionRequests |
| AuthorizationsIDResponse |
| AuthorizationsIDPutResponse |
| AuthorizationsErrorByID |
:::

::: col-third
| metadata.trace.service |
| ----- |
| forwardAuthorizationMessage |
| forwardAuthorizationError |
| ... |
:::
:::::::::

### Useful tools
#### Kafka explorer
The [‘kowl’](https://github.com/cloudhut/kowl) software from cloudhut is very useful to explore all the kafka messages in mojaloop cluster. We can deploy it in the same namespace as the mojaloop core services.

The custom values file for the OSS deployment can be found in this [repository](https://github.com/mojaloop/deploy-config/tree/deploy/PI15.2/mojaloop/kowl-kafka-ui)
(This is private repository, you may need permission to access this link)

**Steps to install:**
```
helm repo add cloudhut https://raw.githubusercontent.com/cloudhut/charts/master/archives
helm repo update
helm install kowl cloudhut/kowl -f values-moja2-kowl-values.yaml
```
**Web UI**
Open the URL configured in ingress section in the values file

Additional customization
See [reference config.](https://github.com/cloudhut/kowl/blob/master/docs/config/kowl.yaml)

#### TTK golden path
The TTK golden path test cases have been design to explore all the possible test outcomes possible when sending transfers. This is therefore and important tool that can be used to test that the functionallity caters for all eventualities. 
I.e. We can use the inbuilt TTK to execute different test-cases like p2p happy path, negative scenarios, settlement related use cases...etc

## Event Processing Service
Responsible for subscribing to Kafka topics and filtering through the events by event type.
The event type further breaks down into several parts depending on which service the events were produced from.

The filtered events will then be processed depending on the context of the event structure and reporting metadata will be created.

Example
```json
{
    “event”: {
        "id" : {},
        "content" : {},
        "type" : {},
        "metadata" : {}
    },
    “metadata” : {
        “reporting” : {
            “transactionId” : “...”,
            “quoteId”: “...”,
            “eventType” : “Quote”
        }
    }
}
```

| eventType	| Event origin |
| ------- | ------- |
|Quote | quoting-service |
|Transfer | ml-api-adapter |
|Settlement	| central-settlement |

It subscribes to the Kafka event stream to build an event transfer related store that is queryable through the operational API. This is in-line with the reference architecture.


## Graph QL API - generic resource implementation explained
At the heart of the implementation of this bounded context is a generic implementation that links a reporting data store and a query to a graph QL data resource that has its own RBAC authorisation.
I.e. a new customised resource can be added to this API by doing these three thing.

1. Define the data store type
2. Define the query
3. Define the graph QL resource names and fields
3. Define the user permission that is linked to this resource

### GraphQL Query examples
**Query Transfers that are filered on a specific payer DFSP**
```GraphQL
query GetTransfers {
  transfers(filter: {
    payer: "payerfsp"
  }) {
    transferId
    createdAt
    payee {
      name
    }
  }
}
```

**Query a summary of the transfers**
```GraphQL
query TransferSumary2021Q1 {
  transferSummary(
    filter: {
        currency: "USD"
        startDate: "2021-01-01"
        endDate: "2021-03-31"
    }) {
        count
        payer
  }
}
```

**Query**
```GraphQL
```
## 
