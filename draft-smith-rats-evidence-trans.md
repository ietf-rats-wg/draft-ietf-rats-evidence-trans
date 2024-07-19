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

# DICE / SPDM input data

An SPDM Requester sends commands to the Attestation Environment which is part of the SPDM Responder.
The responses to those commands include a certificate chain containing DICE measurements and SPDM MEASUREMENTS RESPONSE, which contains SPDM measurements.

# ACS Entries generated from DICE / SPDM input data

DICE/SPDM evidence is transformed into one or more ACS entries, each of which is created from upto four separate components:
- Environment class provides a unique name, which is the same for all instances of the Target Environment
- Environment instance provides an identifier which differes for each instance of the TE
- Fields in measurement map provide measurements of parts of the TE named in environment
- Authorized-by indicates the root key used to authorize the key chain leading to SPDM signatures

The sections below describe how these fields are filled in for different types of measurements

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

# Populating the environment instance field

A verifier transforming evidence from DICE/SPDM to CoMID may choose to add an `instance` key to the `environment-map` of the ACS entries it is creating.
If it does this then a verifier can distinguish between evidence from two Target Environments of the same type, for example two physically identical network cards.
Physically identical TEs may be running different versions of firmware, so each must be verified separately.

The verifier SHOULD add an `instance` key to the `environment-map` of each ACS entry created from the DICE/SPDM measurement information.

If the verifier adds such an `instance` key then the corresponding value SHOULD be set to the key which represents the hardware identity, that is the key which does not change when the measurements of the device change.
The hardware identity key is the subject key of a certificate containing one of the hardware identity extensions below.
If multiple certificates in the chain contain one of the hardware identity extensions then the verifier SHOULD use the subject key from the certificate closest to the root key.

The OIDs which identify the certificate containing the hardware identity key are:

- joint-iso-itu-t(2) international-organizations(23) tcg(133) platformClass(5) tcg-dice(4) tcg-dice-kp(100) identityInit(6)
- joint-iso-itu-t(2) international-organizations(23) tcg(133) platformClass(5) tcg-dice(4) tcg-dice-kp(100) identityLoc(6)
- iso(1) identified-organization(3) dod(6) internet(1) private(4) enterprise(1) dmtf(412) spdm(274) hardwareIdentity(2)

The hardware entity key SHOULD be represented using a format which contains a single key, for example `tagged-cose-key-type`.

# Populating the authorized-by field

A verifier transforming evidence from DICE/SPDM to CoMID SHOULD include `authorized-by` field in each `measurement-values-map` indicating the entity which is responsible for ensuring that the evidence is a valid measurement of the current state of the Target Environment.
This is often the entity which manufactured or configured the Attestation Environment which measured the Target Environment.

For measurements translated from DICE/SPDM the authorized-by field should contain the key at the root of the certificate chain returned in the SPDM CERTIFICATES response message.
The value of the `authorized-by` entry SHOULD use a format which contains a single key, for example `tagged-cose-key-type`.

## Organisational unit changes within authorized-by

Some DICE root keys are owned by a registry, which authorise lower level keys of different organisations.
If a registry has authorized both organisations A and B to authorize certificates which can be used to authorize SPDM responses then there is a possibility that organisation A might generate fake responses purporting to authorise organisation B's attesters.

To make this detectable, the registry may indicate when it signs a certificate authorizing a key controlled by a different organization.
This indication is copied to an ACS entry, and can be matched against a CoMID file.

If the subject key of a certificate is not controlled by the same organisation as the issuer key then the issuer SHOULD include an OU field in the subjectName field. The OU value indicates the entry which controls the subject key in the certificate.

For example, the subject key might have the value `CNAME="Example Corporation" OU=1234`.

A verifier processing a certificate containing an OU field in its subjectName field should generate an ACS entry with these values:
- `environment-map / class / class-id` SHALL be set to the value <OID for OU>
- `environment-map / class / vendor` SHALL be set to the string representation of the issuerName
- `measurement-map / mkey / raw-value` SHALL be set to the numeric value of the OU field

The verifier can include a reference value or the condition code within a conditional-environment triple to ensure that the key chain anchored on the registry was authorizing the correct organisation.

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

