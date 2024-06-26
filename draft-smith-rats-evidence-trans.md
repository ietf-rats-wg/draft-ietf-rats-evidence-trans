---
v: 3

title: Evidence Transformations
abbref: EvTrans
docname: draft-smith-rats-evidence-trans-latest
category: std
consensus: true
submissiontype: IETF

ipr: trust200902
area: "Security"
workgroup: "Remote ATtestation ProcedureS"
keyword: Evidence, RATS, attestation, verifier, supply chain, RIM, appraisal

stand_alone: true
pi:
  toc: yes
  sortrefs: yes
  symrefs: yes
  tocdepth: 6

author:
- ins: A. Draper
  name: Andrew Draper
  org: Intel
  email: andrew.draper@intel.com
- ins: N. Smith
  name: Ned Smith
  org: Intel
  email: ned.smith@intel.com

normative:
  I-D.ietf-rats-corim: corim
  DICE.CoRIM:
    -: dice-corim
    title: DICE Endorsement Architecture for Devices
    author:
      org: Trusted Computing Group (TCG)
    seriesinfo: Version 1.0, Revision 0.38
    date: November 2022
    target: https://trustedcomputinggroup.org/wp-content/uploads/TCG-Endorsement-Architecture-for-Devices-V1-R38_pub.pdf
  DICE.Attest:
    -: dice-attest
    title: DICE Attestation Architecture
    author:
      org: Trusted Computing Group (TCG)
    seriesinfo: Version 1.1, Revision 18
    date: January 2024
    target: https://trustedcomputinggroup.org/wp-content/uploads/DICE-Attestation-Architecture-Version-1.1-Revision-18_pub.pdf
  RFC9334: rats-arch
  X.690:
    -: x690
    title: >
      Information technology — ASN.1 encoding rules:
      Specification of Basic Encoding Rules (BER), Canonical Encoding
      Rules (CER) and Distinguished Encoding Rules (DER)
    author:
      org: International Telecommunications Union
    date: 2015-08
    seriesinfo:
      ITU-T: Recommendation X.690
    target: https://www.itu.int/rec/T-REC-X.690
  SPDM:
    -: spdm
    title: Security Protocol and Data Model (SPDM)
    author:
      org: Distributed Management Task Force
    seriesinfo: Version 1.3.0
    date: May 2023
    target: https://www.dmtf.org/sites/default/files/standards/documents/DSP0274_1.3.0.pdf
  TCG.CE:
    -: ce
    title: TCG DICE Concise Evidence Binding for SPDM
    author:
      org: Trusted Computing Group
    seriesinfo: Version 1.00, Revision 0.54
    date: January 2024
    target: https://trustedcomputinggroup.org/wp-content/uploads/TCG-DICE-Concise-Evidence-Binding-for-SPDM-Version-1.0-Revision-54_pub.pdf
  I-D.ietf-rats-endorsements: rats-endorsements

informative:
  RFC4122: uuid
  RFC7468: pkix-text
  RFC8610: cddl
  RFC9090: cbor-oids
  STD96:
    -: cose
    =: RFC9052
  STD94:
    -: cbor
    =: RFC8949
  STD66:
    -: uri
    =: RFC3986
  RFC9393: coswid
  RFC7942:
  I-D.fdb-rats-psa-endorsements: psa-endorsements
  I-D.tschofenig-rats-psa-token: psa-token
  DICE.Layer:
    title: DICE Layering Architecture
    author:
      org: Trusted Computing Group
    seriesinfo: Version 1.0, Revision 0.19
    date: July 2020
    target: https://trustedcomputinggroup.org/wp-content/uploads/DICE-Layering-Architecture-r19_pub.pdf
  IANA.coswid: coswid-reg
  I-D.ietf-rats-eat: eat
  I-D.ietf-rats-concise-ta-stores: ta-store

entity:
  SELF: "RFCthis"

--- abstract

Remote Attestation Procedures (RATS) enable Relying Parties to assess the trustworthiness of a remote Attester and therefore to decide whether to engage in secure interactions with it - or not.
Evidence about trustworthiness can be rather complex and it is deemed unrealistic that every Relying Party is capable of the appraisal of Evidence.
Therefore that burden is typically offloaded to a Verifier.
In order to conduct Evidence appraisal, a Verifier requires fresh Evidence from an Attester.
Before a Verifier can appraise Evidence it may require transformation to an internal representation.
This document specifies Evidence transformation methods for DICE and SPDM formats to the CoRIM internal representation.

--- middle

# Introduction {#sec-intro}

Remote Attestation Procedures (RATS) enable Relying Parties to assess the trustworthiness of a remote Attester and therefore to decide whether to engage in secure interactions with it - or not.
Evidence about trustworthiness can be rather complex and it is deemed unrealistic that every Relying Party is capable of the appraisal of Evidence.
Therefore that burden is typically offloaded to a Verifier.
In order to conduct Evidence appraisal, a Verifier requires fresh Evidence from an Attester.
Before a Verifier can appraise Evidence it may require transformation to an internal representation.
This document specifies Evidence transformation methods for DICE and SPDM formats to the CoRIM internal representation.

