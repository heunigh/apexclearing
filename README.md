**The documentation in this repository is no longer updated, and will be deprecated on April 10th, 2020.  Contact your Client Partner (cs@apexclearing.com), or visit https://go.apexclearing.com/developer-portal-home to register for the new Apex Developer Portal!**

# ALPS (ACAT Linear Processing System)
### Overview
The Automated Customer Account Transfer Service ("ACATS") is the National Securities Clearing Corporation's ("NSCC") central processing system for the timely transfer of customer account assets. ACATS enables participants to efficiently and automatically enter, review, and settle transfers. The service standardizes transfer procedures, reduces operating costs, and speeds transaction settlements.  The full ACAT user guide published by the NSCC can be found at [here](https://dtcclearning.com/learning/clearance/topics/acats.html).

ALPS provides a RESTFUL API to Apex correspondents for initiating ACATS. It also leverages [ALE](../ale) (Apex’s messaging system) to provide correspondents with status updates for all ACAT states from initiation to settlement.

ACATS can only be initiated by the firm that is requesting the assets, referred to as the "RECEIVING FIRM". The firm that gets the request and sends the assets is referred to as the "DELIVERING FIRM". The DELIVERING FIRM cannot initiate ACATS but will be notified of pending ACATS and ACAT state changes via ALE.
and can also see the details of the outgoing ACAT via the ALPS API.

**Resource Information**

