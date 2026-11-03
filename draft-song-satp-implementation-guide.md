---
# RFCXML v3 source; native v3 tables, lists and inline code avoid
# deprecated XML elements in the kramdown-rfc output.
# Build submission XML and remove kramdown-generated line PIs:
# kramdown-rfc --v3 draft-song-satp-implementation-guide-03.md |
#   sed -E 's/<\?line[[:space:]][^?]*\?>//g' > draft-song-satp-implementation-guide-03.xml
stand_alone: yes
pi:
  toc: yes
  tocdepth: 4
  sortrefs: yes
  symrefs: yes

title: Secure Asset Transfer Protocol (SATP) Implementation Guide
abbrev: SATP Implementation Guide
docname: draft-song-satp-implementation-guide-latest
category: info
ipr: trust200902
area: Applications and Real-Time
workgroup: Secure Asset Transfer Protocol
date: 2 Nov 2026
v: 3
keyword:
 - Gateway
 - Digital asset network
 - Asset transfer
 - Interoperability testing
consensus: false

venue:
  group: "Secure Asset Transfer Protocol"
  type: ""
  mail: "sat@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/sat/"
  github: "anawhj/draft-song-satp-implementation-guide"
  latest: "https://anawhj.github.io/draft-song-satp-implementation-guide/draft-song-satp-implementation-guide.html"

author:
 -
    ins: H. Song
    fullname: Hyojin Song
    organization: Seoul National Univ.
    email: fun@snu.ac.kr
    country: South Korea
 -
    ins: Y-G. Hong
    fullname: Yong-Geun Hong
    organization: Daejeon Univ.
    email: yonggeun.hong@gmail.com
    country: South Korea
 -
    ins: G. Jeong
    fullname: GukSik Jeong
    organization: TTA
    email: jgsigi@tta.or.kr
    country: South Korea
 -
    ins: T. Hardjono
    fullname: Thomas Hardjono
    organization: MIT
    email: hardjono@mit.edu
    country: USA

informative:
  SATcore:
    title: Secure Asset Transfer Protocol (SATP) Core
    author:
    - ins: M. Hargreaves
    - ins: T. Hardjono
    - ins: R. Belchior
    - ins: V. Ramakrishna
    - ins: A. Chiriac
    date: 2026-09-25
    seriesinfo:
      Internet-Draft: draft-ietf-satp-core-17
    target: https://datatracker.ietf.org/doc/html/draft-ietf-satp-core-17
  SATarch:
    title: Secure Asset Transfer (SAT) Interoperability Architecture
    author:
    - ins: T. Hardjono
    - ins: M. Hargreaves
    - ins: N. Smith
    - ins: V. Ramakrishna
    date: 2026-09-15
    seriesinfo:
      Internet-Draft: draft-ietf-satp-architecture-10
    target: https://datatracker.ietf.org/doc/html/draft-ietf-satp-architecture-10
  SETUP:
    title: SATP Setup Stage
    author:
    - ins: D. Avrilionis
    - ins: T. Hardjono
    date: 2026-09-23
    seriesinfo:
      Internet-Draft: draft-avrilionis-satp-setup-stage-05
    target: https://datatracker.ietf.org/doc/html/draft-avrilionis-satp-setup-stage-05
  REGISTRY:
    title: Artefacts Registry
    author:
    - ins: D. Avrilionis
    - ins: T. Hardjono
    date: 2026-09-23
    seriesinfo:
      Internet-Draft: draft-avrilionis-satp-artefacts-registry-01
    target: https://datatracker.ietf.org/doc/html/draft-avrilionis-satp-artefacts-registry-01
  TRANSPORT-DISCUSSION:
    title: "Question about citing non-IETF specifications (action items from Interim WG meeting)"
    author:
    - ins: T. Hardjono
    date: 2026-10-19
    target: https://mailarchive.ietf.org/arch/msg/sat/qY1ulLiPlwXrdana51YBTW3irv4/
  HERMES:
    title: Hyperledger Cacti SATP Hermes source code
    author:
    - org: Hyperledger Cacti contributors
    date: 2026-10-28
    target: https://github.com/hyperledger-cacti/cacti/tree/768916ca4b0819bb8febaeb1806339f9f1b38124/packages/cactus-plugin-satp-hermes
  CONNECT:
    title: Connect for Go - gRPC compatibility
    author:
    - org: The Connect Authors
    target: https://connectrpc.com/docs/go/grpc-compatibility/
  PROTO-SERIALIZATION:
    title: Proto Serialization Is Not Canonical
    author:
    - org: Protocol Buffers Authors
    target: https://protobuf.dev/programming-guides/serialization-not-canonical/
  GRPC-RETRY:
    title: gRPC Retry
    author:
    - org: The gRPC Authors
    target: https://grpc.io/docs/guides/retry/
  RFC7515: RFC7515
  RFC7516: RFC7516
  RFC7518: RFC7518
  RFC8259: RFC8259
  RFC8446: RFC8446
  RFC8785: RFC8785
  RFC9457: RFC9457

--- abstract

This document provides non-normative implementation guidance for the Secure
Asset Transfer Protocol (SATP). It describes a small, independent peer
implementation and reproducible tests for evaluating protocol behavior and
interoperability. It relates Core processing to Setup and Artefacts Registry
proposals, while distinguishing published requirements from experimental
implementation choices. The guide
addresses transport mappings, message protection, asset-state observation,
failure handling, and reporting of specification ambiguities. It does not
define a new protocol, certify an implementation, or claim completed
interoperability experiments.

--- middle

# Introduction {#introduction-doc}

SATP coordinates asset transfers between gateways representing different
asset networks. {{SATarch}} supplies the architectural context and
{{SATcore}} defines the Core exchange. Independent implementations help
test whether the specifications provide enough detail for different
developers to reach compatible behavior.

This guide supports that work with implementation boundaries, observable
state transitions, and reproducible test scenarios. Its initial target is
a small peer that can exercise both sides of a bilateral transfer against
another implementation, including Hyperledger Cacti SATP Hermes. Such a
peer can expose ambiguous requirements without providing the operational
capabilities expected of a production gateway.

This revision focuses on implementing the preparation described in
{{SETUP}} and integrating the artefact services in {{REGISTRY}}. These
documents have separate scopes and revision histories. Their inclusion
here does not make a particular Setup or Registry implementation
mandatory for Core. An experiment should identify the specifications
and subsets it exercises.