## Terminology

This document uses terms and concepts defined by the RATS architecture.
For a complete glossary see {{Section 4 of -rats-arch}}.
Addintional RATS architecture is found in {{-rats-endorsements}}.
RATS architecture terms and concepts are always referenced as proper nouns, i.e., with Capital Letters.

In this document, an Evidence structure describes an external representation.
There are many possible Evidence structures including {{-eat}}
The bytes composing the CoRIM data structure are the same either way.

The terminology from CoRIM {{-corim}}, CBOR {{-cbor}}, CDDL {{-cddl}} and COSE {{-cose}} applies.

{::boilerplate bcp14}

# Verifier Reconciliation {#sec-verifier-rec}

This specification assumes the reader is familiar with Verifier Reconsiliation as described in {{-corim}}.
It describes how a Verifier should process the CoRIM to enable CoRIM authors to convey their intended meaning and how a Verifier reconciles its various inputs.
Evidence is one of its inputs.
The Verifier is expected to create an internal representation from an external representation.
By using an internal representation, the Verifier processes Evidence inputs such that they can be appraised consistently.

This specification describes how Evidence in DICE {{-dice-attest}}, SPDM {{-spdm}}, and concise evidence {{-ce}} formats are transformed into the CoRIM {{-corim}} internal representation.
If other internal representations exist, a similar specification may be required that transforms Evidence to some other internal representation.

# Transforming SPDM Evidence {#sec-spdm-trans}

This section defines how Evidence from SPDM {{-spdm}} is transformed into a format where it can be added to an appraisal claims set.
A Verifier supporting SPDM format Evidence should implement this section.

The TCG DICE Concise Evidence Binding for SPDM specification {{-ce}} describes the process by which measurements in an SPDM Measurement Block are converted to Evidence suitable for matching using the rules below.
The SPDM measurements are converted to `concise-evidence` which has a format that is similar to CoRIM `triples-map` (their semantics follows the matching rules described above).

# Transforming DICE Evidence {#sec-dice-trans}

This section defines how Evidence from DICE {{-dice-attest}} is transformed into a format where it can be added to an appraisal claims set.
A Verifier supporting DICE format Evidence should implement this section.

DICE Evidence appears in certificates in the TcbInfo or MultiTcbInfo extension.
Each TcbInfo, and each entry in the MultiTcbInfo, is converted to an `endorsed-triple-record` using the rules in this section.
In a MultiTcbInfo each entry in the sequence is treated as independent and translated into a separate Evidence object.

The Verifier SHALL translate each field in the TcbInfo into a field in the created endorsed-triple-record

- The TcbInfo `type` field SHALL be copied to the field named `environment-map / class / class-id` and tagged with tag #6.111
- The TcbInfo `vendor` field SHALL be copied to the field named `environment-map / class / vendor`
- The TcbInfo `model` field SHALL be copied to the field named `environment-map / class / model`
- The TcbInfo `layer` field SHALL be copied to the field named `environment-map / class / layer`
- The TcbInfo `index` field SHALL be copied to the field named `environment-map / class / index`

- The TcbInfo `version` field SHALL be translated to the field named `measurement-map / mval / version / version`
- The TcbInfo `svn` field SHALL be copied to the field named `measurement-map / mval / svn`
- The TcbInfo `fwids` field SHALL be translated to the field named `measurement-map / mval / digests`
  - Each digest within fwids is translated to a CoMID digest object, with an appropriate algorithm identifier
- The TcbInfo `flags` field SHALL be translated to the field named `measurement-map / mval / flags`
  - Each flag is translated independently
- The TcbInfo `vendorInfo` SHALL shall be copied to the field named `measurement-map / mval / raw-value`

If there are multiple `endorsed-triple-record`s with the same `environment-map` then they MUST be merged into a single entry.
If the `measurement-values-map` fields in Evidence triples have conflicting values then the Verifier MUST fail validation.

# Transforming Concise Evidence {#sec-ce-trans}

# Security and Privacy Considerations {#sec-sec}

There are no security and privacy considerations.

# IANA Considerations {#sec-iana-cons}

There are no IANA considerations.

--- back


# Contributors
{:unnumbered}

The authors would like to thank the following people for their valuable contributions to the specification.

Henk Birkholz

Email: henk.birkholz@ietf.contact

Yogesh Deshpande

Email: yogesh.deshpande@arm.com

Thomas Fossati

Email: Thomas.Fossati@linaro.org

Dionna Glaze

Email: dionnaglaze@google.com

# Acknowledgments
{:unnumbered}

