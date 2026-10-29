# Phase 1: Financial System Fundamentals
------------------------------------------------------------------------

## Table of Contents

1.  [How to Use This Guide](#1-how-to-use-this-guide)
2.  [Money, Ledgers, and Financial
    Claims](#2-money-ledgers-and-financial-claims)
3.  [The Three Layers of a Payment](#3-the-three-layers-of-a-payment)
4.  [Financial System Participants](#4-financial-system-participants)
5.  [Bangladesh's Payment Ecosystem](#5-bangladeshs-payment-ecosystem)
6.  [Banking and Financial
    Operations](#6-banking-and-financial-operations)
7.  [Financial Terminology Glossary](#7-financial-terminology-glossary)
8.  [Accounting Preview for Software
    Engineers](#8-accounting-preview-for-software-engineers)
9.  [How Financial Concepts Shape Backend
    Engineering](#9-how-financial-concepts-shape-backend-engineering)
10. [Failure Scenarios and Recovery](#10-failure-scenarios-and-recovery)
11. [Exercises and Assessment](#11-exercises-and-assessment)
12. [Progressive Projects](#12-progressive-projects)
13. [Three-Week Study Plan](#13-three-week-study-plan)
14. [Completion Checklist](#14-completion-checklist)
15. [Glossary Quick Reference](#15-glossary-quick-reference)
16. [References and Verification
    Sources](#16-references-and-verification-sources)

------------------------------------------------------------------------

# 1. How to Use This Guide

The goal of this phase is to develop a reliable mental model of the
financial system before studying advanced ledger engines, payment
orchestration, distributed systems, and MFS architecture.

For every topic, ask four questions:

1.  **Business:** What is the real-world activity?
2.  **Financial:** Who owes whom money or value before and after the
    activity?
3.  **Operational:** Which institutions, teams, and processes are
    involved?
4.  **Technical:** Which APIs, services, database records, controls, and
    reports represent the activity?

Do not assume that a successful HTTP response proves that money has
settled. Do not assume that a wallet balance is the same thing as the
cash or bank balance backing the institution's obligations. Do not
assume that every institution or payment rail follows the same operating
or accounting model.

### Suggested study method

-   Read one small topic at a time.
-   Write the key definitions in your own words.
-   Draw the participant and transaction flow.
-   Work through the numerical examples manually.
-   Explain the workflow aloud as if onboarding a new engineer.
-   Implement a small simulation after understanding the business flow.
-   Record questions that require confirmation from your company's
    finance, operations, risk, or compliance teams.

### Learning outcomes

By the end of this phase, you should be able to:

-   Explain the difference between money, a financial claim, a ledger
    record, and settlement.
-   Identify the main participants in a payment ecosystem.
-   Describe common MFS workflows.
-   Use foundational payment terminology precisely.
-   Explain why accounting, reconciliation, idempotency, and
    transaction-state management matter.
-   Connect a business transaction to APIs, database changes, and
    operational controls.
-   Identify Bangladesh-specific regulatory or infrastructure claims
    that need official verification.

------------------------------------------------------------------------

# 2. Money, Ledgers, and Financial Claims

## 2.1 What is money?

Money is commonly used as a **medium of exchange**, a **unit of
account**, and a **store of value**.

-   **Medium of exchange:** People use it to buy goods and services.
-   **Unit of account:** Prices, debts, and financial records are
    expressed in it.
-   **Store of value:** It allows purchasing power to be carried from
    one point in time to another, although inflation and other factors
    can change that purchasing power.

Modern financial systems represent much of the money people use through
electronic records. A customer's bank account or MFS wallet typically
displays a balance maintained by a financial institution's systems.

The balance is not merely an arbitrary number. It represents a financial
relationship. For example, a customer wallet balance is generally a
claim by the customer against the wallet issuer under the relevant legal
and operating model. The issuer must maintain records and arrangements
that support its obligations.

### Physical cash, bank deposits, and wallet balances

  -----------------------------------------------------------------------
  Form                    What the user holds or  Important distinction
                          sees                    
  ----------------------- ----------------------- -----------------------
  Physical cash           Banknotes and coins     Can be transferred
                                                  physically without
                                                  updating a payment API

  Commercial bank deposit A balance recorded by a Generally represents
                          commercial bank         the bank's liability to
                                                  the depositor

  MFS wallet balance      An electronic balance   Generally represents
                          recorded by an MFS      the provider's
                          provider                obligation to the
                                                  wallet customer under
                                                  the applicable model

  Central bank money      Physical currency and   Used in central-bank
                          eligible balances held  settlement
                          at the central bank     arrangements, subject
                                                  to system access and
                                                  rules
  -----------------------------------------------------------------------

A bank deposit and an MFS wallet balance can both be represented
electronically, but their legal structure, permitted uses, safeguarding
arrangements, and settlement processes can differ.

### Example: buying a product with a wallet

A customer pays BDT 500 to a merchant using an MFS wallet.

At the customer interface, the visible event may be "Payment
successful." Behind that message, systems may have to:

1.  Authenticate the customer.
2.  Check the customer's account status and limits.
3.  Verify available balance.
4.  Create a transaction record.
5.  Record the financial effect in the relevant ledger.
6.  Record a merchant payable or credit the merchant's wallet, depending
    on the model.
7.  Notify the merchant and customer.
8.  Include the transaction in operational reports and reconciliation.
9.  Settle the merchant or relevant institution according to the
    contractual arrangement.

The user-facing event, internal accounting event, and later settlement
event are related, but they are not necessarily the same event or
timestamp.

## 2.2 Assets, liabilities, and financial claims

A simple way to start understanding financial statements is:

-   **Asset:** Something the entity owns or has a right to receive.
-   **Liability:** Something the entity owes to another party.
-   **Equity:** The residual interest after liabilities are subtracted
    from assets.

The accounting equation is:

**Assets = Liabilities + Equity**

Suppose, for illustration, an MFS provider has BDT 10,000,000 in an
eligible bank account and owes customers BDT 9,000,000 in wallet
balances. This simplified picture does not establish that the provider
is compliant or fully funded; actual arrangements can include other
assets, liabilities, restrictions, safeguarding rules, and accounting
classifications. It does illustrate that customer wallet balances are
obligations and are not the same thing as the bank-account asset.

### Why this matters to software engineers

A wallet table may contain a `balance` field, but the existence of that
field does not itself explain:

-   The financial account represented by the balance.
-   Which journal entries changed it.
-   Whether the balance is posted, available, or subject to a hold.
-   Whether the related institution has settled.
-   Whether the value is reconciled to independent evidence.
-   How a correction should be recorded and audited.

In a robust financial design, balances should be explainable from
authoritative financial records and controlled business rules.

## 2.3 How does money move?

### Scenario A: Customer sends money to another customer in the same MFS

Assume Customer A sends BDT 300 to Customer B.

At the customer-ledger level, the platform may reduce A's wallet
liability and increase B's wallet liability by BDT 300. If both wallets
are on the same platform, there may be no immediate bank-to-bank
transfer for this individual P2P payment. The platform still needs a
valid, atomic internal financial record.

Conceptual customer subledger view:

  Account                           Debit    Credit
  ----------------------------- --------- ---------
  Customer A wallet liability         300       ---
  Customer B wallet liability         ---       300
  **Total**                       **300**   **300**

This is a conceptual illustration, not a complete general-ledger posting
specification. The organization's accounting policy determines the full
chart-of-accounts treatment.

### Scenario B: Customer cashes out through an agent

The customer requests BDT 1,000 in cash. The platform may debit the
customer's wallet and credit the agent's electronic float, while the
agent physically gives BDT 1,000 to the customer. The electronic ledger
transfer and physical cash handover are two connected events.

The agent could have enough electronic float but not enough banknotes.
Conversely, the agent might have enough cash but insufficient electronic
float. Operational liquidity management must consider both.

### Scenario C: Bank-to-wallet transfer

A customer requests a transfer from a bank account into an MFS wallet.
The systems may need to coordinate a debit at the bank, a credit at the
MFS provider, partner messages, status inquiries, and reconciliation.
The customer may see a pending state while the two institutions
determine the outcome.

Do not assume that a bank debit automatically proves that the wallet was
credited, or that a timeout proves that the bank debit did not occur.
The integration contract and recovery workflow determine how the result
is confirmed.

## 2.4 Messaging, accounting, and settlement are different records

A single payment can create multiple records:

-   **Message record:** What was requested and sent to a partner.
-   **Transaction record:** The business operation and its current
    state.
-   **Ledger record:** The financial effect recognized by the system.
-   **Settlement record:** The obligation or transfer between
    institutions.
-   **Reconciliation record:** The comparison between independent
    sources of evidence.

These records need stable references and traceable relationships. They
should not be treated as interchangeable representations of a single
status field.

## 2.5 Financial invariants

A financial invariant is a property that must remain true under the
system's defined rules.

Examples:

1.  Every posted journal balances according to the applicable accounting
    model.
2.  A retry must not create a second financial effect for the same
    logical request.
3.  A debit must obey the account's balance and limit rules.
4.  A posted financial record must remain traceable.
5.  A reversal or correction must preserve the history of the original
    event.
6.  Reconciliation must be able to detect mismatches between internal
    and independent records.
7.  Concurrent operations must not bypass limits or spend the same
    available balance twice.

These rules should be reflected in business logic, database constraints,
transaction boundaries, tests, monitoring, and operational procedures.

------------------------------------------------------------------------

# 3. The Three Layers of a Payment

## 3.1 Layer A: Messaging and authorization

This layer answers: **What was requested, and is the request
permitted?**

Typical activities include:

-   Receiving a payment request.
-   Authenticating the caller or customer.
-   Validating request fields.
-   Checking account status and transaction limits.
-   Checking available balance.
-   Applying risk controls.
-   Creating a transaction reference.
-   Sending a request to a partner.
-   Receiving a response or callback.
-   Handling a timeout.
-   Querying the status of an uncertain request.

Authorization means that a request is allowed under the relevant rules.
Depending on the design, authorization may create a hold or reservation,
or it may be closely coupled with posting. Authorization is not
universally the same as final posting.

### Example

A merchant payment API receives a request to pay BDT 750. The API
validates the merchant, customer, amount, and idempotency key. It then
authorizes the payment according to its rules. The request may be
accepted for processing even if the final outcome is not yet known.

An API response such as `202 Accepted` usually means that processing has
been accepted, not that every financial and settlement step is complete.

## 3.2 Layer B: Accounting and ledger

This layer answers: **What financial effect has been recognized, and how
is it recorded?**

It can include:

-   Account identifiers.
-   Journal entries.
-   Debit and credit lines.
-   Posted transaction references.
-   Holds and releases.
-   Fee and commission entries.
-   Reversals and adjustments.
-   Balance calculations.
-   Audit history.

A ledger should provide a traceable explanation of how a balance was
formed. A transaction table alone may not be sufficient to provide that
accounting trail.

### Important distinction

The operational transaction status and ledger posting status may need
separate representation. For example, a transaction could be awaiting an
external confirmation while a reservation is active, or an externally
successful transaction could require a controlled local posting/recovery
step. The correct design depends on the transaction type and integration
contract.

## 3.3 Layer C: Clearing and settlement

This layer answers: **How are obligations between institutions
calculated, confirmed, and discharged?**

-   **Clearing** generally involves exchanging, validating, and
    calculating payment instructions or obligations.
-   **Settlement** is the discharge of those obligations through the
    relevant settlement arrangements.
-   **Reconciliation** compares records to determine whether the
    parties' views agree.

The precise sequence varies by payment system. Some systems combine or
tightly couple steps; others separate them by batch, cut-off, or
settlement cycle.

### Comparison

  -------------------------------------------------------------------------------------
  Question          Messaging/authorization   Accounting/ledger   Clearing/settlement
  ----------------- ------------------------- ------------------- ---------------------
  Main concern      Is this request valid and What financial      What do institutions
                    allowed?                  effect is recorded? owe and how is it
                                                                  discharged?

  Typical evidence  API request, response,    Journal entries and Settlement report,
                    signed callback           account records     clearing file, bank
                                                                  statement

  Common failure    Timeout, invalid          Duplicate posting,  Missing settlement,
                    signature, rejected       incorrect balance,  amount mismatch,
                    request                   failed commit       delayed settlement

  Main controls     Authentication,           Atomicity, balanced Reconciliation,
                    validation, idempotency   journals,           settlement
                                              concurrency         monitoring, exception
                                              controls            handling
  -------------------------------------------------------------------------------------

## 3.4 Example timeline: a payment through a partner

``` mermaid
sequenceDiagram
    participant C as Customer App
    participant M as MFS Platform
    participant L as Ledger
    participant P as Partner/Bank
    participant S as Settlement System
    participant R as Reconciliation

    C->>M: Initiate payment
    M->>M: Authenticate and validate
    M->>L: Reserve or post according to flow
    M->>P: Send partner request
    P-->>M: Accepted / pending / final response
    M->>L: Apply permitted financial transition
    M-->>C: Return transaction status
    P->>S: Participate in clearing/settlement process
    S-->>P: Settlement result or statement
    P-->>R: Partner report
    L-->>R: Internal ledger records
    R->>R: Match records and raise exceptions
```

This diagram is illustrative. Actual systems may use asynchronous
callbacks, separate settlement operators, different ledger timing, or
additional controls.

### Learning exercise

For each payment type you work with, draw three separate flows:

1.  Message flow.
2.  Financial posting flow.
3.  Clearing and settlement flow.

Mark which events are synchronous, asynchronous, final, provisional, or
unknown.

------------------------------------------------------------------------

# 4. Financial System Participants

Regulatory categories and permissions depend on the entity and the
framework in force. The examples below are educational. Confirm current
status, licensing, and permitted activities through official sources and
your company's compliance team.

## 4.1 Central bank

A central bank typically has responsibilities related to monetary
policy, currency issuance, financial stability, supervision, and
payment-system oversight. Its exact responsibilities and the services it
provides are defined by law and applicable arrangements.

In Bangladesh, Bangladesh Bank is the central bank. For a fintech
engineer, it is a primary source for official circulars, payment-system
information, and relevant regulatory documents.

### Why engineers care

-   Regulations can create mandatory product and control requirements.
-   Payment infrastructure has participation and operating rules.
-   Settlement arrangements affect transaction design and
    reconciliation.
-   Reporting obligations can influence data models and retention.
-   Changes in official requirements may require coordinated
    engineering, operations, and compliance work.

Do not infer that every retail payment is individually settled in
central bank money. Settlement models vary.

## 4.2 Commercial banks

Commercial banks commonly take deposits, lend, maintain customer
accounts, and provide payment services under their permissions. They may
participate in payment systems and hold settlement accounts.

An MFS provider may integrate with a bank for bank-to-wallet transfers,
wallet-to-bank transfers, settlement, funding, disbursement, or other
services.

### Engineering implications

-   Partner-specific authentication and message formats.
-   Bank-side transaction references.
-   Status inquiry and callback workflows.
-   Cut-off and holiday handling.
-   Reconciliation against statements or partner files.
-   Careful handling of uncertain outcomes.

## 4.3 Non-bank financial institutions (NBFIs)

NBFIs provide financial services under their applicable regulatory
permissions but should not be assumed to have all the powers or services
of a commercial bank. Their activities and relationship to payment
services vary.

For engineering work, identify the actual legal entity, its authorized
activity, and the contractual integration rather than relying on a
generic category label.

## 4.4 Payment Service Providers (PSPs)

A PSP is an organization authorized under the relevant framework to
provide specified payment services. The permitted services and exact
classification depend on current regulations and the organization's
authorization.

A PSP may provide payment-related services to customers, merchants, or
other institutions. The details of custody, settlement, onboarding, and
technical responsibilities vary by business model.

**Engineering implication:** the regulatory permissions and contractual
role of the organization determine which workflows it may operate, what
records it must retain, and which controls apply.

## 4.5 Payment System Operators (PSOs)

A PSO generally operates or manages payment-system infrastructure or
services under its relevant authorization. This role is different from
running a consumer-facing wallet.

Infrastructure operators may define participation rules, message
requirements, operational standards, certification processes, and
availability requirements. Verify the current scope and role of each
Bangladesh system through official documentation.

## 4.6 MFS providers

An MFS provider offers mobile financial services within its authorized
business and regulatory model. Services may include wallet-based
transfers, agent-assisted transactions, merchant payments,
disbursements, collections, and integrations with other institutions.

Illustrative Bangladesh market names include bKash, Nagad, Upay, and
Rocket. Do not infer current licensing details, ownership structure,
product availability, or specific capabilities from a brand name alone.

### Common software domains

-   Customer onboarding and identity.
-   Wallet/account management.
-   Transaction orchestration.
-   Ledger and balance management.
-   Agent and merchant management.
-   Fee and commission calculation.
-   Partner integrations.
-   Risk and transaction limits.
-   Reconciliation and settlement.
-   Customer notifications and support.
-   Audit, monitoring, and regulatory reporting.

## 4.7 Payment aggregators

An aggregator may connect multiple merchants through a shared
integration and may provide onboarding, payment collection, routing,
reporting, or settlement allocation services.

Important concepts include:

-   Merchant and sub-merchant identifiers.
-   Merchant verification.
-   Fee allocation.
-   Settlement split rules.
-   Refund ownership.
-   Dispute responsibilities.
-   Reconciliation by merchant and transaction.

An aggregator is not necessarily the same as a gateway, acquirer, PSP,
or settlement institution. The actual role depends on its contracts and
authorization.

## 4.8 Payment gateways

A payment gateway is commonly a technical service that connects a
merchant checkout to payment methods or payment processors. It may
support hosted checkout, APIs, redirects, callbacks, payment status
inquiries, and refund requests.

A gateway does not necessarily hold customer funds or itself settle
money. The underlying payment method, acquiring institution, MFS
provider, or other authorized party may perform those functions.

## 4.9 Payment switches

A switch routes payment messages among participants according to network
rules. Depending on the system, it may validate message structure, route
requests, return responses, support interoperability, or participate in
clearing processes.

A switch's message routing role should not be confused with the
institution's ledger or with settlement itself.

Relevant message standards may include:

-   **ISO 8583:** Commonly associated with card and certain
    transaction-message environments.
-   **ISO 20022:** A family of financial messaging standards with
    structured business concepts and message definitions.

Neither standard should be assumed to be used by every local payment
rail. Check the specific system's technical documentation.

## 4.10 Card schemes, issuers, and acquirers

In a simplified four-party card model:

-   **Cardholder:** Uses a payment card.
-   **Merchant:** Accepts the payment.
-   **Issuer:** Provides the card or payment account to the cardholder.
-   **Acquirer:** Provides acquiring services to the merchant.
-   **Card scheme/network:** Defines rules and supports message routing
    and network services.

A card payment commonly involves authorization, clearing, settlement,
and potentially disputes or chargebacks. These stages are distinct.

**PCI DSS** may apply where payment-card data is stored, processed, or
transmitted, or where the environment falls within the applicable scope.
Confirm scope with qualified security and compliance teams.

## 4.11 Merchants and sub-merchants

A merchant accepts payments for goods or services. A sub-merchant may
operate through an aggregator or another intermediary.

Important records can include merchant identifiers, payment
transactions, fees, refunds, settlement amounts, and reconciliation
exceptions.

The merchant's displayed payment status, its receivable, and the final
settlement to its bank account may be different records with different
timing.

## 4.12 Agents and distributors

MFS agents provide cash-in and cash-out services under the relevant
operating model. Distributors may support agent liquidity or network
operations.

Distinguish:

-   **Physical cash:** Banknotes the agent can hand to a customer.
-   **Electronic float:** The agent's electronic value or account
    position used to provide services.
-   **Commission receivable/payable:** A separate financial amount
    earned or owed under the commission rules.

An agent with BDT 20,000 electronic float and BDT 2,000 physical cash
may be unable to satisfy a BDT 5,000 cash-out despite having enough
float. Another agent may have BDT 30,000 physical cash but only BDT
1,000 float, limiting its ability to complete cash-in operations under a
model where cash-in consumes agent float.

## 4.13 Customers, payers, and beneficiaries

The payer is the party funding a payment. The payee is the party to whom
payment is made. The beneficiary is the party intended to receive the
benefit. The account holder and transaction initiator may be different
people or organizations.

Systems need clear definitions because identity, consent, limits, and
fraud controls may depend on the role being performed.

## 4.14 Billers and collection organizations

A biller receives payments for services such as utilities, telecom,
education, or other obligations. A bill-payment workflow may include
bill inquiry, reference validation, payment submission, biller
confirmation, and reconciliation.

A payment platform's successful request response is not always identical
to the biller's final record of the bill being paid. The contract and
status inquiry process determine how confirmation is established.

## 4.15 Government payment organizations

Government-related flows may include:

-   **G2P:** Government-to-person disbursement.
-   **P2G:** Person-to-government payment or collection.

These flows can require bulk processing, beneficiary validation,
reporting, reconciliation, and specific compliance controls. Use current
official program documentation for actual operational details.

## 4.16 Settlement and correspondent banks

A settlement bank may hold accounts through which funds are moved
between institutions or merchants. Correspondent banking can enable
institutions to access services or currencies through relationships with
other banks.

Relevant engineering concerns include settlement instructions, statement
ingestion, cut-off times, reference matching, and exception resolution.

Terms such as **nostro** and **vostro** describe account relationships
from different institutions' perspectives. Learn them when working with
correspondent banking or cross-border payments; they are not required in
every domestic MFS flow.

## 4.17 Participant relationship map

``` mermaid
flowchart LR
    Customer --> MFS[MFS Provider]
    Agent --> MFS
    Merchant --> MFS
    Merchant --> Gateway[Payment Gateway]
    Gateway --> PSP[Payment Service Provider / Processor]
    PSP --> Bank[Commercial Bank]
    MFS --> Bank
    MFS --> Switch[Payment Switch / Rail]
    Bank --> Switch
    Switch --> Settlement[Settlement Arrangement]
    Biller[Biller / Collection Organization] --> MFS
    Government[Government Organization] --> MFS
    Settlement --> Bank
    CentralBank[Central Bank / Oversight / Eligible Settlement Services] -. applicable role .-> Settlement
```

This is a conceptual map, not a claim that every transaction uses all
participants or that every arrow represents a direct legal or technical
relationship.

### Exercise

Choose one transaction type used by your company. List every actual
participant and label each relationship as one of the following:

-   Customer-facing service.
-   Technical message route.
-   Financial account relationship.
-   Contractual relationship.
-   Clearing relationship.
-   Settlement relationship.
-   Regulatory or oversight relationship.

------------------------------------------------------------------------

# 5. Bangladesh's Payment Ecosystem

This section is a research framework as well as an introduction.
Payment-system capabilities, names, participation rules, operating
arrangements, and regulatory requirements can change. Confirm current
information through Bangladesh Bank and the relevant system's official
documentation.

## 5.1 Bangladesh Bank

Start with the official Bangladesh Bank website and locate the latest
applicable materials on payment systems, MFS, payment services,
settlement, KYC/e-KYC, AML/CFT, and reporting. Record publication dates
and effective dates where available.

Do not rely only on blog posts, vendor summaries, or old presentations
when deciding what a regulation requires.

## 5.2 BEFTN

The Bangladesh Electronic Funds Transfer Network is associated with
electronic funds transfer processing. When researching it, establish
from current official materials:

-   Supported transfer types.
-   Participants and access model.
-   Processing and clearing cycles.
-   Settlement arrangements.
-   File or message specifications.
-   Cut-off times and exception handling.
-   How returned or rejected items are represented.

Do not assume that every transfer is instant or that all institutions
use identical interfaces.

## 5.3 NPSB

The National Payment Switch Bangladesh is a payment infrastructure
reference point for interoperable payment services. Verify its currently
supported channels, participants, operating model, and technical
requirements using official sources.

Questions for an engineer to answer from the official documentation:

-   Which transaction types are supported?
-   What roles do participating institutions perform?
-   What are the message and response requirements?
-   How are timeouts and reversals handled?
-   What evidence is available for reconciliation?
-   Which parts of the transaction are real-time versus later
    settlement?

## 5.4 RTGS

Real Time Gross Settlement systems settle eligible transfers
individually in real time, subject to system rules and participant
access. Verify Bangladesh's current RTGS service scope, operating hours,
eligibility, and technical rules from official sources.

Do not assume that every retail MFS payment directly uses RTGS. A
payment system may use a different settlement arrangement or participate
in a separate clearing process.

## 5.5 Automated cheque processing

Research Bangladesh's automated cheque-processing arrangements,
including the official name, scope, and current operating model.
Distinguish cheque image processing and clearing from the settlement
that discharges the resulting obligations.

## 5.6 Bangla QR

Research the official Bangla QR standard and implementation materials.
Confirm supported use cases, participant roles, interoperability rules,
merchant requirements, and transaction flow.

A QR code is a way to represent payment information; it does not itself
settle money. The payment method and participating institutions
determine the actual transaction flow.

## 5.7 Binimoy and interoperability initiatives

For Binimoy and any other interoperability initiative, verify the
current status, supported services, participation model, and operating
availability through official sources. Do not assume that a previously
announced service remains available or supports every institution and
transaction type.

## 5.8 Bank-to-MFS and MFS-to-bank transfers

Document the exact arrangement used by your organization. Questions to
ask internally:

-   Which institution initiates the debit?
-   Which institution confirms the debit?
-   What reference uniquely identifies the transfer?
-   What does an immediate success response mean?
-   What happens if the response is lost?
-   Is a status inquiry available?
-   How are callbacks authenticated and deduplicated?
-   What independent file or statement supports reconciliation?
-   How are unmatched transactions escalated?

## 5.9 Research worksheet

For each rail or initiative, fill in:

  Field                                      Research notes
  ------------------------------------------ ----------------
  Official name                              
  Issuing or operating authority             
  Official documentation URL                 
  Last verified date                         
  Supported transaction types                
  Participants and roles                     
  Processing model                           
  Clearing arrangement                       
  Settlement arrangement                     
  Message/interface standard                 
  Reversal or return handling                
  Reconciliation evidence                    
  Open questions for compliance/operations   

------------------------------------------------------------------------

# 6. Banking and Financial Operations

For each operation, distinguish the business event from its technical
and accounting representation. The following examples are conceptual;
the actual journal entries depend on the institution's legal model,
contracts, and accounting policy.

## 6.1 Customer onboarding and KYC

A typical conceptual workflow:

1.  Customer submits registration information.
2.  The system validates required fields.
3.  Identity information is checked using approved methods.
4.  Screening and risk classification are performed as required.
5.  The application is accepted, rejected, or referred for review.
6.  An account is created or activated according to policy.
7.  Permitted services and limits are applied.
8.  Relevant events and evidence are logged.

KYC is not simply a form submission. The system needs to distinguish
submitted, verified, rejected, expired, and review-required states as
appropriate.

### Engineering implications

-   Keep access to identity data restricted.
-   Record verification outcomes and relevant references.
-   Do not expose sensitive identity information in logs.
-   Make account activation dependent on the required checks.
-   Support review and audit processes.
-   Confirm data retention and deletion requirements with compliance.

## 6.2 Electronic KYC

Electronic KYC may use digital identity information, approved
verification sources, biometrics, document checks, or other methods
permitted by the applicable framework.

The exact requirements and approved methods must be verified from
current Bangladesh-specific official guidance and the institution's
compliance team.

Engineering concerns include vendor availability, failed verification,
retries, consent, sensitive-data handling, evidence retention, and
escalation for manual review.

## 6.3 Customer due diligence and risk classification

Customer due diligence helps an institution understand its customer and
assess relevant risks. Risk classification can inform monitoring, review
frequency, and controls under the applicable policy.

Avoid hard-coding assumptions about legal limits or customer tiers into
scattered business logic. Use controlled configuration, effective dates,
auditability, and compliance-approved rules.

## 6.4 Account lifecycle

Possible account states include pending, active, restricted, suspended,
frozen, dormant, and closed. These are examples, not a universal status
list.

A restriction may block debits, credits, or particular transaction types
depending on its reason and governing rules. Account closure must
account for remaining balances, pending transactions, unresolved
disputes, holds, and reporting obligations.

A status change is not necessarily a financial transaction, but it can
affect whether later transactions are allowed.

## 6.5 Cash-in

Illustrative flow:

1.  Customer gives physical cash to the agent.
2.  Agent and customer details are validated.
3.  Agent float and applicable limits are checked.
4.  The platform authorizes the operation.
5.  The customer's wallet is credited and the agent's electronic
    position is reduced according to the operating model.
6.  The system records the transaction and relevant fee/commission.
7.  Customer and agent receive confirmation.
8.  The transaction is included in reports and reconciliation.

Example: customer gives the agent BDT 1,000. Ignoring fees for the
moment, the customer receives BDT 1,000 in wallet value. The agent has
BDT 1,000 less electronic float and BDT 1,000 less physical cash. The
exact institutional accounting is more detailed than this operational
description.

## 6.6 Cash-out

Illustrative flow:

1.  Customer requests a withdrawal.
2.  The system validates customer, agent, limits, and balance.
3.  The transaction is authorized.
4.  Customer wallet value is debited and agent float is credited
    according to the model.
5.  The agent gives physical cash to the customer.
6.  Fees and commissions are handled according to approved rules.
7.  The transaction is confirmed and reconciled.

A digital debit and a physical cash handover are related but distinct
events. Operational procedures should handle cases where one occurs and
the other is disputed or incomplete.

## 6.7 Person-to-person transfer

Example: Customer A sends BDT 300 to Customer B and pays a BDT 5 fee.

If the fee is charged to the sender, the total sender debit may be BDT
305 while the recipient receives BDT 300. The BDT 5 fee must be
allocated according to the fee model; it should not be silently mixed
into the principal amount.

A robust flow includes:

-   Sender and recipient validation.
-   Account-status checks.
-   Available-balance and limit checks.
-   Fee calculation.
-   Idempotency.
-   Atomic financial posting.
-   Notifications.
-   Status inquiry and audit trail.

## 6.8 Merchant payment

Example: a customer pays a merchant BDT 1,000. If a merchant discount
rate applies, the merchant's net receivable may be less than BDT 1,000,
depending on the contract and accounting model.

The system must distinguish the customer's payment amount, the fee or
discount, any tax, the merchant payable, and the amount eventually
settled. Do not assume that the customer, merchant, and operator each
see the same amount as their financial result.

## 6.9 Utility bill payment

Typical workflow:

1.  Customer enters a bill reference.
2.  Platform queries or validates the bill with the biller.
3.  Customer confirms the payment.
4.  Platform submits the payment request.
5.  Biller returns a result or processes it asynchronously.
6.  Platform records the outcome and provides a receipt or pending
    status.
7.  Internal and biller records are reconciled.
8.  Exceptions are investigated and resolved under controlled
    procedures.

A payment provider's success response and a biller's final account
update may not always be simultaneous. Design for status inquiry and
reconciliation.

## 6.10 Salary and bulk disbursements

Bulk payment processing needs batch-level and item-level status. One
file may contain thousands of beneficiaries, and a subset may fail
validation or processing.

Consider: - File identity and duplicate-upload protection. - Beneficiary
validation. - Batch authorization. - Item-level idempotency. - Partial
success. - Retry rules. - Funding checks. - Reports and
reconciliation. - Privacy and access controls.

## 6.11 Government payments

G2P and P2G workflows may involve beneficiary lists, program-specific
rules, payment batches, identity checks, government reporting, and
reconciliation. Always verify actual program requirements from
authoritative program documentation.

## 6.12 Bank-to-MFS and MFS-to-bank transfers

A transfer may involve separate source and destination systems. Design
the integration around the documented protocol:

-   Validate source and destination.
-   Use a stable end-to-end reference.
-   Authenticate and sign requests as required.
-   Handle synchronous and asynchronous responses.
-   Support status inquiry if available.
-   Deduplicate callbacks.
-   Preserve all attempts and responses.
-   Reconcile internal transactions with partner and bank records.

## 6.13 Remittances

Remittance flows may involve a sender, an authorized remittance
provider, a payout institution, a bank or MFS provider, and a
beneficiary. Cross-border flows can add currency conversion,
exchange-rate disclosure, screening, reporting, and additional
regulatory requirements.

Do not assume that every MFS provider can directly receive every type of
international remittance. Verify the specific authorized channel and
partner arrangement.

## 6.14 Reversals, refunds, adjustments, and chargebacks

These terms are not interchangeable.

-   **Reversal:** A compensating action that offsets a previous
    transaction according to defined rules, often used for failed or
    incorrectly completed operations.
-   **Refund:** A return of value after a completed payment, often
    initiated for a business reason.
-   **Adjustment:** A controlled correction to address a discrepancy or
    approved accounting change.
-   **Chargeback:** A dispute-related process used in certain payment
    systems, especially card ecosystems, under scheme rules.

Use new, linked financial records rather than deleting a posted record.
Preserve the original event, reason, approval, references, and resulting
entries.

## 6.15 Transaction limits and velocity

Limits may be defined by regulation, product policy, risk policy,
partner contract, or internal controls.

Examples include: - Maximum amount per transaction. - Aggregate amount
per day or month. - Maximum number of transactions in a time window. -
Agent or merchant limits. - Risk-based restrictions.

The engineering challenge is enforcing limits under concurrency. Two
requests that individually pass a check may collectively exceed the
limit if checks and updates are not coordinated atomically.

## 6.16 Fees, commissions, and taxes

Keep principal, fee, commission, and tax conceptually separate.

Example: a customer sends BDT 1,000 and pays a BDT 10 fee. The sender's
total debit may be BDT 1,010. The recipient may receive BDT 1,000. The
BDT 10 may be revenue, a payable, tax-inclusive value, or shared revenue
according to the actual agreement and accounting rules.

Fee rules should be auditable and versioned. Store enough information to
explain how the fee was calculated, including the applicable rule
version where appropriate.

## 6.17 Daily reconciliation and settlement

Reconciliation compares independent records, such as internal
transaction data against partner files or bank statements. Settlement is
the process by which eligible obligations are discharged under the
relevant arrangements.

A reconciliation engine should identify: - Missing internal
transactions. - Missing partner transactions. - Amount mismatches. -
Duplicate records. - Status mismatches. - Unexpected fees. -
Late-arriving records. - Unsettled obligations.

Do not silently modify transaction states or balances to make reports
match. Create traceable exceptions, investigate the cause, and apply
approved corrections.

------------------------------------------------------------------------

# 7. Financial Terminology Glossary

  -----------------------------------------------------------------------
  Term                    Definition              Engineering implication
  ----------------------- ----------------------- -----------------------
  Principal               Amount being            Store separately from
                          transferred before      fee components
                          separate fees or taxes  

  Transaction amount      Amount defined by the   Document semantics in
                          specific transaction    APIs
                          contract; clarify       
                          whether it includes     
                          fees                    

  Gross amount            Amount before specified Needed for transparent
                          deductions              reporting

  Net amount              Amount after specified  Define which deductions
                          deductions or additions apply

  Fee                     Amount charged for a    Use a separate
                          service                 component and account
                                                  mapping

  Commission              Compensation paid or    Track recipient,
                          payable to an agent or  calculation, and
                          partner                 settlement

  Merchant discount rate  Merchant-facing fee     Do not assume the
  (MDR)                   rate or charge under a  entire amount is
                          contract                operator revenue

  Ledger balance          Balance derived from    Must be traceable to
                          posted ledger activity  entries
                          under the account's     
                          rules                   

  Available balance       Amount currently usable Must be calculated or
                          after holds and other   maintained consistently
                          restrictions            

  Hold/reservation        Amount earmarked for a  Requires release,
                          possible or pending     capture, expiry, and
                          operation               audit rules

  Authorization           Decision that a request Does not always mean
                          is permitted            final posting

  Posting                 Recording a financial   Needs controlled
                          effect in the ledger    atomicity and
                                                  traceability

  Debit/credit            Accounting entry sides; Never assume debit
                          meaning depends on      always means decrease
                          account type            

  Journal entry           Group of accounting     Should balance under
                          lines recording an      the applicable model
                          event                   

  Subledger               Detailed records for a  Reconcile to the
                          class of accounts or    general ledger where
                          obligations             applicable

  General ledger          Main accounting ledger  Subject to accounting
                          of an entity            policies and controls

  Clearing                Exchange, validation,   Track files, messages,
                          and/or calculation of   and exceptions
                          payment instructions    
                          and obligations         

  Settlement              Discharge of            Track settlement
                          obligations between     evidence separately
                          parties                 

  Settlement finality     Point at which a        Defined by system rules
                          settlement is           and law
                          irrevocable under the   
                          applicable rules        

  Gross settlement        Settlement of items     More granular funding
                          individually rather     needs may result
                          than netting them       
                          together                

  Net settlement          Settlement of netted    Requires correct
                          obligations             netting and reporting

  Reconciliation          Comparing records from  Must detect and explain
                          independent sources     mismatches

  Suspense account        Temporary accounting    Must be controlled,
                          location for an item    aged, and investigated
                          not yet properly        
                          classified              

  Reversal                Compensating            Link to original;
                          transaction that        prevent duplicate
                          offsets an earlier      reversals
                          effect under defined    
                          rules                   

  Refund                  Return of value after a Apply eligibility and
                          completed payment       business rules

  Adjustment              Controlled correction   Requires reason,
                          or accounting entry     approval, and audit

  Chargeback              Dispute process defined Track evidence,
                          by applicable           deadlines, and outcomes
                          payment-system rules    

  Liquidity               Ability to meet payment Monitor funding and
                          obligations when due    exposure

  Prefunding              Funding placed in       Track available funds
                          advance to support      and replenishment
                          future transactions     

  Float                   Electronic value        Distinguish from
                          available to an         physical cash
                          agent/distributor in an 
                          MFS model               

  E-money                 Electronically stored   Legal treatment depends
                          monetary value under a  on jurisdiction and
                          defined legal model     product

  Receivable              Amount owed to the      Track counterparty and
                          entity                  expected collection

  Payable                 Amount the entity owes  Track due amount and
                          another party           settlement status

  Counterparty risk       Risk that another party Monitor exposures and
                          fails to meet its       partner health
                          obligation              

  Settlement exposure     Amount at risk while    Set monitoring and
                          settlement remains      escalation rules
                          incomplete              

  Idempotency             Repeating the same      Use stable keys and
                          logical request does    durable records
                          not create an           
                          additional effect       

  Unknown outcome         The system cannot yet   Do not blindly retry as
                          establish whether an    a new transaction
                          external operation      
                          completed               

  Compensating entry      New entry that corrects Preserve original audit
                          or offsets an earlier   history
                          entry                   

  Transaction reference   Identifier used to      Define uniqueness and
                          trace a transaction     scope
                          across systems          

  Available limit         Remaining amount or     Update atomically under
                          count allowed under a   concurrency
                          limit rule              

  Settlement account      Account used for        Not the same as a
                          institutional           customer wallet
                          settlement arrangements 

  Safeguarding/trust      Arrangement for         Confirm legal structure
  arrangement             protecting or           and requirements
                          segregating customer    
                          funds where required    
  -----------------------------------------------------------------------

### Frequently confused pairs

#### Ledger balance vs available balance

Ledger balance reflects posted entries. Available balance may subtract
holds and other unavailable amounts. The exact definition must be
consistent across product, ledger, and API documentation.

#### Authorization vs posting

Authorization is permission to proceed, sometimes with a reservation.
Posting records the financial effect. Some systems combine stages, but
the concepts remain useful to distinguish.

#### Clearing vs settlement

Clearing handles instructions and obligations; settlement discharges
obligations. Exact system definitions vary.

#### Reversal vs refund

A reversal compensates for a previous financial effect under defined
rules. A refund returns value after a completed payment. The distinction
affects eligibility, reporting, and accounting.

#### Customer wallet vs settlement-bank account

The wallet represents the customer's claim under the MFS model. The
settlement-bank account is an institutional account used under a funding
or settlement arrangement. One cannot be substituted for the other in
accounting explanations.

#### Payment success vs settlement completion

A customer payment can be accepted or posted before a later settlement
cycle completes. The system must define what "success" means for each
API and transaction type.

------------------------------------------------------------------------

# 8. Accounting Preview for Software Engineers

This section is an introduction. It does not replace formal accounting
training or the institution's approved accounting policy.

## 8.1 Debit and credit

Debit and credit are the two sides of an accounting entry. They do not
universally mean decrease and increase.

In simplified traditional accounting:

-   Assets generally increase with debits and decrease with credits.
-   Expenses generally increase with debits.
-   Liabilities, equity, and revenue generally increase with credits.

The correct interpretation depends on the account type and accounting
framework.

## 8.2 Balanced journals

A double-entry journal typically satisfies:

**Total debits = Total credits**

Example of a simplified P2P subledger transfer of BDT 300:

  Account                         Debit    Credit
  --------------------------- --------- ---------
  Sender wallet liability           300         0
  Receiver wallet liability           0       300
  **Total**                     **300**   **300**

This illustrates the transfer between two customer wallet liabilities.
It is not a complete chart-of-accounts design for every MFS operator.

## 8.3 Customer fee example

Suppose the customer sends BDT 1,000 and pays a BDT 10 fee. The receiver
gets BDT 1,000.

A simplified conceptual view may be:

  Account                                       Debit      Credit
  --------------------------------------- ----------- -----------
  Sender wallet liability                       1,010           0
  Receiver wallet liability                         0       1,000
  Fee revenue or applicable fee account             0          10
  **Total**                                 **1,010**   **1,010**

The exact accounting classification and any tax or revenue-sharing lines
must follow the entity's approved policy and contractual model.

## 8.4 Agent commission example

Suppose an agent earns a BDT 2 commission on an eligible service. The
system may need to recognize a commission expense and a payable to the
agent, or use another treatment based on the accounting model. The
commission may be accrued immediately and settled later.

Do not confuse "commission calculated," "commission payable recorded,"
and "commission paid."

## 8.5 Why not delete a posted entry?

If an entry was posted incorrectly, deleting it can destroy the audit
trail and make downstream reports inconsistent. A controlled correction
typically records a compensating entry, links it to the original,
records a reason, and follows the required approval process.

------------------------------------------------------------------------

# 9. How Financial Concepts Shape Backend Engineering

## 9.1 Domain modeling

Common entities may include:

-   Customer.
-   Financial account.
-   Wallet.
-   Agent.
-   Merchant.
-   Transaction.
-   Transaction attempt.
-   Ledger journal.
-   Ledger entry.
-   Hold.
-   Fee assessment.
-   Commission payable.
-   Partner request.
-   Callback delivery.
-   Settlement batch.
-   Reconciliation item.
-   Audit event.

Do not automatically place all these concepts into one giant transaction
model. Their lifecycle and source of truth may differ.

## 9.2 API semantics

A payment API should define: - Whether the amount includes fees. - The
currency. - The identity of the payer and payee. - The idempotency-key
scope and lifetime. - What each response status means. - Whether a
response is final or provisional. - How a caller can query the
outcome. - How duplicate requests are handled. - Which reference should
be supplied during support and reconciliation.

Example request:

``` json
{
  "client_reference": "demo-order-2026-00041",
  "amount": "1000.00",
  "currency": "BDT",
  "recipient_wallet": "DEMO-WALLET-002"
}
```

Use strings for decimal amounts in JSON examples to avoid accidental
binary floating-point representation. Define precision, allowed scale,
rounding, and validation rules in the API contract.

## 9.3 Python money representation

Avoid binary floating-point types such as `float` for monetary
calculations. Use `Decimal` with explicit precision and rounding
policies, or integer minor units when appropriate for the currency and
use case.

Illustrative Python:

``` python
from decimal import Decimal, ROUND_HALF_UP

amount = Decimal("1000.00")
rate = Decimal("0.015")
fee = (amount * rate).quantize(
    Decimal("0.01"),
    rounding=ROUND_HALF_UP,
)

print(fee)  # 15.00
```

This is a calculation example, not a complete fee engine. Real fee rules
may include caps, floors, tax, tiering, minimum charges, exemptions, or
contract-specific rounding.

## 9.4 Database transactions and concurrency

A dangerous pattern is:

1.  Read a wallet balance.
2.  Check that it is sufficient.
3.  Perform a separate update later.

Two concurrent requests can both read the same balance and both pass the
check unless the operation is coordinated safely.

Possible controls include: - Database transactions. - Appropriate row
locking. - Atomic conditional updates. - Unique constraints. -
Idempotency records. - Consistent lock ordering. - Concurrency tests
against the production database engine.

The correct strategy depends on the data model and throughput
requirements. Do not assume an in-memory lock protects multiple
application workers or servers.

## 9.5 Audit records and logs

Keep separate purposes clear:

-   **Application logs:** Help diagnose technical behavior.
-   **Business events:** Describe meaningful domain actions.
-   **Ledger records:** Represent recognized financial effects.
-   **Audit records:** Support accountability and traceability.
-   **Metrics:** Measure service behavior and operational health.
-   **Reconciliation records:** Capture matching results and exceptions.

Do not log PINs, passwords, full secrets, or unnecessary personal data.
Apply masking, access controls, retention rules, and approved logging
policies.

------------------------------------------------------------------------

# 10. Failure Scenarios and Recovery

## 10.1 Timeout after a partner request

A timeout means the caller did not receive a response within the
expected time. It does not prove that the partner failed to process the
request.

Possible sequence:

1.  MFS sends a request to a partner.
2.  Partner processes the transaction.
3.  Partner sends a response.
4.  Network interruption prevents the response from reaching the MFS
    platform.
5.  MFS observes a timeout.

The transaction outcome is uncertain. A safe recovery process may use
the original reference for a status inquiry, wait for an authenticated
callback, or follow the partner's documented recovery protocol. Do not
submit a fresh, unrelated payment merely because the first request timed
out.

## 10.2 Duplicate request

A mobile app may retry because the user sees a spinner or loses
connectivity. The backend should recognize the same logical operation
using an idempotency key or another contractually defined unique
reference.

The system should return the existing result or continue processing the
same operation rather than create a second financial effect.

## 10.3 Duplicate callback

Partners may deliver a callback more than once. Verify the callback,
persist its event identity or deduplication key, and ensure that
processing is idempotent. A callback should not cause a second credit or
reversal simply because it was delivered twice.

## 10.4 Conflicting status

The internal platform may receive a pending response followed by a
success callback, or receive contradictory information from different
sources. Define allowed state transitions and source-of-truth rules.
Preserve evidence and escalate irreconcilable cases rather than
arbitrarily overwriting history.

## 10.5 Reconciliation mismatch

Examples: - Internal record exists but partner record is missing. -
Partner record exists but internal record is missing. - Amounts
differ. - Statuses disagree. - The same reference appears more than
once. - Settlement amount differs from the expected amount.

The correct response is to identify the discrepancy, retain the
evidence, investigate, and apply a controlled correction if justified.
Reconciliation is not a license to silently change money until totals
match.

------------------------------------------------------------------------

# 11. Exercises and Assessment

## 11.1 Fundamentals

1.  Explain the three functions of money.
2.  Why is a wallet balance a financial claim rather than merely a
    number?
3.  Distinguish a commercial bank deposit from central bank money.
4.  Explain why a payment message is not the same as settlement.
5.  What does the accounting equation express?

**Evaluation criteria:** Definitions are accurate, examples are
realistic, and the answer distinguishes customer-facing records from
institutional obligations.

## 11.2 Participants

1.  Explain the difference between a PSP and a PSO.
2.  Explain why a payment gateway does not necessarily settle funds.
3.  Describe the roles of issuer and acquirer in a card payment.
4.  Explain the difference between an MFS agent's cash and electronic
    float.
5.  Draw a participant map for a bank-to-wallet transfer.

**Evaluation criteria:** Correctly identify roles and avoid assuming
that every institution has identical permissions.

## 11.3 Transaction analysis

Scenario: A customer requests a BDT 800 transfer from a bank account to
an MFS wallet. The bank debits the account, but the MFS provider times
out before receiving a final response.

Answer: - What is known? - What remains unknown? - What evidence should
be preserved? - What should the platform avoid doing? - Which status
inquiry or callback mechanisms should be used? - How will reconciliation
help?

A strong answer does not equate timeout with failure and does not
blindly create a new transfer.

## 11.4 Accounting preview

1.  Create a conceptual balanced journal for a BDT 400 P2P transfer.
2.  Add a BDT 4 customer fee and show the total sender debit.
3.  Explain why a reversal should preserve the original entry.
4.  Explain why debit does not universally mean "decrease."
5.  Identify what information is needed to determine the complete
    general-ledger treatment.

## 11.5 Engineering application

1.  Propose an idempotency strategy for a transfer endpoint.
2.  Explain how two concurrent requests can overspend a wallet if checks
    are not atomic.
3.  List the records needed to trace a transaction across a partner API
    and the ledger.
4.  Explain why callbacks must be authenticated and deduplicated.
5.  Design a reconciliation exception record.

------------------------------------------------------------------------

# 12. Progressive Projects

## Project A: Financial ecosystem documentation

### Objective

Build a concise but accurate reference for the payment ecosystem
relevant to your work.

### Deliverables

-   Participant glossary.
-   Diagram of the Bangladesh ecosystem.
-   Flow diagrams for cash-in, cash-out, P2P, merchant payment, and
    bank-to-wallet transfer.
-   Glossary of at least 40 terms.
-   Research worksheet with official sources and verification dates.

### Acceptance criteria

-   Each participant has a clearly stated role.
-   Messaging, accounting, clearing, and settlement are shown
    separately.
-   Unverified regulatory claims are explicitly flagged.
-   Each flow identifies failure and reconciliation points.

## Project B: MFS transaction simulator

### Objective

Simulate basic MFS behavior without connecting to real money or
production systems.

### Suggested components

-   Synthetic customers and wallets.
-   Agent float and physical-cash tracking.
-   Merchant accounts.
-   P2P transfers.
-   Cash-in and cash-out.
-   Fee rules.
-   Transaction states.
-   Idempotency keys.
-   Event/audit records.

### Acceptance criteria

-   Amounts use `Decimal` or a well-defined integer-minor-unit
    representation.
-   Duplicate requests do not duplicate financial effects.
-   Each transaction has a stable reference.
-   Invalid account states and insufficient balances are rejected.
-   Tests cover success, failure, duplicate request, and concurrent
    requests.

## Project C: Basic ledger prototype

### Objective

Build a small educational ledger engine.

### Suggested models

-   `Account`
-   `Journal`
-   `JournalLine`
-   `TransactionReference`
-   `ReversalReference`

### Acceptance criteria

-   A journal cannot be posted unless its debits and credits balance.
-   Posted entries cannot be edited through normal application paths.
-   Reversals create new linked records.
-   Account balances can be explained from journal lines.
-   Automated tests cover invalid journals and duplicate references.

**Production warning:** These projects are educational prototypes.
Handling real customer money requires extensive additional design,
controls, independent review, security assessment, compliance approval,
operational readiness, disaster recovery, and formal accounting
validation.

------------------------------------------------------------------------

# 13. Three-Week Study Plan

This is a first-pass foundation plan. It is not a claim that every
subject can be mastered in three weeks. Reserve advanced ledger
accounting, regulatory interpretation, and production architecture for
later phases.

## Week 1 --- Money, participants, and infrastructure

  -------------------------------------------------------------------------
  Day              Topics            Practical work          Estimated time
  ---------------- ----------------- ---------------- ---------------------
  1                What money is;    Write                       60--90 min
                   cash, deposits,   definitions and  
                   wallet balances   a simple         
                                     balance-sheet    
                                     example          

  2                Assets,           Explain why                 60--90 min
                   liabilities,      wallet balances  
                   claims,           are obligations  
                   accounting        under a typical  
                   equation          model            

  3                Central bank,     Draw a                      60--90 min
                   commercial banks, relationship map 
                   NBFIs                              

  4                PSPs, PSOs, MFS   Build a                     60--90 min
                   providers,        comparison table 
                   gateways,                          
                   aggregators                        

  5                Switches, card    Draw a                      60--90 min
                   networks,         card-payment     
                   issuer/acquirer   flow             

  6                Bangladesh        Verify official            90--120 min
                   payment ecosystem source material  
                   research          and fill in the  
                                     worksheet        

  7                Revision and      Answer the Week             45--90 min
                   catch-up          1 questions      
                                     without notes    
  -------------------------------------------------------------------------

## Week 2 --- Operations and transaction workflows

  ------------------------------------------------------------------------
  Day              Topics           Practical work          Estimated time
  ---------------- ---------------- ---------------- ---------------------
  8                Onboarding, KYC, Draw an                     60--90 min
                   e-KYC, due       onboarding state 
                   diligence        flow             

  9                Account          Define sample               60--90 min
                   lifecycle and    account states   
                   restrictions     and allowed      
                                    actions          

  10               Cash-in and      Trace both                  60--90 min
                   cash-out         physical cash    
                                    and electronic   
                                    float            

  11               P2P and merchant Calculate                   60--90 min
                   payments         principal, fees, 
                                    and recipient    
                                    amount           

  12               Bill payment and Draw synchronous            60--90 min
                   bulk             and asynchronous 
                   disbursement     paths            

  13               Bank transfers   Mark uncertain             90--120 min
                   and remittances  outcomes and     
                                    status inquiry   
                                    points           

  14               Revision and     Explain three               45--90 min
                   catch-up         transaction      
                                    flows from       
                                    memory           
  ------------------------------------------------------------------------

## Week 3 --- Terminology, accounting preview, and engineering

  ------------------------------------------------------------------------
  Day              Topics           Practical work          Estimated time
  ---------------- ---------------- ---------------- ---------------------
  15               Financial        Create                      60--90 min
                   vocabulary       flashcards for   
                                    key terms        

  16               Debit, credit,   Work through                60--90 min
                   assets,          basic journal    
                   liabilities,     examples         
                   journals                          

  17               Clearing,        Create a sample             60--90 min
                   settlement,      mismatch report  
                   reconciliation                    

  18               Fees,            Write a fee                 60--90 min
                   commissions,     calculation      
                   limits           function using   
                                    `Decimal`        

  19               APIs,            Design a                    60--90 min
                   idempotency,     transfer request 
                   transaction      and response     
                   states           contract         

  20               Failure handling Write a timeout            90--120 min
                   and auditability recovery         
                                    scenario         

  21               Final assessment Complete the               60--120 min
                   and review       checklist and    
                                    identify topics  
                                    for Phase 2      
  ------------------------------------------------------------------------

### Weekly review template

``` markdown
## Weekly Review

- Week:
- Topics completed:
- Concepts I can explain without notes:
- Concepts I can explain only with help:
- Questions for finance/operations/compliance:
- Official sources reviewed:
- Practical work completed:
- Mistakes or misconceptions corrected:
- Topics to revisit:
- Next week's goal:
```

------------------------------------------------------------------------

# 14. Completion Checklist

## Conceptual foundations

-   [ ] I can explain money as a medium of exchange, unit of account,
    and store of value.
-   [ ] I can distinguish physical cash, commercial bank deposits, and
    MFS wallet balances.
-   [ ] I can explain assets, liabilities, and financial claims.
-   [ ] I understand the accounting equation.
-   [ ] I can distinguish messaging, accounting, clearing, settlement,
    and reconciliation.

## Participants

-   [ ] I can explain the role of a central bank.
-   [ ] I can explain commercial banks and NBFIs.
-   [ ] I can distinguish PSPs and PSOs.
-   [ ] I can explain MFS providers, gateways, aggregators, and
    switches.
-   [ ] I can explain issuer and acquirer roles.
-   [ ] I can distinguish an agent's physical cash from electronic
    float.
-   [ ] I can map the participants in a bank-to-wallet transaction.

## Operations

-   [ ] I can describe onboarding and KYC at a high level.
-   [ ] I can explain cash-in and cash-out.
-   [ ] I can explain P2P and merchant payment workflows.
-   [ ] I can explain bill payment and bulk disbursement workflows.
-   [ ] I can explain why bank-to-wallet transfers may have uncertain
    outcomes.
-   [ ] I can distinguish reversal, refund, adjustment, and chargeback.

## Engineering

-   [ ] I can explain why `float` is unsuitable for exact financial
    calculations.
-   [ ] I can describe an idempotency strategy.
-   [ ] I can explain a basic concurrency risk.
-   [ ] I can identify records needed to trace a transaction.
-   [ ] I can explain why audit records should preserve financial
    history.
-   [ ] I can describe the purpose of reconciliation.
-   [ ] I can identify official sources for verifying
    Bangladesh-specific requirements.

------------------------------------------------------------------------

# 15. Glossary Quick Reference

Use this section for quick revision. The detailed definitions are in
Section 7.

-   **Authorization:** Decision that an operation is permitted.
-   **Available balance:** Amount currently usable after holds and
    restrictions.
-   **Beneficiary:** Party intended to receive the benefit of a payment.
-   **Clearing:** Exchange/validation of payment instructions and
    calculation of obligations.
-   **Commission:** Compensation paid or payable to an agent or partner.
-   **Customer wallet liability:** The issuer's obligation represented
    by a customer wallet balance under the applicable model.
-   **Fee:** Charge for a service.
-   **Float:** Electronic value available to an agent or distributor.
-   **Idempotency:** Repeating the same logical request does not create
    an extra financial effect.
-   **Ledger:** Financial records used to explain account activity and
    balances.
-   **Ledger balance:** Balance based on posted entries under the
    account's rules.
-   **MDR:** Merchant discount rate or related merchant-facing charge.
-   **Net settlement:** Settlement of netted obligations.
-   **Posting:** Recording a financial effect in the ledger.
-   **Reconciliation:** Comparing independent records and investigating
    differences.
-   **Reversal:** Compensating action that offsets an earlier financial
    effect under defined rules.
-   **Settlement:** Discharge of obligations between institutions or
    parties.
-   **Settlement exposure:** Amount at risk while settlement remains
    incomplete.
-   **Suspense account:** Temporary account for items not yet properly
    classified.
-   **Transaction reference:** Identifier used to trace an operation
    across systems.
-   **Unknown outcome:** A state where the system cannot yet determine
    whether an external operation completed.

------------------------------------------------------------------------

# 16. References and Verification Sources

Use the following official sources as starting points. Before relying on
a specific requirement, locate the current document, confirm its
effective status, and record the date you checked it.

## Bangladesh-specific sources

-   [Bangladesh Bank --- Official website](https://www.bb.org.bd/)\
    Start here for current circulars, payment-system information,
    MFS-related regulations, and official publications.

-   [Bangladesh Financial Intelligence Unit
    (BFIU)](https://www.bfiu.org.bd/)\
    Use for official AML/CFT materials, guidelines, and relevant
    financial intelligence requirements. Confirm which materials apply
    to the specific institution and activity.

For BEFTN, NPSB, RTGS, automated cheque processing, Bangla QR, Binimoy,
MFS regulations, e-KYC, and transaction limits, locate the current
official documentation through Bangladesh Bank or the responsible
official operator. Do not rely on an unverified summary for production
decisions.

## Technical sources

-   [Python documentation](https://docs.python.org/3/)
-   [Python `decimal`
    documentation](https://docs.python.org/3/library/decimal.html)
-   [Django documentation](https://docs.djangoproject.com/)
-   [Django REST Framework
    documentation](https://www.django-rest-framework.org/)
-   [PostgreSQL documentation](https://www.postgresql.org/docs/)
-   [ISO 20022](https://www.iso20022.org/)
-   [PCI Security Standards
    Council](https://www.pcisecuritystandards.org/)

## Source-verification log

Record your findings in a table like this:

  --------------------------------------------------------------------------------------------
  Topic            Official          Published/effective   Date        Key finding Open
                   source/document   date                  verified                questions
  ---------------- ----------------- --------------------- ----------- ----------- -----------
  MFS regulation                                                                   

  KYC/e-KYC                                                                        

  AML/CFT                                                                          

  BEFTN                                                                            

  NPSB                                                                             

  RTGS                                                                             

  Bangla QR                                                                        

  Transaction                                                                      
  limits                                                                           

  Settlement and                                                                   
  reconciliation                                                                   
  --------------------------------------------------------------------------------------------

## Final guidance

The most important outcome of Phase 1 is not memorizing names or
acronyms. It is being able to explain, accurately and clearly:

1.  Who is involved in a financial transaction.
2.  What each participant is responsible for.
3.  What the customer requested.
4.  What financial effect was recorded.
5.  What obligations remain between institutions.
6.  How settlement is evidenced.
7.  How the system detects and resolves mismatches.
8.  Which rules are regulatory requirements and which are
    company-specific design choices.

Once these fundamentals are clear, proceed to a dedicated phase on
double-entry accounting and ledger engineering, followed by
transaction-state management, partner integrations, reconciliation, and
production-grade MFS architecture.