{{implementation-scope}} illustrates the logical scope covered by this
guide. The two application-gateway-network stacks support a bilateral
Core exchange, with preparation and supporting services shown separately.

<figure anchor="implementation-scope" markdown="0">
<name>Conceptual scope of SATP implementation guidance</name>
<artset>
<artwork type="svg" name="satp-implementation-scope.svg">
<svg xmlns="http://www.w3.org/2000/svg" version="1.2" baseProfile="tiny" viewBox="0 0 1020 630" width="1020" height="630">
<title>Conceptual scope of SATP implementation guidance</title>
<desc>Two application, gateway and network stacks coordinate a transfer context and exchange Core asset-transfer messages. An Artefacts Registry supports Setup through API3 lookups.</desc>
<text x="150" y="22" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black" font-weight="bold">Origin side</text>
<text x="870" y="22" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black" font-weight="bold">Destination side</text>
<rect x="40" y="38" width="220" height="65" fill="white" stroke="black" stroke-width="2"/>
<rect x="760" y="38" width="220" height="65" fill="white" stroke="black" stroke-width="2"/>
<text x="150" y="64" font-family="sans-serif" font-size="20" text-anchor="middle" fill="black" font-weight="bold">Application App1</text>
<text x="150" y="89" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">Intent / preparation</text>
<text x="870" y="64" font-family="sans-serif" font-size="20" text-anchor="middle" fill="black" font-weight="bold">Application App2</text>
<text x="870" y="89" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">Intent / preparation</text>
<path d="M 260 76 L 760 76" fill="none" stroke="black" stroke-width="2" stroke-dasharray="8 6"/>
<polygon points="260,76 270,71 270,81" fill="black"/>
<polygon points="760,76 750,71 750,81" fill="black"/>
<text x="510" y="61" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black">Transfer-context coordination</text>
<rect x="370" y="132" width="280" height="68" fill="white" stroke="black" stroke-width="2"/>
<text x="510" y="159" font-family="sans-serif" font-size="21" text-anchor="middle" fill="black" font-weight="bold">Artefacts Registry</text>
<text x="510" y="186" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">TARs / profiles / identifiers</text>
<path d="M 150 103 L 150 265" fill="none" stroke="black" stroke-width="2"/>
<polygon points="150,103 145,113 155,113" fill="black"/>
<polygon points="150,265 145,255 155,255" fill="black"/>
<path d="M 870 103 L 870 265" fill="none" stroke="black" stroke-width="2"/>
<polygon points="870,103 865,113 875,113" fill="black"/>
<polygon points="870,265 865,255 875,255" fill="black"/>
<text x="133" y="174" font-family="sans-serif" font-size="17" text-anchor="end" fill="black">API1</text>
<text x="133" y="197" font-family="sans-serif" font-size="16" text-anchor="end" fill="black">Initiate /</text>
<text x="133" y="219" font-family="sans-serif" font-size="16" text-anchor="end" fill="black">status</text>
<text x="887" y="174" font-family="sans-serif" font-size="17" text-anchor="start" fill="black">API1</text>
<text x="887" y="197" font-family="sans-serif" font-size="16" text-anchor="start" fill="black">Initiate /</text>
<text x="887" y="219" font-family="sans-serif" font-size="16" text-anchor="start" fill="black">status</text>
<path d="M 215 265 L 215 224 L 440 224 L 440 200" fill="none" stroke="black" stroke-width="2" stroke-dasharray="8 6"/>
<polygon points="215,265 210,255 220,255" fill="black"/>
<polygon points="440,200 435,210 445,210" fill="black"/>
<path d="M 805 265 L 805 224 L 580 224 L 580 200" fill="none" stroke="black" stroke-width="2" stroke-dasharray="8 6"/>
<polygon points="805,265 800,255 810,255" fill="black"/>
<polygon points="580,200 575,210 585,210" fill="black"/>
<text x="320" y="249" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">API3 lookup</text>
<text x="700" y="249" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">API3 lookup</text>
<rect x="40" y="265" width="220" height="130" fill="white" stroke="black" stroke-width="2"/>
<rect x="760" y="265" width="220" height="130" fill="white" stroke="black" stroke-width="2"/>
<text x="150" y="302" font-family="sans-serif" font-size="22" text-anchor="middle" fill="black" font-weight="bold">Gateway G1</text>
<text x="150" y="336" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black">Core peer</text>
<text x="870" y="302" font-family="sans-serif" font-size="22" text-anchor="middle" fill="black" font-weight="bold">Gateway G2</text>
<text x="870" y="336" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black">Core peer</text>
<path d="M 260 292 L 760 292" fill="none" stroke="black" stroke-width="2" stroke-dasharray="8 6"/>
<polygon points="260,292 270,287 270,297" fill="black"/>
<polygon points="760,292 750,287 750,297" fill="black"/>
<text x="510" y="278" font-family="sans-serif" font-size="18" text-anchor="middle" fill="black">Context / gateway binding (before Core)</text>
<path d="M 260 353 L 760 353" fill="none" stroke="black" stroke-width="2"/>
<polygon points="260,353 270,348 270,358" fill="black"/>
<polygon points="760,353 750,348 750,358" fill="black"/>
<text x="510" y="338" font-family="sans-serif" font-size="19" text-anchor="middle" fill="black">Asset-transfer exchange (Stages 1-3)</text>
<path d="M 150 395 L 150 487" fill="none" stroke="black" stroke-width="2"/>
<polygon points="150,395 145,405 155,405" fill="black"/>
<polygon points="150,487 145,477 155,477" fill="black"/>
<path d="M 870 395 L 870 487" fill="none" stroke="black" stroke-width="2"/>
<polygon points="870,395 865,405 875,405" fill="black"/>
<polygon points="870,487 865,477 875,477" fill="black"/>
<text x="168" y="443" font-family="sans-serif" font-size="17" text-anchor="start" fill="black">Asset operations</text>
<text x="852" y="443" font-family="sans-serif" font-size="17" text-anchor="end" fill="black">Asset operations</text>
<rect x="40" y="487" width="220" height="88" fill="white" stroke="black" stroke-width="2"/>
<rect x="760" y="487" width="220" height="88" fill="white" stroke="black" stroke-width="2"/>
<text x="150" y="520" font-family="sans-serif" font-size="18" text-anchor="middle" fill="black" font-weight="bold">Asset network NW1</text>
<text x="150" y="550" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">Assets / ledger state</text>
<text x="870" y="520" font-family="sans-serif" font-size="18" text-anchor="middle" fill="black" font-weight="bold">Asset network NW2</text>
<text x="870" y="550" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">Assets / ledger state</text>
<path d="M 280 516 L 350 516" fill="none" stroke="black" stroke-width="2"/>
<text x="366" y="523" font-family="sans-serif" font-size="17" text-anchor="start" fill="black">Core or local network interaction</text>
<path d="M 280 551 L 350 551" fill="none" stroke="black" stroke-width="2" stroke-dasharray="8 6"/>
<text x="366" y="558" font-family="sans-serif" font-size="17" text-anchor="start" fill="black">Preparation or supporting lookup</text>
<text x="510" y="606" font-family="sans-serif" font-size="17" text-anchor="middle" fill="black">Logical relationships; not a required execution sequence.</text>
</svg>
</artwork>
<artwork type="ascii-art" name="satp-implementation-scope.txt">
     Origin side                               Destination side
