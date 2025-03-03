---
stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  tocdepth: 4
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: o-*+
  compact: yes
  subcompact: no

title: Secure Asset Transfer Protocol (SATP) Implementation Guide
abbrev: SATP Implementation Guide
docname: draft-song-satp-implementation-guide-latest
category: info
ipr: trust200902
area: Applications and Real-Time
workgroup: Secure Asset Transfer Protocol
date: 3 March 2025
v: 3
stream: IETF
keyword:
 - Gateway
 - Digital asset network
 - Asset transfer
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
 -
    ins: Y-G. Hong
    fullname: Yong-Geun Hong
    organization: Daejeon Univ.
    email: yonggeun.hong@gmail.com
 -
    ins: T. Hardjono
    name: Thomas Hardjono
    organization: MIT
    email: hardjono@mit.edu

informative:
  SATcore:
    author:
    - ins: M. Hargreaves
    - ins: T. Hardjono
    - ins: R. Belchior
    - ins: V. Ramakrishna
    date: Jan 2025
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-core/
    title: Secure Asset Transfer Protocol (SATP) Core
  SATarch:
    author:
    - ins: T. Hardjono
    - ins: M. Hargreaves
    - ins: N. Smith
    - ins: V. Ramakrishna
    date: Dec 2024
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-architecture/
    title: Secure Asset Transfer (SAT) Interoperability Architecture

--- abstract

This memo provides guidelines to developers of gateway systems, digital asset networks and client applications for the Secure Asset Transfer Protocol (SATP). Multiple gateways can represent the same digital asset network following the SATP standards, which necessitate basic implementation guidelines as outlined in this document. It also serves as an introduction to the SATP processing workflow for those new to the SATP standards.

(Note. the initial draft (00) is submitted for a brief review regarding the document's direction by SATP WG members during IETF 12 in Bangkok, Mar 2025)

--- middle

# Introduction

{: #introduction-doc}

This document proposes implementation guidance from the perspective of developers of peer gateways and digital asset networks.

The Secure Asset Transfer Protocol (SATP) facilitates communication between peer gateways and other entities such as digital asset networks and client applications. A digital asset network can be connected to multiple peer gateways to enable asset transfers while ensuring interoperability. To ensure consistency across multiple peer gateways, appropriate guidelines are necessary.

Peer gateways must implement a secure asset transfer mechanism that meets essential requirements related to atomicity,
consistency, isolation, and durability (ACID). Security and privacy are also critical requirements. Therefore, this document involves several considerations beyond the SATP interface.

The existing SATP core documents ([SATcore], [SATarch]) provides most of the key implementation guidelines, but this document offers a more elaborated description of each phase within peer gateways and digital asset networks.

              Originator                         Beneficiary
           (Origin network)                 (Destination network)
                   |                                   |
      +------------------------+          +------------------------+
      |         Client         |          |         Client         |
      |       Application      |          |       Application      |
      |         (App1)         |          |         (App2)         |
      +------------------------+          +------------------------+
           |              |                    |              |
           V              V                    V              V
      +---------+    +---------+          +---------+    +---------+
      |         |    |         |          |         |    |         |
      |         |    |         |          |         |    |         |
      | Digital |    |         |          |         |    | Digital |
      |  Asset  |    | Gateway |          | Gateway |    |  Asset  |
      | Network |<-->|   GW1   | <------> |   GW2   |    | Network |
      |   NW1   |    |         |          |         |    |   NW2   |
      |         |    |         |          |         |    |         |
      |         |    |         |          |         |    |         |
      +---------+    +---------+          +---------+    +---------+

                Figure 1. Scope of the SATP implementation

    



{: #satp-fig-overview}

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview of Architecture

TBA

# Gateway Implementation

TBA

# Network Implementation

TBA

# Implementation Report

TBA

# Security Considerations

TBA

# Privacy Considerations

TBA

# IANA Considerations

There are no IANA considerations related to this document.


--- back

# Acknowledgments
{:numbered="false"}

TBA