| Response format | JSON |
| --- | --- |
| Prod Swagger docs | [https://api.apexclearing.com/alps/docs](https://api.apexclearing.com/alps/docs) |
| UAT Swagger docs | [https://uat-api.apexclearing.com/alps/docs](https://uat-api.apexclearing.com/alps/docs) |

**ALPS API**
The ALPS API allows correspondents to initiate (incoming) and track/manage (incoming and outgoing) ACAT requests. [Click here for ALPS API documentation and examples.](alps_api.md)

### Lifecycle of an ACAT
ALPS supports two kinds of ACATS: FULL or PARTIAL. A FULL ACAT is when the RECEIVING FIRM requests all assets from the customers account at the DELIVERING FIRM. A PARTIAL ACAT is when the RECEIVING FIRM requests a specific asset or list of assets from the DELIVERING FIRM. The lifecyles are different for each type of request. However, in both cases the RECEIVING FIRM initiates the transfer by submitting a Transfer Input ("TI") record, also known as a Transfer Initiation Form ("TIF").

A FULL ACAT is often referred to as the 'standard' ACAT because of it's commonality.

_**ACAT States**_
The state of an ACAT changes as it flows through the ACAT process. A typical ACAT goes through just 4 states, but additional states are possible. See [Appendix A: State Descriptions](#appendix-a-state-descriptions) for a complete list. ALPS use ALE to notify correspondents of changes to the ACAT state on the `alps-acat-status` topic.

### FULL ACAT
The workflow for a typical FULL ACAT looks like this:

![](img/ALPS_Flow_FULL_ACAT.jpg)

1.	The RECEIVING FIRM submits a TI record. ALPS validates the account and customer information. At this point the status is `REQUEST`.
2.	ACATS assigns a control number to the transfer and distributes output to both parties.
3.	The DELIVERING FIRM must respond within one business day, by either approaching and adding assets to the transfer, or by rejecting the transfer. Once the DELIVERING FIRM adds the assets, the status is set to `REVIEW`.
4.	The RECEIVING FIRM gets the response with the list of assets and can accept or reject the transfer in full.  
5.	Once the review period is complete ACATS stages the transfer for settlement. At this point, the RECEIVING FIRM and DELIVERING FIRM can no longer process any updates to the transfer. This status, `SETTLE_PREP`, lasts for one business day. At this point, ACATS issues a settlement report/file to both the DELIVERING FIRM and RECEIVING FIRM to inform them of the assets settling in the transfer. ACATS also sends the settling assets to their eligible settlement interfaces (CNS, DTC, Fund/SERV). Both parties to the transfer update their systems and prepare for the settlement of assets.
6.	The transfer settles. The transfer is now in `SETTLE_CLOSE` status. Settlement of assets begins.

#### DELIVERING FIRMS options on a FULL ACAT
The DELIVERING FIRM must respond with an approval or rejection of the TIF within 24 hours after the ACAT has been received. Rejects cannot be arbitrary - the DELIVERING FIRM can 'soft' reject the ACAT if the Social security number , account title or account number does not match, or if the account is flat. The DELIVERING FIRM can 'hard' reject if the request is a duplicate or they have received a rescind letter from the customer. Hard and soft rejects are effectively the same for ALPS clients and both require the RECEIVING FIRM to resubmit a TIF.

When accepting the ACAT, the DELIVERING FIRM adds the list of assets to the ACAT. At this point the ACAT goes into `REVIEW` status.

The DELIVERING FIRM can add additional assets when the ACAT is in review status. This can happen for example, if the customer made a trade while the ACAT was in progress. If the DELIVERING FIRM makes a change to the list of assets, the ACAT goes into `REVIEW_ADJUST` status.

**Note: The DELIVERING FIRM must approve or reject the ACAT, all-or-none and CANNOT exclude or include individual assets.**

####  RECEIVING FIRMS options on a FULL ACAT
The RECEIVING FIRM looks at the list of assets during the `REVIEW` phase and has the option of approving or rejecting the full transfer. The RECEIVING can not remove or reject single assets in a FULL transfer, they must approve or reject the entire ACAT (all-or-none).  The sole exception to the all-or-none approve or reject is for Mutual Funds. Possible reason to reject a FULL transfer is that the assets do not meet the RECEIVING FIRMS credit or margin requirements.

### PARTIAL ACAT
The workflow for a PARTIAL ACAT looks like this:

![](img/ALPS_Flow_PARTIAL_ACAT.jpg)

The primary difference in workflow between FULL and PARTIAL transfers is that the request goes to SETTLE_CLOSE immediately after the DELIVERING FIRM approves the ACAT. There is no need for the RECEIVING FIRM to review the assets, since they specifically requested the transferring assets and therefore already implied approving the transfer.

1.	The RECEIVING FIRM submits a TI record. ALPS validates the account and customer information. At this point the status of the ACAT is `REQUEST`.
2.	ACATS assigns a control number to the transfer and distributes output to both parties.
3.	The DELIVERING FIRM must respond within 24 hours and can only approve or reject the ACAT. The status is now set to `REVIEW`.
4.	The ACAT settles. The ACAT status is now `SETTLE_CLOSE`. Settlement of assets begins.

#### DELIVERING FIRM's options for a PARTIAL ACAT
As with FULL ACATS, the DELIVERING FIRM  must approve or reject a PARTIAL ACAT within 24 hours and  rejects cannot be arbitrary. In addition to the reasons above, the deliverer can reject a partial ACAT if the assets do not exist and/or if the removal of those assets would result in a credit policy violation or margin call.

**Note: The DELIVERING FIRM must approve or reject the PARTIAL ACAT, all-or-none and CANNOT exclude or include individual assets (with the exception of Mutual Funds).**

#### RECEIVING FIRM's options on a PARTIAL ACAT
Since the RECEIVING FIRM is responsible for creating the assets list, there is no review process for the RECEIVING FIRM after the ACAT is processed by the DELIVERING FIRM. The ACAT goes immediately to `SETTLE_CLOSE` after the DELIVERING FIRM's approval.

### RESIDUAL ACAT
A residual ACAT is a non-standard ACAT that is not supported directly from ALPS, but it can be relevant to APEX correspondents on occasion. Residual ACATS will only occur after a FULL ACAT has been settled.  A residual ACAT is the only type of ACAT that would be initiated by the DELIVERING FIRM.  A residual ACAT will transfer and assets received into the former account after the completion of the ACAT to the RECEIVING FIRM (i.e. dividend payout of cash or shares during or after the ACAT transfer).

### ACAT Timing
A FULL ACAT generally settles in five business days (four business days if expedited). A PARTIAL ACAT generally settles in three business days, depending upon the assets included in the transfer.

In both ACAT types, once an ACAT request has been received by the DELIVERING FIRM, the DELIVERING FIRM must approve or reject the ACAT within 24 hours.

### ACAT Cycles

ACATS are processed by the NSCC in batches at specific times, called cycles. ALPS automatically holds ACAT requests until the next valid cycle. Cycle times are as follows:

* Cycle 1 = 7:30 CST (7:30 AM CST)
* Cycle 2 = 9:30 CST (9:30 AM CST)
* Cycle 3 = 11:30 CST (11:30 AM CST)
* Cycle 4 = 13:30 CST (1:30 PM CST)
* Cycle 5 = 15:30 CST (3:30 PM CST) - EOD Cycle

**Note:  New TIF requests can only be sent in cycles 1 and 2. Any TIF requests received after 9:30 CST (9:30 AM CST) will be held until cycle 1 of the next business day.**
**Note:  ALE messages will only be returned during ACAT cycle runs.**

### Using ALPS as the RECEIVING FIRM
A TIF is created by the RECEIVING FIRM using the [`/api/v1/tif`](alps_api.md#creating-an-acat) endpoint.  An ACAT can be either FULL or PARTIAL. For a FULL ACAT, the RECEIVING FIRM must include the customer’s account information (name, social security number, account type). For a PARTIAL ACAT, the RECEIVING FIRM must also include a list of the assets to be transferred along with the customer's account information.

**Note: Only the RECEIVING FIRM can initiate an ACAT.**

### Using ALPS as the DELIVERING FIRM
Apex automatically responds to an ACAT request, on behalf of the DELIVERING FIRM, in no more than one business day.  The DELIVERING FIRM will be notified of an ACAT request via an ALE message.   To view the details of the ACAT request, you can use the [`/api/v2/acats/{acatsControlNumber}/details`](alps_api.md#reviewing-acat-details) endpoint.

**Note: Correspondents should block trading for any account for which a full ACAT message is received.**

### Internal ACAT
An internal ACAT differs from FULL and PARTIAL that it does not go to NSCC for processing.  Internal ACATS are all processed in house through journal entries within Apex systems and back office.  Things to note for an Internal ACAY submitted via ALPS:
* `acatsControlNumber` is an internal number only.  You will be able to search for it within the end points but will not be a valid reference number outside of Apex
* ALE messages that are important to know: "INITIATE_REQUST" and "COMPLETE".  All interim messages do not apply for an Internal ACAT.

### ALE Messages

ALPS writes to ALE for any status updates or changes on ACAT transfers that it receives from NSCC.  It will also provide updates for internal changes to transfers. The ALE topic for status updates is `alps-acat-status` and the partition id would the firm's correspondent code.  These messages are in JSON format. A typical ACAT would send the following sequence of payloads:

```
{"acatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"FULL_TRANSFER","currentState":"REQUEST","previousState":"UNDEFINED"}

{"acatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"FULL_TRANSFER","currentState":"REVIEW","previousState":"REQUEST"}

{"acatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"FULL_TRANSFER","currentState":"REVIEW_ACCELERATE","previousState":"REVIEW"}

{"acatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"FULL_TRANSFER","currentState":"SETTLE_PREP","previousState":"REVIEW_ACCELERATE","expectedSettlementDate":"2016-02-02T00:00:00"}

{"acatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"FULL_TRANSFER","currentState":"SETTLE_CLOSE","previousState":"SETTLE_PREP","expectedSettlementDate":"2016-02-02T00:00:00"}

{"acatsControlNumber":123456789,"associatedAcatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"RESIDUAL_CREDIT","currentState":"REVIEW","previousState":"UNDEFINED"}

{"acatsControlNumber":123456789,"associatedAcatsControlNumber":123456789,"account":"AAA123456","direction":"JOINING","transferType":"RESIDUAL_CREDIT","currentState":"SETTLE_CLOSE","previousState":"REVIEW","expectedSettlementDate":"2016-02-08T00:00:00"}
```

See [Appendix B: ALE Message Field Specifications](#appendix-b-ale-message-field-specifications) for more info about the fields in the ALE messages.

### White, Black and Gray Lists & Asset Type

ALPS allows the RECEIVING FIRM to specify a list of assets that will be used to automatically reject an ACAT. For example, you may not want to receive any ACAT requests that contain AAPL. You may also only want to receive assets that are part of a pre-defined list of ETFs. This is accomplished using White, Black and Gray lists. See [White, Black and Gray List API](alps_bwg_list.md) for a description of the API that manages these lists.

**White List**
Your white list are symbols you are willing to accept during an ACAT. If you provide a white list of acceptable to transfer symbols, any ACAT that contains a symbol **NOT INCLUDED**  in that white list, will be automatically rejected.

**Black List**
Your black list are symbols you do not want to accept during an ACAT. If you provide a black list of symbols not to accept, any ACAT that contains a symbol **INCLUDED** in the black list will be automatically rejected.

**Gray List**
Your gray list are symbols that you are willing to accept during an ACAT, but only within a specified percentage of the total Net Asset Value of the ACAT. For example, you may not want the symbol "DIRT" to represent more than 5% of a given incoming ACAT value. ALPS allows you to set the acceptable percentage limit for the list of symbols that you specify.

**Note: It is possible to specify both a white and black/gray list, specifying only one of these lists is considered ideal and preferred.**

**Asset Type**
Adding a specific asset type, will prevent any asset classified as such from transferring over.  There are two asset types that can be added here, "Options" or "Mutual Fund".  

### Appendix A: State Descriptions

| State | Description | Action |
| --- | --- | --- |
| UNDEFINED | ALPS internal state for transfer requests received via the API prior to the NSCC assigning the ACAT control number | |
| INITIATE_REQUEST | New incoming transfer submission | |
| REQUEST | Transfer submitted to NSCC successfully | |
| REQUEST_ADJUST | Previously rejected ACAT resubmitted by the contra firm | |
| REQUEST_ADJUST_PAST | The delivering firm did not respond to receiving firms request in the allotted time | If no action is taken, request will need to be re-submitted |
| REQUEST_PAST | The delivering firm did not respond to receiving firms request in the allotted time | If no action is taken, request will need to be re-submitted |
| REQUEST_REJECT | "Soft" reject by the DELIVERING firm that can be amended within 24 hours | Receiving firm can amend instructions within 24 hours |
| REVIEW |  Both the receiving firm and delivering firm review the assets expected to transfer | |
| REVIEW_ADJUST_DELIVERER | Both the receiving firm and delivering firm review the assets expected to transfer | |
| REVIEW_ERROR | The delivering firm has an error on asset input | Contra firm has 24 hours to adjust; if no action is taken, request will need to be re-submitted as it will go into "REQUEST_PAST" |
| REVIEW_ACCELERATE | The receiving firm accelerated the transfer, reducing settlement by one day | |
| REVIEW_ADJUST_RECEIVER_ACCELERATE | The receiving firm accelerated the transfer, reducing settlement by one day | |
| SETTLE_PREP | The day before settlement | Day before settlement, no adjustments can be made |
| SETTLE_CLOSE | The transfer settled and assets were transferred to the account. | |
| CLOSE_PURGE | Delivering firm did not respond to the request within the allotted time | RECEIVING FIRM will need to re-submit |
| REQUEST_500 | Internal issues with the server | Contact Apex to see if there are any issues with the system or access |
| PARTIAL_REQUEST | Partial Transfer submitted to NSCC successfully | |
| MEMO_PURGE_PARTIAL_TRANSFER_REQUEST_RECEIVER | Delivering firm did not respond to the request within the allotted time | RECEIVING FIRM will need to re-submit |
| REJECT | There was an issue with the transfer, processing was stopped | ACAT department will have to follow up with the contra firm |
| TRANSFER_COMPLETE | The transfer settled and assets were transferred.  Client can now view account and see the transferred assets. | |
| ERROR | Issue with the transfer at NSCC | Contact ACAT department for additional information |
| SYSTEM_PURGE | Assigned to transfers once NSCC notifies us the ACAT will be removed from the system | RECEIVING FIRM will need to re-submit |

### Appendix B: ALE Message Field Specifications
[ALE](alps_ale_messages.md) messages from ALPS will contain the following fields:

| Field	| Description | Values |
| --- | --- | --- |
| acatsControl<br />Number | The ACATS control number assigned by NSCC | |
| associatedAcats<br />ControlNumber | The earlier ACATS control number associated with this case | |
| accountNumber	| The account number involved in the transfer | |
| direction | The direction of the transfer | UNDEFINED, JOINING, LEAVING, JOINING_AND_LEAVING |
| transferType | The type of transfer | UNDEFINED, FAIL_REVERSAL_BROKER_TO_BROKER_ONLY, <br />FULL_TRANSFER, MUTUAL_FUND_CLEANUP, <br />PARTIAL_TRANSFER_DELIVERER, PARTIAL_TRANSFER_RECEIVER, <br />POSITION_TRANSFER_FUND_FIRM_TO_MUTUAL_FUND_COMPANY_ONLY, <br />RECLAIM, RESIDUAL_CREDIT |
| currentState | The current state of the transfer | UNDEFINED, INITIATE_REQUEST, REQUEST, <br />REQUEST_ADJUST, REQUEST_ADJUST_PAST, REQUEST_PAST, <br />REQUEST_REJECT, REVIEW, REVIEW_ADJUST_DELIVERER, <br />REVIEW_ERROR, REVIEW_ACCELERATE, <br />REVIEW_AS_JUST_RECEIVER_ACCELERATE, SETTLE_PREP, <br />SETTLE_CLOSE, CLOSE_PURGE, PARTIAL_REQUEST, <br />MEMO_PURGE_PARTIAL_TRANSFER_REQUEST_RECEIVER, <br />REJECT, SYSTEM_PURGE, TRANSFER_COMPLETE, ERROR |
| previousState | The previous state of the transfer | UNDEFINED, INITIATE_REQUEST, REQUEST, <br />REQUEST_ADJUST, REQUEST_ADJUST_PAST, REQUEST_PAST, <br />REQUEST_REJECT, REVIEW, REVIEW_ADJUST_DELIVERER, <br />REVIEW_ERROR, REVIEW_ACCELERATE, <br />REVIEW_AS_JUST_RECEIVER_ACCELERATE, SETTLE_PREP, <br />SETTLE_CLOSE, CLOSE_PURGE, PARTIAL_REQUEST, <br />MEMO_PURGE_PARTIAL_TRANSFER_REQUEST_RECEIVER, <br />REJECT, SYSTEM_PURGE, TRANSFER_COMPLETE, ERROR |
| tifId | The unique id assigned by Alps when a transfer is initiated | |
| expected<br />Settlement<br />Date | The date when the transfer is expected to settle | |
| notes | Any additional comments/notes on the transfer. | |