+---------------------+                     +---------------------+
|   Application App1  | &lt;-- TC intent ---&gt;  |   Application App2  |
|   intent / prepare  |                     |   intent / prepare  |
+---------------------+                     +---------------------+
           ^                                           ^
           |   API1                                    |   API1
           v                                           v
+---------------------+                     +---------------------+
|      Gateway G1     | &lt;-- TC binding ---&gt; |      Gateway G2     |
|                     |    (before Core)    |                     |
|      Core peer      |&lt;== Core exchange =&gt; |      Core peer      |
|                     |    (Stages 1-3)     |                     |
+---------------------+                     +---------------------+
       ^   ^                                           ^   ^
       |   :                                           :   |
       |   :   API3: artefact lookup                   :   |
       |   +................+         +................+   |
       |                    v         v                    |
       |           +---------------------------+           |
       |           |     Artefacts Registry    |           |
       |           | TARs/profiles/identifiers |           |
       |           +---------------------------+           |
       |                                                   |
       |                 asset operations                  |
       v                                                   v
+---------------------+                     +---------------------+
|  Asset network NW1  |                     |  Asset network NW2  |
|    assets / state   |                     |    assets / state   |
+---------------------+                     +---------------------+

TC = transfer context. API1: initiation and status.
API3: artefact lookup supporting Setup.
</artwork>
</artset>
</figure>

Preparation spans the applications, their networks, and the gateway pair:
{{SETUP}} explains transfer-context establishment, asset preparation,
and gateway binding before Core starts. Side lookups through API3 obtain
artefacts described by {{REGISTRY}}; the registry does not relay the
gateway-to-gateway Core exchange. One registry is shown for readability,
but deployments may use different registries for different artefacts.

The links express responsibilities and relationships, not a complete
message sequence. Dashed links in the graphical rendering denote
preparation or supporting lookups. These interactions support Core
entry; they are not additional Core stages.
Application-to-network preparation is summarized by the application
labels rather than drawn as additional links. The experimental test
harness can drive and observe these interfaces as described later.

## Scope and Limitations {#scope}

The initial implementation scope is a direct transfer between two
gateways, one test asset per transfer, and controlled network
adapters. The test peer should implement both sender and receiver roles.
It should support invalid-input and failure injection as well as
successful transfers.

Production operations, high availability, general gateway discovery,
complete regulatory workflows, arbitrary ledger integrations, and
performance qualification are outside the initial scope. Mock networks
can demonstrate decisions made by the peer, but cannot establish a real
ledger's finality or cross-system economic guarantees.

Chaining bilateral transfers does not by itself establish atomicity over
an entire multi-hop route. An implementation studying that case should
identify its additional coordination protocol and failure assumptions.
This guide does not define that protocol. Similarly, a successful
two-phase exchange under stated assumptions does not prove unconditional
termination or automatic recovery after every possible failure.

## Specification Baseline and Pending Changes {#baseline}

The following revisions form the documentary baseline for this draft.
The section numbers cited in this guide refer to these revisions.

<table markdown="0">
<thead>
<tr>
<th align="left">Subject</th>
<th align="left">Baseline</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Core</td>
<td align="left">draft-ietf-satp-core-17</td>
</tr>
<tr>
<td align="left">Architecture</td>
<td align="left">draft-ietf-satp-architecture-10</td>
</tr>
<tr>
<td align="left">Setup</td>
<td align="left">draft-avrilionis-satp-setup-stage-05</td>
</tr>
<tr>
<td align="left">Registry</td>
<td align="left">draft-avrilionis-satp-artefacts-registry-01</td>
</tr>
</tbody>
</table>

Core revision -17 specifies an HTTPS and JSON baseline. The October 2026
discussion in {{TRANSPORT-DISCUSSION}} proposes separating transport
properties from gRPC and HTTP mappings. That discussion records a change
plan, not an already published replacement for the requirements of
revision -17. Guidance about transport separation in this document
anticipates that direction without defining a standard gRPC binding.

When a later revision is selected, implementers should update the
requirements-to-test mapping and record changes in accepted messages,
security processing, and failure behavior. An editor's copy should be
identified by a commit, not merely by a moving "latest" URL.

# Conventions and Terminology {#conventions}

This is an Informational, non-normative guide. Lowercase words such as
"should" express implementation advice. Requirements belong to the
referenced specifications; a summary or example here does not modify
them. A conflict should be resolved through the specification's authors
and working group rather than by silently treating this guide as the
authoritative definition.

The terms gateway, origin network, destination network, and commitment
agreement follow {{SATarch}}. In an asset transfer, G1 is the sending
gateway and G2 is the receiving gateway. A single implementation can
perform either role in different transfers.

The following terms describe the test environment:

<ul markdown="0">
<li><t>Independent peer: an implementation whose protocol decisions are
developed from the specification independently of the counterpart's
state-machine code. A different programming language alone does not
establish this independence.</t></li>
<li><t>Test harness: the driver that supplies inputs, injects failures,
observes peer and backend effects, and evaluates expected outcomes.</t></li>
<li><t>Test profile: a versioned, locally named configuration of specification
revisions, supported features, transport, and security choices. This
guide does not register profile names or a discovery format.</t></li>
<li><t>Ledger adapter: the boundary between a gateway's protocol processing
and the represented network's asset operations and evidence.</t></li>
<li><t>Journal: the local record of accepted inputs, protocol decisions,
backend operation identifiers, and observed outcomes.</t></li>
<li><t>Inconclusive outcome: a result for which available evidence cannot
establish the expected decision or backend effect.</t></li>
</ul>

