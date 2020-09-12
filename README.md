**The documentation in this repository is no longer updated, and will be deprecated on April 10th, 2020.  Contact your Client Partner (cs@apexclearing.com), or visit https://go.apexclearing.com/developer-portal-home to register for the new Apex Developer Portal!**

# Apex API Docs

This GitHub repo contains application documentation for Apex's APIs.  Swagger docs exist for each app at api.apexclearing.com/{app_name}/docs.

**Application list**

Application | Description
--- | ---
[ale](./ale) | ALE, or Apex Log Events, is the primary means for correspondents to get notifications (events) from Apex applications.
[alps](./alps) | A restful API for initiating and managing ACATS.
[atlas](./atlas) | APEX’s account management API. It is the primary way to create, read and update clearing accounts.
[braggart](./braggart) |  APEX’s system for reporting execution and allocations data.
[edocuments](./edocuments) | Web services for retrieving Customer Confirms, Account Statements and Tax Documents. **See note below regarding special considerations regarding edocs.**
[equilibrium](./equilibrium) | APEX's rebalancer API.
[hermes](./hermes) | A Java service that manages delivery of email notifications of statement and transaction confirmation (confirm) availability.
[herodotus](./herodotus) | Legacy RESTful API that supports submission of trade and other transactions via JSON. Superseded by [Braggart](/braggart).
[legit](./legit) | Apex's authentication and authorization system.
[sentinel](./sentinel) | Apex's cash management system.
[sketch](./sketch) |Apex Clearing's identity-checking system. Sketch is primarily used when opening accounts with Atlas.
[snap](./snap) | A simple image storage and retrieval service for use by Apex internal services and API clients.

### How to use this repo
Apex's applications support and build on each other. For example, legit is used to authenticate access to all other applications. Apex recommends you review _legit_, _atlas_ and _ale_ first. Then move on to other applications as they apply to you.

Each application has an FAQ doc called '[application]_faq.md' within each repo. It's a good idea to review these as well to familiarize yourself with some common issues and questions.

You may also find the [example code](../../../example-code) helpful as you work through the documentation.

### Web Service Availability

**APEX's APIs and WWW Services have been designed for reliabilty. However they can be unavailable at certain times, for example during off-hours maintenance windows (you will be notified before a maintenance window occurs). As such, it is important to design your systems to be fault tolerant and able to recover if a given service is temporarily offline.**   

#### Note on EDocuments
The EDocs services are legacy web services and do not follow the same authorization or calling conventions as the rest of APEX's API suite. The differences and considerations are described in the [edocuments](edocuments) section.
