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

## Four use cases defined
In order to effectively trace a transfer end to end, the following four use cases where defined to enable this.
1. **Dashboard view use case**
As a Hub Operator Business Operations Specialist,
I want a high level dashboard summary of the transactions moving through the hub that is derived from a date time range 
So that I can proactively monitor the health of the ecosystem.

| Data returned |
| --- |
| Transaction count |
| Transaction amount total per currency |
| Transaction count per error code  |
| Transaction count per payer DFSP |
| Transaction count per payee DFSP |
| Transaction amount per currency per payer DFSP |
| Transaction amount per currency per payee DFSP |

2. **Transfer list view use case**
As a Hub Operator Business Operations Specialist,
I want to view a list of transactions that can be filtered based on one or more of the following 
Always required (must be provided in every call)
- Date time range
Optional filters 
- A specific Payee DFSP
- A specific Payee Id type 
- A specific Payee
- A specific Payer DFSP
- A specific Payer Id type
- A specific Payer
- State of the transaction
- Currency
Nice to have filters (These are not a strict requirement, but should be provided if the design allows for it.)
- A Specific Error Code
- Settlement window
- Settlement Batch Id: The unique identifier of the settlement batch in which the transfer was settled. If the transfer has not been settled yet, it is blank.
Search String on messages

... So that I proactively monitor the health of the ecosystem by having a more detailed view of the transaction data moving through the switch.

| Data returned | |
| --- | --- |
| Transfer ID | The unique identifier of the transfer. |
| Transfer State | Indicates if the transfer has succeeded, is pending, or an error has occurred. |
| Transaction Type | (e.g.P2P) |
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

3. **Transfer detail view use case**
As a Hub Operator Business Operations Specialist, I want to trace a specific transaction from it’s transaction ID
- So that I can Identify 
- The timing and current state of the transaction 
- Any error information that is associated with that transaction
- The associated quoting information and timing for that transaction
- The associated settlement process status and identifiers

| Data returned | |
| --- | --- |
| Transfer ID | The unique identifier of the transfer. |
| Transfer State | Indicates if the transfer has succeeded, is pending, or an error has occurred. |
| Transaction Type | (e.g.P2P) |
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

4. **Transfer message view use case**
As a Hub Operator Business Operations Specialist, 
I want to view the detailed messages from it’s transaction ID
So that I can investigate any unexpected problem associated with that transaction

| Data returned | |
| --- | --- |
| Scheme Transfer ID | |
| TransactionD | |
| QuoteID | |
| Home Transfer ID | |
| Payer and Payee Information | Id Type, Id Value, Display Name, First Name,Middle Name, Last Name, Date of birth, Mechant classification code, FSP Id, Extension List |
| Party Lookup Response | |
| Quote Request | |
| Quote Response | |
| Transfer Prepare | |
| Transfer Fulfill | |
| Error message/s | |

## Business work flow
Here is a business work flow that describes how the use cases are called.
![Business Work Flow](/BusinessFlowView.png)

## Tools Chosen
1. **Event Data Store: MongoDB**
todo: Need to state the reasons why
2. **GraphQL**
todo: Need to state the reasons why

## Building the Event data store
The purpose of
Persistent storage of events.

### Choosing and mapping the events
Here is a more detailed view of the topic-event structure.

#### Event data

Only audit logs will be stored but there are different event types in the topic-event

| metadata.event.type |
| ---- |
| audit |

Events can further be classified by actions

| metadata.event.action |
| ---- |
| *default* |
| start |
| egress |
| *ingress* |
| *finish* |
(Italic event.actions not found in audits, yet?)

#### Trace Data
The trace data gives us more information on which service created the events. 

##### ml-api-adapter

| metadata.trace.service |
| ---- |
| ml_transfer_prepare |
| ml_transfer_fulfil |
| ml_transfer_abort |
| ml_transfer_getById |
| ml_notification_event |

##### quoting-service

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


| metadata.trace.service |
| ---- |
| qs_bulkquote_forwardBulkQuoteRequest |
| qs_quote_forwardBulkQuoteUpdate |
| qs_quote_forwardBulkQuoteGet |
| qs_quote_forwardBulkQuoteError |
| qs_bulkQuote_sendErrorCallback |


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

##### central-settlement
| metadata.trace.service |
| ---- |
| cs_process_transfer_settlement_window |
| cs_close_settlement_window |
| ... |


| metadata.trace.service |
| ---- |
| getSettlementWindowsByParams |
| getSettlementWindowById |
| updateSettlementById |
| getSettlementById |
| createSettlement |
| closeSettlementWindow |
| ... |

##### account-lookup-service (not in PI - included as a reference)
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


| metadata.trace.service |
| ---- |
| OraclesGet |
| OraclesPost |
| OraclesByIdPut |
| OraclesByIdDelete |


| metadata.trace.service |
| ---- |
| postParticipants |
| getPartiesByTypeAndID |
| ... |

##### transaction-requests-service (not in PI - included as a reference)
| metadata.trace.service |
| ----- |
| TransactionRequestsErrorByID |
| TransactionRequestsByID |
| TransactionRequestsByIDPut |
| TransactionRequests |
| AuthorizationsIDResponse |
| AuthorizationsIDPutResponse |
| AuthorizationsErrorByID |


| metadata.trace.service |
| ----- |
| forwardAuthorizationMessage |
| forwardAuthorizationError |
| ... |

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

It subscribes to the Kafka event stream to build an event transaction related store that is queryable through the operational API. This is in-line with the reference architecture.
Reporting DB

This service also queries the reporting DB to report on transactional state. In the new reference architecture design, a new transactional state store would need to be built up from the Kafka event stream.

## Graph QL API - generic resource implementation explained
At the heart of the implementation of this bounded context is a generic implementation that links a reporting data store and a query to a graph QL data resource that has its own RBAC authorisation.
I.e. a new customised resource can be added to this API by doing these three thing.

1. Define the data store type
2. Define the query
3. Define the graph QL resource names and fields
3. Define the user permission that is linked to this resource

Todo: need to provide file examples for MySQL and MongoDB

## 