The Transfer Context (TC), TCID, Tokenized Asset or Artefact Record
(TAR), and Digitized Asset Record (DAR) are used in the context of
{{SETUP}} and {{REGISTRY}}. Their distinct roles are discussed in
{{setup-context}}.

# Implementation Architecture {#implementation-architecture}

## Separation of Responsibilities {#implementation-components}

A small implementation can use the following components without adopting
a large gateway framework.

<table markdown="0">
<thead>
<tr>
<th align="left">Component</th>
<th align="left">Responsibility</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Core engine</td>
<td align="left">Validate permitted transitions and decide protocol actions.</td>
</tr>
<tr>
<td align="left">Codec and protection</td>
<td align="left">Decode messages, verify signatures and hashes, and encode outputs.</td>
</tr>
<tr>
<td align="left">Transport adapter</td>
<td align="left">Map delivery, responses, metadata, and errors to a selected transport.</td>
</tr>
<tr>
<td align="left">Ledger adapter</td>
<td align="left">Execute and observe network-specific asset operations.</td>
</tr>
<tr>
<td align="left">Context and journal store</td>
<td align="left">Preserve transfer associations, decisions, and operation evidence.</td>
</tr>
<tr>
<td align="left">Preparation modules</td>
<td align="left">Provide Setup orchestration and Registry access.</td>
</tr>
<tr>
<td align="left">Test harness</td>
<td align="left">Drive independent peers and evaluate protocol and asset effects.</td>
</tr>
</tbody>
</table>

The Core engine should not depend on a particular HTTP handler or RPC
framework. A useful local API accepts validated input and current state,
then produces a proposed next state and explicit actions. The surrounding
code records decisions, executes backend actions, and records their
outcomes. This design makes the same decision logic testable without a
network connection.

Separating actions does not make a local database and a remote ledger
atomic. The adapter should retain backend operation identifiers and allow
the harness to query whether an operation happened when an outcome is
uncertain. Tests should include a crash after backend execution but
before the journal records success.

## Network Adapter and Journal {#ledger-journal}

For the initial experiment, the adapter should expose asset inspection,
source locking, source extinguishing, destination creation under gateway
control, and final assignment to the beneficiary. The selected network
determines the actual operations and proofs. Any cleanup operation should
state its preconditions and irreversible boundaries.

The journal should associate the TCID, Session ID, gateway roles, asset
identifiers, message type, relevant sequence and hash information,
signature verification result, backend operation ID, and before/after
observations. Persisting a decision before sending its corresponding
message reduces uncertainty, but is not a complete recovery protocol.

The harness should independently inspect backend state rather than rely
only on the gateway's success response. In the controlled asset model it
should check that the same transferable value does not become spendable
by both the originator and beneficiary. Temporary destination assets
held by G2 should be distinguished from assets available to the final
beneficiary.

# Core Protocol Processing {#core-processing}

The implementation should derive wire messages from the selected Core
revision. The simplified TransferRequest and TransferCommit names used
in revision -02 of this guide are not a substitute for the messages
below. Full message schemas and required fields remain defined by
{{SATcore}}.

## Successful Bilateral Transfer {#core-success}

The following sequence summarizes Sections 8 through 10 of {{SATcore}}.
Message names are suffixes of the namespace
<tt>urn:ietf:params:satp:core:msgtype:</tt>. The state observations are test
boundaries rather than new wire states.

<ol markdown="0">
<li><t>Stage 1 exchanges <tt>transfer-proposal-msg</tt>, <tt>proposal-receipt-msg</tt>,
<tt>transfer-commence-msg</tt>, and <tt>ack-commence-msg</tt>, alternating between
G1 and G2. The peers establish the proposed transfer and its
commencement.</t></li>
<li><t>Stage 2 sends <tt>lock-assert-msg</tt> from G1 after the source lock. G2
validates the claim, records it in NW2 as depicted in Core Figure 2,
and returns <tt>assertion-receipt-msg</tt>.</t></li>
<li><t>Stage 3 sends <tt>commit-prepare-msg</tt> from G1. G2 creates the destination
asset under its own control and sends <tt>commit-ready-msg</tt>.</t></li>
<li><t>G1 extinguishes the source asset and sends <tt>commit-final-msg</tt> with
the relevant evidence. G2 verifies it, assigns the destination asset
to the beneficiary, and returns <tt>ack-commit-final-msg</tt>.</t></li>
<li><t>G1 records completion and sends <tt>commit-transfer-complete-msg</tt> to G2.</t></li>
</ol>

The implementation should compare this sequence against the normative
field descriptions, not copy draft examples without checking them.
In particular, creation under G2's control and assignment to the
beneficiary are distinct observations in this baseline.

## Validation and Cryptographic Processing {#core-validation}

For each message, validation should use that message's own schema and
the current session state. Useful checks include the authenticated
counterpart, expected message type and sequence where defined, transfer
association, applicable expiration, signature, claim format, and relevant
hashes. A message that decodes successfully can still be invalid in the
current state.

The signed-message and cryptographic baseline is described in Section
5.2 of {{SATcore}}: peer messages are protected with JWS {{RFC7515}},
with ECDSA P-256 and SHA-256 (ES256 {{RFC7518}}) as the baseline.
The standalone burn assertion in Section 10.3 is also signed
independently; verifying only the enclosing message does not replace
verification of that claim. The implementation
should distinguish transport keys, gateway message-signing keys, and
network evidence keys. A key received in a message is not automatically
a trusted identity.

Hash validation should be field-specific. Proposal Receipt, for example,
has its own claim-hash and timestamp semantics; the guide does not add a
previous-message hash to every message. The test profile should specify
the exact bytes used for each hash and signature, including any
envelope exclusion, serialization, and encoding rule that needs
clarification. A semantic JSON comparison is insufficient to establish
byte-level signature interoperability.

## Errors, Abort, and Uncertain Outcomes {#core-failures}

Core error handling should follow Sections 10.6, 10.7, and 11 of
{{SATcore}}. Protocol failure, an application-requested abort, and a
connection failure are distinct events. For the -17 HTTP baseline,
Section 11.3 describes error representations using
<tt>application/problem+json</tt> and {{RFC9457}}, with SATP-specific information.
These should not be replaced by invented private Core error codes.

An abort ends the session exchange, but its effect on assets depends on
the point reached. The implementation should evaluate the irreversible
boundary described in Section 11.5 before attempting cleanup. A timeout
after extinguishing the source asset does not justify recreating it or
assuming that the destination assignment failed.

