# Settlement Operational Implementation

## Introduction

The objectives of this implementation is to provide a solution to Settlements that supports the core-settlement operations and the high-level settlement business process functions. This guide outlines the high-level design and explains the thinking that went into the design.

## Core-Settlement Operations

This is the existing Settlement functionality provided by the supporting [Central-Settlement](https://github.com/mojaloop/central-settlement) Mojaloop Core component. Detailed information can be found in the [Mojaloop Technical Overview Documentation](https://github.com/mojaloop/documentation/tree/master/legacy/mojaloop-technical-overview/central-settlements).

The Core-Settlement operations support the following capabilities:

- Create a Settlement Matrix Report based on a list of Settlement-Windows
- Process Settlement Acknowledgements for an existing Settlement Matrix Report
- Manage Settlement-Windows (i.e. Create, Close, etc)
- Queries for Settlement Matrix Reports, Settlement-Windows, etc

The OpenAPI definition is available at the [Mojaloop-Specification repository](https://github.com/mojaloop/mojaloop-specification/tree/master/settlement-api).

## High-level Architecture

![High-level Settlement Architecture](../.vuepress/public/BizOps-Framework-Settlements.png)

### Experience layer
The settlement experience layer is a stateless API that exposes the data to be consumed by its intended audience. Currently it's main function is to add the looked up user information into the API. (This information is currently sitting the in headers after being added there by the ORY Oathkeeper proxy.) This function is expected to become larger as the product develops.

### Process layer
Process APIs provide a means of combining data and orchestrating multiple System APIs for a specific business purpose. The mojaloop central-settlement, and central-ledger API are being consumed by this process API.

## High-level Settlement Business Process

This is a process that relies on the existing core-settlement operations to orchestrate the following capabilities:

1. **Closing a settlement window**
The current settlement window can be closed manually as if there have been transfers linked to the settlement window. The hub operator can select the current open window, and then choose to close the window.

1. **Settlement Initiation**
Settlement Initiation is used by the hub operator to create a settlement batch which controls and drives the settlement process. 
To initiate the settlment process, the hub operator selects : 
   - a set of settlement windows 
   - and optionally a settlement currency or a settlment module. (If a settlement currency is provided, then this is used to determine the 
   settlement module.)
The position ledgers of the net credit participants are adjusted during Settlement Initiation.
**Note:** It is important to create the batch settlment object in the way that the settlement is to be completed and finalised. 
1. A **Settlement Matrix report** is generated and used to comunicated to the settlement bank the requirements of the settlement.
1. **Settlement Finalization & settlement account rebalancing**
This process needs to occur after the settlement bank has applied the settlement changes. In this step the:
   - settlement process is completed. 
   - the net debit participants in the settlment have their position ledgers adjusted.
   - the settlement ledgers are adjusted for all participants to match the transferred amount for the settlement.
   - the settlement ledgers are checked against the real settlment account balances and adjustments processed to ensure that they are aligned.


## Detailed Sequence Diagram

![Settlement Detailed Process](../.vuepress/public/settlementProcessAPI.svg) 


