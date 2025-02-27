---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###

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
submissiontype: IETF
category: info
ipr: trust200902
area: Applications and Real-Time
workgroup: Secure Asset Transfer Protocol
date: March 2025
v: 3
stream: IETF
keyword:
 - Blockchain
 - Distributed ledger
 - Asset transfer
consensus: true

venue:
  group: Secure Asset Transfer Protocol
  type: Working Group
  mail: sat@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/sat/
  github: anawhj/draft-song-satp-implementation-guide
  latest: https://anawhj.github.io/draft-song-satp-implementation-guide/draft-song-satp-implementation-guide.html

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

normative:
  JWT: RFC7519
  REQ-LEVEL: RFC2119

informative:
  NIST:
    author:
    - ins: D. Yaga
    - ins: P. Mell
    - ins: N. Roby
    - ins: K. Scarfone
    date: October 2018
    target: https://doi.org/10.6028/NIST.IR.8202
    title: NIST Blockchain Technology Overview (NISTR-8202)

  MICA:
    author:
    - ins: European Commission
    date: June 2023
    target: https://www.esma.europa.eu/esmas-activities/digital-finance-and-innovation/markets-crypto-assets-regulation-mica
    title: EU Directive on Markets in Crypto-Assets Regulation (MiCA)

  ARCH:
    author:
    - ins: T. Hardjono
    - ins: M. Hargreaves
    - ins: N. Smith
    - ins: V. Ramakrishna
    date: June 2024
    target: https://datatracker.ietf.org/doc/draft-ietf-satp-architecture/
    title: Secure Asset Transfer (SAT) Interoperability Architecture

  RFC5939:
    author:
    - ins: F. Andreasen
    date: September 2010
    target: https://www.rfc-editor.org/info/rfc5939
    title: Session Description Protocol (SDP) Capability Negotiation

  RFC9334:
    author:
    - ins: H. Birkholz
    - ins: D. Thaler
    - ins: M. Richardson
    - ins: N. Smith
    - ins: W. Pan
    date: January 2023
    target: https://www.rfc-editor.org/info/rfc9334
    title: Remote Attestation Procedures Architecture (RATS)

--- abstract

This memo describes the Secure Asset Transfer (SAT) Implementation Guide. SAT is a protocol operating between two gateways that conducts the transfer of a digital asset from one gateway to another, each representing their corresponding digital asset networks. The protocol establishes a secure channel between the endpoints and implements a 2-phase commit (2PC) to ensure the properties of transfer atomicity, consistency, isolation and durability. The implementation guide provides how to design and utilize the SATP protocol standards.

--- middle

# Introduction

TODO Introduction


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

There are no IANA considerations related to this document.


--- back

# Acknowledgments
{:numbered="false"}

TBA