Section 10.8 of Core revision -17 does not provide session recovery or
resumption. The proposed peer can preserve evidence and expose an
inconclusive outcome for operator diagnosis. Any additional recovery
exchange should be separately identified and tested as an extension.
A Hermes recovery facility is not automatically a Core -17 requirement.

The initial harness should disable configured application-level retries
for state-changing operations, document lower-level retry behavior, and
then test explicitly configured retransmission scenarios. Duplicate
inputs should be checked before repeating backend effects. This advice
does not create a retransmission protocol absent from the baseline.

## Clarifications to Track {#core-clarifications}

Revision -17 contains points that independent implementations should
raise for clarification. These include the <tt>ack-prepare-msg</tt> entry in
Section 5.3.2 versus <tt>commit-ready-msg</tt> in Section 10.2, and
<tt>mintAssertionFormat</tt> in a field description versus
<tt>mintAssertionClaimFormat</tt> in an example. Example syntax and spelling
also need validation before becoming fixtures.

The exact serialization and hash boundary, and the interaction between
general lock-release wording in Section 11.2 and the post-commit boundary
in Section 11.5, deserve explicit review. A test profile can record a
provisional interpretation to reproduce an experiment, but this guide
does not settle these specification questions or declare a new wire
format. Cases affected by unresolved choices should identify that fact
in their result.

# Transport Mappings and Message Representation {#transport-mappings}

## Published HTTP Baseline {#http-baseline}

For Core revision -17, the HTTP adapter should exercise the specified
HTTPS POST, TLS 1.3 {{RFC8446}}, JSON {{RFC8259}}, and
<tt>application/satp+json</tt> behavior, while
handling the error representation described in {{core-failures}}.
It should retain the distinction between transport delivery, protocol
acceptance, and completion of a backend operation.

The profile should document routing, status handling, response
correlation, connection timeouts, size limits, credential placement,
and any use of mutual TLS. It should identify which choices are explicit
in Core and which need agreement for the experiment. This guide does
not assign a new standard HTTP endpoint or negotiation mechanism.

## Experimental gRPC and Hermes Compatibility {#grpc-experiment}

A gRPC experiment should keep the Core domain model separate from its
service definitions and serialization. The profile should name the
service methods, unary or streaming behavior, metadata placement,
deadline and cancellation handling, and status/error mapping. A gRPC
status should not erase a signed SATP protocol error. Cancellation of a
call also does not roll back a backend operation already performed.

Native gRPC, gRPC-Web, and Connect are different protocol mappings.
Native gRPC normally uses HTTP/2; an HTTP/1.1 gRPC-Web endpoint is not
interchangeable with it. The inspected {{HERMES}} snapshot constructs
gRPC-Web transports for the Core stages in <tt>gateway-orchestrator.ts</tt>.
The adapter should match the actual counterpart's endpoints and protocol.
That observation is not a claim that Hermes defines the standard binding.

The harness should distinguish a Hermes compatibility test from a Core
-17 HTTP conformance test. It can share the same domain engine, while
the compatibility adapter records the counterpart's schema, paths,
metadata, and serialization. Using a supplied <tt>.proto</tt> file is permitted
for interoperability, but adopting its state-machine implementation
would weaken the independence of the test peer.

The inspected Hermes <tt>jws-interceptors.ts</tt> signs Protobuf binary content
and carries the JWS in the <tt>satp-signature</tt> header. Cross-language
fixtures should therefore establish the exact signed bytes before a
full transfer is attempted. {{PROTO-SERIALIZATION}} explains why
deterministic Protobuf output is not a canonical representation across
all schemas, runtimes, and changes. Re-encoding received data can change
the bytes used in signature verification.

The experiment should cover default and absent values, unknown fields,
maps, byte fields, and JSON-to-Protobuf conversion. A decision to use a
particular canonical representation is a mapping choice that needs
agreement; this guide does not silently replace the counterpart's
representation with JCS. JCS requirements for Registry are discussed
separately in {{registry-security}}.

gRPC includes retry mechanisms {{GRPC-RETRY}}; selecting it does not
provide exactly-once application effects. Retransmission and duplicate
suppression remain explicit subjects for the harness.

## Other Encodings {#other-encodings}

CBOR/COSE and other representation experiments should have separate
mapping identifiers, schemas, and test vectors. They are not presented
here as interchangeable alternatives to the -17 JSON/JWS baseline.
This revision does not define CBOR integer-key assignments or imply
that such assignments already exist in Core.

# Setup Stage Implementation {#setup-implementation}

{{SETUP}} describes preparation before the Core exchange. An implementation
should treat this as a separate module whose completion authorizes entry
into Core. Supporting the Core protocol does not by itself establish
interoperability with a particular Setup implementation. The functions,
local records, and additional checks below are implementation suggestions,
not additional standardized messages or endpoint definitions.

## Transfer Context and Asset Binding {#setup-context}

A useful local model separates four identifiers:

<ul markdown="0">
<li><t>TCID identifies the transfer context linking the transfer intent,
participants, assets, and gateways.</t></li>
<li><t>Session ID identifies a particular Core execution associated with that
context.</t></li>
<li><t>TAR ID references information held by an Artefacts Registry.</t></li>
<li><t>DAR ID identifies an asset record managed by the represented network.</t></li>
</ul>

These identifiers should have separate fields and validation rules. A TAR
reference does not replace the source network's DAR ownership and transfer
authorization checks. Similarly, assigning a DAR to a TC during Setup is
not itself the lock asserted in Core Stage 2. The ledger adapter should
expose these operations separately so tests can observe the distinction.

Section 2 of {{SETUP}} describes creating a context, propagating its ID to
the beneficiary application, preparing the destination network, binding
the gateways, and starting Core. Section 4 illustrates multiple sessions
associated with one context. This example does not define a session
recovery protocol. An implementation should preserve the association for
diagnosis without treating a new Session ID as permission to repeat an
irreversible asset operation. Core recovery limitations remain those of
the selected revision of {{SATcore}}.

## A Minimal Setup Interface {#setup-interface}

The following names describe a possible experimental interface. They do
not assign RPC methods, HTTP routes, message schemas, or error codes.
Some operations belong to applications or network adapters rather than
to a gateway service; a test driver can coordinate them without changing
the responsibilities described in {{SETUP}}.

