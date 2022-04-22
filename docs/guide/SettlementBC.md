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
To initiate the settlement process, the hub operator selects : 
   - a set of settlement windows 
   - and optionally a settlement currency or a settlement model. (If a settlement currency is provided, then this is used to determine the settlement model.)
The position ledgers of the net credit participants are adjusted during Settlement Initiation.
**Note:** It is important to create the batch settlment object in the way that the settlement is to be completed and finalised. 
1. A **Settlement initiation report**  is generated and used to comunicated to the settlement bank the requirements of the settlement.
1. **Settlement Finalization & settlement account rebalancing**
This process needs to occur after the settlement bank has applied the settlement changes. In this step the:
   - settlement process is completed and a settlement finalisation report has been recieved from the settlement bank.
   - the net debit participants in the settlment have their position ledgers adjusted.
   - the settlement ledgers are adjusted for all participants to match the transferred amount for the settlement.
   - the settlement ledgers are checked against the real settlment account balances and adjustments processed to ensure that they are aligned.


## Detailed Sequence Diagram

![Settlement Detailed Process](../.vuepress/public/settlementProcessAPI.svg) 

There are a couple of processes in the sequence diagram that are worth elaborating on.
### Determining settlement Model
This need to first determine which currencies are involved in the settlement seleted, and then choose an appropriate settlement model or list of settlement models.

### Validation of the settlement finalisation data
The data that is presented as part of the settlement finalisation needs a significant amount of validations on the data. 
Some validaions check integrity of the data, and these check will fail the process if not passed. Other valications do not prevent the process continuing, however will show warnings that need to be presented to the operator.
The continuation of the process is only possible once the operator has accepted the confirmed warnings with their resultant effects, and has provided their selected options for how the process should be applied.
It is for this reason that the validation of the data is a necessary step, and must be referenced when accepting and proceeding with the process.

### Use cases that need to be catered for in the settlment finalisation
| Validation Description | Expected behaviour |
| --- | --- |
| Selected settlement ID does not match report settlement ID | Abort finalisation with error |
| Sum of transfers in the report is non-zero | Abort finalisation with error |
| Transfer amount does not match net settlement amount | Abort finalisation with error |
| Balance not modified corresponding to transfer amount | Continue --> Adjust Settlement Account Balance to align |
| Balance provided in the report is not a positive value | Continue --> Set settlement balance to zero; set NCD = 0; disable participant POSITION account |
| Accounts in settlement not present in report | Abort finalisation with error |
| Accounts in report not present in settlement | Abort finalisation with error |
| Participant identifiers do not match - participant ID, account ID and participant name must match | Abort finalisation with error |
| Account type should be POSITION | Abort finalisation with error |
| New balance amount not valid for currency | Abort finalisation with error |
| Transfer amount not valid for currency| Abort finalisation with error |
| Account ID does not exist in switch | Abort finalisation with error |
|Attempted to finalize an aborted settlement | Abort finalisation with error |
| Error processing adjustment for participant | Continue with other participants - notify user of error |
| Error attempting to set settlement state to PS_TRANSFERS_RECORDED | Continue with other participants - notify user of error |
| Error attempting to set settlement state to PS_TRANSFERS_RESERVED | Continue with other participants - notify user of error|
|Error attempting to set settlement state to PS_TRANSFERS_COMMITTED | Continue with other participants - notify user of error|
|Errors attempting to settle accounts | Continue with other participants - notify user of error|
| Error attempting to set NDC | Continue with other participants - notify user of error |
| Error attempting to process funds in/out|Continue with other participants - notify user of error |
| Balance unchanged after processing funds in/out| Continue with other participants - notify user of error |
|Incorrect resulting balance after processing funds in/out |Continue with other participants - notify user of error  |
|Failed to record settlement participant account state |Continue with other participants - notify user of error  |

### Audit information in the current Mojaloop verion
The process being performed is captured in the settlement reason field, and is therefore available in the audit reports.
Additionally the user and the references are captured in the extension lists. These too can be queried in the audit reports.

### RBAC
In order to make full use of the RBAC controls, the above four processes will be implemented as seperate API endpoint & HTTP method combinations. This is to allow a different permissions to be associated with each process.

## Multi-currency support
How multi-currency settlement are executed, depends on two things:
1. How the settlment models are constructed?
Settlement models can be linked to a currency or left un-linked and applicable to all currencies.
1. How the settlements are initiated?
Settlement can be initiated with optionally a currency, or optionally a settlement model.

As it is not easy to seperate a settlement once it has been initiated, it is preferable to decide how settlement should be applied, and then design the system accordingly.

::: tip E.g.
Running a single currency multi-lateral deferred net settlement model, and using test currencies to perform regular platform health tests. Would prefer to have all settlements of test currencies to be created seperately to the real currency. Would prefer not to have to select a currency or settlement model when initiating a settlement.
This can be achieved by creating seperate settlement models I.e. one for each test currency, and one for the real currency.
The default action on initiating the settelment with transaction in both currencies, would be that seperate settlements are initiated. (The determine settlement model function would find both settlement models.)
:::