<table markdown="0">
<thead>
<tr>
<th align="left">Suggested operation</th>
<th align="left">Local responsibility</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">CreateTC</td>
<td align="left">Record transfer intent and return a TCID.</td>
</tr>
<tr>
<td align="left">BindAssets</td>
<td align="left">Associate authorized source DARs with that TCID.</td>
</tr>
<tr>
<td align="left">PrepareReceive</td>
<td align="left">Record the beneficiary and destination preparation.</td>
</tr>
<tr>
<td align="left">BindGateways</td>
<td align="left">Authenticate and record the gateway pair for the TC.</td>
</tr>
<tr>
<td align="left">BeginTransfer</td>
<td align="left">Check all prerequisites and associate a Core session.</td>
</tr>
<tr>
<td align="left">QueryStatus</td>
<td align="left">Return observed Setup and Core progress with evidence.</td>
</tr>
</tbody>
</table>

A concrete experimental profile should specify who can invoke each
operation, what constitutes successful completion, how duplicates are
identified, and which actor can cancel an incomplete Setup. It should
also specify how concurrent updates are serialized. For example, a
transactional compare-and-set on the local context record can prevent two
successful BeginTransfer calls from allocating competing active sessions.
This local mechanism is not a new Core concurrency requirement.

## Checking Entry into Core {#setup-entry}

Section 3 of {{SETUP}} gives five prerequisites: an unambiguous context
held by the originator application; source-network awareness of the
assets and TC; a gateway pair bound to that TC; beneficiary awareness of
the incoming assets; and destination-network preparation for that
beneficiary and TC.

The proposed implementation should turn each prerequisite into a separate
check with an identifiable source of evidence. BeginTransfer should
evaluate a consistent snapshot of the evidence, authenticated participant
identities, asset set, and context expiry. It should record which check
prevented entry, without exposing sensitive beneficiary information.
An application acknowledgment alone should not be mistaken for a
network's completed preparation.

The following is a synthetic local diagnostic record. Its field names and
values are not a standardized Transfer Context or a SATP wire payload.

    {
      "contextHandle": "example-tc-42",
      "coreSessionHandle": null,
      "sourceDarHandle": "example-source-asset-42",
      "tarVersion": 3,
      "entryChecks": {
        "originatorContext": true,
        "sourcePrepared": true,
        "gatewaysBound": true,
        "beneficiaryAware": true,
        "destinationPrepared": false
      },
      "eligibleToStart": false
    }
{: .language-json}

## Consistency, Expiry, and Setup Tests {#setup-tests}

As an experimental protection against time-of-check/time-of-use changes,
the local context should retain the registry identity, TAR ID, selected
version, content digest, and applicable policy version. Before entering
Core, the implementation can check that both gateways accepted the same
snapshot. A changed snapshot should trigger explicit reconsideration of
Setup rather than silently changing the meaning of an active transfer.
This proposal does not define a new on-wire negotiation field.

An expiry policy should distinguish abandoned preparation from an active
Core transfer. An elapsed timer or lost response does not demonstrate
that a remote asset operation failed. In particular, a Setup expiry
should not automatically unlock or recreate assets after commitment has
begun. The implementation should retain the relevant evidence and use
only procedures justified by the Core revision and ledger semantics.

Tests should omit each entry prerequisite separately, substitute a
gateway or beneficiary, submit concurrent contexts for the same DAR,
repeat BeginTransfer, change a TAR after binding, and restart a gateway
between preparation and entry. The expected outcomes should include both
the application-visible decision and the observed network state. These
tests are proposed coverage; no successful execution is claimed here.

# Artefacts Registry Integration {#registry-integration}

The API3 registry in {{REGISTRY}} provides information used during
preparation. It should be implemented behind a distinct adapter so that
registry access, Setup orchestration, and Core state transitions can be
tested independently. A registry's availability and trust assumptions
should be explicit in the selected experimental profile.

## Transport, Authentication, and Trust {#registry-security}

{{REGISTRY}} specifies JSON API calls over HTTPS, supports GET and POST,
describes JWS signatures {{RFC7515}}, and identifies ES256 and JSON
Canonicalization Scheme (JCS) {{RFC8785}} support. It also describes
JWT/OAuth-based client authentication. These are properties of the
Registry draft: a possible gRPC mapping for Core does not replace API3's
specified HTTP mapping.

A test deployment can select TLS 1.3 and a small, preconfigured set of
registry identities and client credentials. This choice simplifies
reproducibility without redefining the Registry draft's transport
baseline. Certificate validation, accepted signing keys, token issuer
and audience, operation authorization, and trust in the returned artefact
should be evaluated separately. A valid registry certificate authenticates
an endpoint; it does not independently prove asset ownership or the
truth of a registered statement.

For reproducible signature checks, the test profile should record the
protected headers, exact signed content, canonicalization boundary,
signature representation, and key selection rules. Sorting JSON object
keys with a language's default serializer is not a substitute for JCS.
Tests should verify signatures across implementations using fixed
inputs and public test keys, including Unicode, numeric, and escaping
cases. Request authentication and response authenticity should both have
documented checks; an omitted response-signing detail should be recorded
as an unresolved mapping question rather than silently invented.

## Records, Versions, and Caches {#registry-versioning}

The Registry draft describes append-only history: update registers a new
version, and the operation named deletion archives a TAR. A test registry
should retain earlier versions and record archive events rather than
physically erasing the history used to explain a transfer.

A minimal deployment can begin with a pre-populated, read-only registry
and then add create, version append, and archive operations. This sequence
allows Setup tests to proceed before a complete registry implementation
exists. The report should identify the supported subset. For asynchronous
creation, the adapter should distinguish a temporary receipt from the
resolved TAR identifier and should avoid starting a dependent transfer
on an unresolved receipt.

The suggested snapshot protection in {{setup-tests}} should also govern
caching. A useful local cache key contains registry identity, TAR ID, and
version; stored digests and provenance allow a caller to verify that the
selected record has not changed. Fetching "latest" repeatedly during an
active transfer risks mixing decisions based on different records.
Archive, revocation, and policy-change handling should be explicit; a
pinned historic snapshot is an audit reference, not a reason to ignore a
new authorization decision.

The Setup diagram's registration of a TC does not define a complete TC
storage API or make a TC equivalent to a TAR. Implementations should
identify where contexts are persisted and how they reference TARs. If an
experimental deployment stores both in one database, their data models,
access rules, and version histories should remain distinguishable.

## Errors and Registry Tests {#registry-tests}

Several Registry error definitions remain unfinished in revision -01.
An adapter can use local outcomes such as not-found, unauthorized,
unsupported-version, invalid-signature, archived, conflict, and
temporarily-unavailable, provided the test profile identifies them as
local classifications rather than registered protocol error codes.
Retries should follow the operation's defined duplicate-handling rules;
loss of a create response does not establish that creation did not occur.

Registry tests should cover rejected credentials, unauthorized version
append, incorrect registry identity, modified content, mismatched
digest, stale cache entries, changes between Setup and Core entry,
repeated create/update, receipt resolution, and historical lookup after
archive. Tests should observe persisted versions as well as returned
responses. Draft examples should undergo syntax and semantic review
before becoming expected-output fixtures: a displayed update response
is not evidence of a passing version-transition test.

# Interoperability Profiles and Test Method {#test-method}

## Three Separate Assessments {#assessments}

An experiment should distinguish the following assessments. Passing one
does not establish the others.

<table markdown="0">
<thead>
<tr>
<th align="left">Assessment</th>
<th align="left">Question</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Specification conformance</td>
<td align="left">Does the observed behavior satisfy an identified requirement in the selected revision?</td>
</tr>
<tr>
<td align="left">Counterpart interoperability</td>
<td align="left">Can independently developed peers complete the agreed exchange and reject invalid input appropriately?</td>
</tr>
<tr>
<td align="left">Extension experiment</td>
<td align="left">Does a separately documented experimental mapping behave as intended?</td>
</tr>
</tbody>
</table>

The profile should identify its scope without claiming an IETF
certification. A Hermes compatibility profile can be useful even if its
mapping differs from the -17 HTTP baseline, provided that distinction
is reported. Conversely, an unsupported extension is not evidence that
the counterpart fails Core conformance.

Profile names and test configurations are local artifacts, not
standardized capability advertisements. Select profiles out of band
for the initial experiment. An automatic negotiation
mechanism would need its own specification and downgrade analysis.

## Test Case Structure and Observations {#case-structure}

A reusable case should contain a stable identifier, assessment type,
source requirement or experimental rule, preconditions, stimulus,
expected peer decision, expected backend effects, observation points,
and permitted timing assumptions. It should also identify the peer
roles and the fixture version. An expected result should come from the
specification or explicitly stated experiment, not solely from a trace
captured from the counterpart.

For a complete transfer, observe both gateways and both networks. For a
rejection, observe that the forbidden operation did not occur. For an
uncertain outcome, preserve enough evidence to distinguish a lost
response from a backend operation that was never executed. Time-bounded
observations should record their bound rather than imply a universal
liveness guarantee.

Record PASS or FAIL only when the expected result and observations are
sufficient. Use UNSUPPORTED for an absent feature, INCONCLUSIVE when
evidence or the specification does not support a decision, and NOT-RUN
for a planned case. These are local report statuses, not protocol errors.
A report should state its coverage denominator and excluded cases;
one aggregate pass percentage is insufficient to describe interoperability.

## Minimum Coverage {#minimum-coverage}

The following groups provide an initial test inventory. They are
recommended coverage, not additional SATP conformance requirements.

<table markdown="0">
<thead>
<tr>
<th align="left">Group</th>
<th align="left">Representative cases</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Schema and encoding</td>
<td align="left">Missing or malformed fields, unsupported versions, numeric and Unicode boundaries.</td>
</tr>
<tr>
<td align="left">Protection</td>
<td align="left">Wrong signing key, altered content, bad claim proof, incorrect hash input.</td>
</tr>
<tr>
<td align="left">Core decisions</td>
<td align="left">Successful flow, invalid ordering, wrong session, protocol error, application abort.</td>
</tr>
<tr>
<td align="left">Network effects</td>
<td align="left">Repeated lock, repeated creation or assignment, backend rejection, conflicting transfers.</td>
</tr>
<tr>
<td align="left">Delivery and restart</td>
<td align="left">Lost response, duplicate input, disconnect, crash around an irreversible action.</td>
</tr>
<tr>
<td align="left">Setup and Registry</td>
<td align="left">Missing prerequisite, mismatched context, changed TAR version, unauthorized update.</td>
</tr>
</tbody>
</table>

Each delivery failure should be injected at a named boundary: before
receipt, after validation, after backend execution, or after persistence
but before response delivery. Random fault injection can supplement
these cases after deterministic tests provide a reproducible baseline.

Transport failure tests should include authentication errors, wrong
protocol mapping, deadline expiry, inconsistent HTTP error status, and
malformed signed errors. A transport success response alone is not
evidence of an accepted SATP message or a completed transfer.

## Both Roles and Independent Oracles {#both-roles}

The initial matrix should include independent-peer sender to Hermes
receiver, and Hermes sender to independent-peer receiver. Two instances
of the new peer are useful for development but should be reported as
same-implementation tests. Optional module results should name the
actual counterpart; self-tests should not become cross-implementation
claims when Hermes does not support the module.

The harness should compare observed results with a small independently
defined transition model and direct network observations. Sharing wire
schemas and public fixtures can improve compatibility. Sharing the
counterpart's state engine or treating every counterpart response as
the expected answer would reduce the value of an independent test.

Failure triage should distinguish a specification ambiguity, a
counterpart mismatch, a new-peer defect, a transport mismatch, and an
unsupported feature. Evidence for a specification issue should include
the relevant revision and section, a minimal input, the competing
interpretations, the observed effect, and a proposed clarification.
Avoid assigning implementation blame before identifying the source of
the expected behavior.

# Open Source Implementation and Development Sequence {#open-source}

## Initial Counterpart and Implementation Status {#implementation-status}

{{HERMES}} is a public implementation selected as an initial
interoperability counterpart. This guide does not claim that a survey
has established it to be the only SATP implementation, or that a
particular release implements every requirement of the baseline.
The referenced snapshot is evidence for the compatibility investigation,
not a validated compatibility declaration.

The independent peer described here is a proposed implementation.
This revision assigns no repository or release to it, and reports no
completed protocol or interoperability test run. The procedures in
this guide describe how to conduct and report future experiments.

## Language and Development Environment {#development-environment}

Go is a practical initial target for the proposed peer: explicit data
types, the standard HTTP and cryptographic libraries, a small executable,
and built-in testing support suit a controlled gateway and test driver.
Connect-Go {{CONNECT}} provides a way to exercise native gRPC and
gRPC-Web while retaining an explicit choice of mapping. This is an
engineering choice for the experiment, not a language requirement for
SATP. Framework version and default protocol selection should be pinned.

A minimal repository can separate the Core engine, codecs, transports,
network adapters, journal, Setup and Registry modules, fixtures, and
command-line test driver. Protobuf generation can use protoc or Buf with fixed
generator versions. The initial deployment can run on Linux or WSL2 and
use Docker Compose to start the counterpart and mock networks. It need
not introduce Kubernetes, multiple production ledgers, or a large
workflow framework.

Python is also a reasonable option when the team is substantially more
experienced with it, particularly for scenario modeling. Choosing a
native gRPC library is not sufficient to establish compatibility with a
gRPC-Web counterpart. The same signed-message feasibility check and
role matrix should precede a larger Python implementation.

## Incremental Development Gates {#development-gates}

<ol markdown="0">
<li><t>Capture the baseline and counterpart contract: specification
revisions, commit, schemas, paths, credentials, protected content,
and expected message semantics.</t></li>
<li><t>Exchange one protected message and response across languages.
Compare signed bytes and verify signatures and hashes. Include a
negative case before expanding the protocol implementation.</t></li>
<li><t>Implement the Core engine and mock networks. Exercise success,
rejection, and irreversible-operation boundaries without depending
on a remote gateway.</t></li>
<li><t>Run both roles against the counterpart, keeping HTTP conformance
and Hermes compatibility results separate.</t></li>
<li><t>Add Setup entry checks and a pre-populated Registry; then add
registry version, signature, and archive scenarios.</t></li>
</ol>

Each gate should produce reviewable fixtures, traces, and a coverage
report. Tests that depend on unresolved choices can remain INCONCLUSIVE
while unaffected cases proceed. A complete production gateway is not a
prerequisite for providing useful feedback to the working group.

## Reproducibility and Revision Tracking {#reproducibility}

Publish implementation commits, build instructions, fixture versions,
container digests when used, public test credentials, and the manifest
needed to repeat an experiment. Live private credentials and production
data do not belong in that package. A counterpart update should trigger
a documented comparison of schemas, protected bytes, and required
behavior rather than an unreviewed change to expected outputs.

The guide should track every substantive source change to affected
implementation modules and tests. Changes to extensions should not
silently change the Core baseline. An issue log can connect each open
question to its provisional profile setting and eventual resolution.

# Security Considerations {#security}

The security considerations of the selected specifications continue to
apply. A test environment should make its trust assumptions explicit;
limited scope is not a reason to accept unauthenticated or tampered
messages in a test that claims to validate protection.

## Identity, Keys, and Representation {#security-identities}

Authenticate gateway and registry endpoints, then separately establish
the authorized message signer and the meaning of supplied evidence.
Keep transport authentication, message authentication, asset
authorization, and ledger proof verification as distinct checks. An
attacker-controlled key in a syntactically valid JWS should not become
a trust anchor merely because it verifies that same JWS.

Fixtures should identify accepted algorithms and key purposes. Tests
should reject algorithm substitution, mismatched identities, corrupted
signatures, and incorrect encoding. Serializing and verifying structured
content across languages needs an agreed byte contract, particularly
when a transport adapter changes the representation.

## State, Replay, and Irreversible Actions {#security-state}

Bind accepted inputs to the expected session, context, role, and state
using fields and checks supported by the selected mapping. Duplicate
detection should precede repeated backend effects. A persistent
operation association helps diagnose whether a response was lost after
an action, but does not authorize an undefined recovery exchange.

Expiry, connection loss, and application abort should not be treated as
proof that an irreversible operation did not happen. Cleanup should be
justified by asset state and the selected protocol's abort semantics.
Unknown outcomes should retain evidence and avoid actions that could
make value available to both parties.

Setup should authenticate the participants and gateway pair associated
with an asset set. A context substitution or change to a referenced
TAR should be visible before Core entry. Both gateways should
validate the context and artefact snapshot used for that entry.

## Exposure and Availability {#security-exposure}

Treat configured gateway and registry endpoints, artefact content,
and asset-operation arguments as untrusted input. The controlled
experiment can use explicit endpoint and operation allowlists. Limit
decoding depth, message and claim size, signature work, pending
contexts, and request duration. Registry lookup should not enable
unrestricted network access or arbitrary code execution.

A valid TLS connection or Registry response does not prove that an
asset exists, belongs to the requester, or has the asserted finality.
Those checks belong to the relevant network and evidence model.
Availability or replication of a registry also does not remove the
need to establish trust in the artefacts it serves.

# Privacy Considerations {#privacy}

Tests should use synthetic identities and asset data. Logs should record
identifiers and evidence sufficient for diagnosis while minimizing
sensitive payloads. Published traces should omit tokens, private keys,
personal data, and restricted artefact content. An append-only history
requires particular care because later archive does not erase earlier
data.

The PII scenario in revision -02 should be treated as an application
and policy integration example. This guide does not add a mandatory
<tt>compliancePayload</tt>, a universal Travel Rule schema, or a regulatory
compliance claim to Core. If a deployment uses JWE {{RFC7516}}, it should
identify recipients, key handling, protected content, and authorization
independently of the Core transfer exchange. Encryption alone does not
establish permission to disclose or retain data.

Registry access and failure diagnostics should disclose only the
artefact information authorized for the caller. A valid signature
does not by itself grant permission to retrieve sensitive metadata.

# IANA Considerations {#iana}

This document requests no IANA action. Example profile names, local
operation names, manifest fields, and report statuses are not registered
SATP identifiers or protocol code points.

--- back

# Items for Subsequent Revisions {#outstanding-work}

This section is editorial material and can be removed before publication
as an RFC.

The following items remain open rather than being completed by this
revision:

<ul markdown="0">
<li><t>Reconcile this guide with the next published Core transport treatment
and any resulting standard bindings.</t></li>
<li><t>Resolve the Core naming, serialization, hash, and abort questions in
<xref target="core-clarifications"/> through the working group.</t></li>
<li><t>Publish the independent implementation and complete protected-message
fixtures after the cross-language feasibility check.</t></li>
<li><t>Add observed both-role results, including counterpart feature limits
and unresolved failures.</t></li>
<li><t>Clarify Setup wire contracts, authenticated binding and expiry, and
Registry routes, error mappings, and version selection with the
respective draft authors.</t></li>
<li><t>Consider Views and Data Sharing guidance in a later revision.</t></li>
<li><t>Add real-network evidence adapters only with explicit finality and
authorization assumptions and separately reported results.</t></li>
</ul>

# Acknowledgments
{:numbered="false"}

The authors thank the SATP working group and implementation contributors
for discussions that motivate independent implementations and
reproducible interoperability testing.
