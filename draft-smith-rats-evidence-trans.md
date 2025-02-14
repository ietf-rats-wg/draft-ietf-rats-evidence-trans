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
  org: Altera
  email: andrew.draper@altera.com
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
    seriesinfo: Version 1.2, Revision 1
    date: January 2025
    target: https://trustedcomputinggroup.org/wp-content/uploads/DICE-Attestation-Architecture-Version-1.2-rc-1_9January25.pdf
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
  RFC7942:
  DICE.Layer:
    title: DICE Layering Architecture
    author:
      org: Trusted Computing Group
    seriesinfo: Version 1.0, Revision 0.19
    date: July 2020
    target: https://trustedcomputinggroup.org/wp-content/uploads/DICE-Layering-Architecture-r19_pub.pdf
  I-D.ietf-rats-eat: eat
  RFC5280: x509

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
There are many possible Evidence structures including {{-eat}} and {{-x509}}.
The bytes composing the CoRIM data structure are the same either way.

The terminology from CoRIM {{-corim}}, CBOR {{-cbor}}, CDDL {{-cddl}} and COSE {{-cose}} applies.

{::boilerplate bcp14}

# Verifier Reconciliation {#sec-verifier-rec}

This specification assumes the reader is familiar with Verifier Reconsiliation as described in {{Section 2 of -corim}}.
It describes how a Verifier should process the CoRIM to enable CoRIM authors to convey their intended meaning and how a Verifier reconciles its various inputs.
Evidence is one of its inputs.
The Verifier is expected to create an internal representation from an external representation.
By using an internal representation, the Verifier processes Evidence inputs such that they can be appraised consistently.

This specification defines format transformations for Evidence in DICE {{-dice-attest}}, SPDM {{-spdm}}, and concise evidence {{-ce}} formats that are transformed into a Verifier's internal representation.
This specification uses the CoMID internal representation ({{Section 8.2.1 of -corim}}) as the transformation target.
Other internal representations are possible but out of scope for this specification.

# Transforming DICE Certificate Extension Evidence {#sec-dice-trans}

This section defines how Evidence from an X.509 certificate containing a DICE certificate extension {{-dice-attest}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the DICE certificate extension Evidence SHOULD implement this transformation.

This specification defines transformation methods for two DICE certificate extensions DiceTcbInfo and DiceMultiTcbInfo.
These extensions are identified by the following object identifiers:

* tcg-dice-TcbInfo - "2.23.133.5.4.1"

* tcg-dice-MultiTcbInfo - "2.23.133.5.4.5"

Each DiceTcbInfo entry in a MultiTcbInfo is converted to a CoRIM ECT (see {{Section 8.2.1 of -corim}}) using the transformation steps in this section.
Each DiceMultiTcbInfo entry is independent of the others such that each is transformed to a separate ECT entry.
A list of Evidence ECTs (i.e., `ae = [ + ECT]`) is constructed using CoRIM attestation evidence internal representation (see {{Section 8.2.1.1 of -corim}}).

For each DiceTcbInfo (DTI) entry in a DiceMultiTcbInfo allocate an ECT structure.

{:dtt-enum: counter="dtt" style="format Step %d."}

{: dtt-enum}
* An `ae` entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The DiceTcbInfo (DTI) entry populates the `ae` ECT.

{:dtt2-enum: counter="dtt2" style="format %i"}

{: dtt2-enum}
* The DTI entry populates the `ae` ECT `environment-map`

> > **copy**(DTI.`type`, ECT.`environment`.`environment-map`.`class-map`.`class-id`).
The binary representation of DTI.`type` MUST be equivalent to the binary representation of `class-id` without the CBOR tag.

> > **copy**(DTI.`vendor`, ECT.`environment`.`environment-map`.`class-map`.`vendor`).

> > **copy**(DTI.`model`, ECT.`environment`.`environment-map`.`class-map`.`model`).

> > **copy**(DTI.`layer`, ECT.`environment`.`environment-map`.`class-map`.`layer`).

> > **copy**(DTI.`index`, ECT.`environment`.`environment-map`.`class-map`.`index`).

{: dtt2-enum}
* The DTI entry populates the `ae` ECT `elemenet-list`.

> > **copy**(DTI.`version`, ECT.`element-list`.`element-map`.`measurement-values-map`.`version-map`.`version`).

> > **copy**(DTI.`svn`, ECT.`element-list`.`element-map`.`measurement-values-map`.`svn`).

> > **copy**(DTI.`vendorInfo`, ECT.`element-list`.`element-map`.`measurement-values-map`.`raw-value`).

> > Foreach FWID in FWIDLIST: **copy**(DTI.`FWID`.`digest`, ECT.`element-list`.`element-map`.`measurement-values-map`.`digests`.`digest`.`val`).

> > Foreach FWID in FWIDLIST: **copy**(DTI.`FWID`.`hashAlg`, ECT.`element-list`.`element-map`.`measurement-values-map`.`digests`.`digest`.`alg`).

{: dtt2-enum}
* The DTI entry populates the `ae` ECT `elemenet-list`.`flags`. Foreach _f_ in DTI.`OperationalFlags` and each _m_ in DTI.`OperationalFlagsMask`:

> > If _m_.`notConfigured` = 1 AND _f_.`notConfigured` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-configured` = FALSE).

> > If _m_.`notConfigured` = 1 AND _f_.`notConfigured` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-configured` = TRUE).

> > If _m_.`notSecure` = 1 AND _f_.`notSecure` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-secure` = FALSE).

> > If _m_.`notSecure` = 1 AND _f_.`notSecure` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-secure` = TRUE).

> > If _m_.`recovery` = 1 AND _f_.`recovery` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-recovery` = FALSE).

> > If _m_.`recovery` = 1 AND _f_.`recovery` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-recovery` = TRUE).

> > If _m_.`debug` = 1 AND _f_.`debug` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-debug` = FALSE).

> > If _m_.`debug` = 1 AND _f_.`debug` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-debug` = TRUE).

> > If _m_.`notReplayProtected` = 1 AND _f_.`notReplayProtected` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-replay-protected` = FALSE).

> > If _m_.`notReplayProtected` = 1 AND _f_.`notReplayProtected` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-replay-protected` = TRUE).

> > If _m_.`notIntegrityProtected` = 1 AND _f_.`notIntegrityProtected` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-integrity-proteccted` = FALSE).

> > If _m_.`notIntegrityProtected` = 1 AND _f_.`notIntegrityProtected` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-integrity-proteccted` = TRUE).

> > If _m_.`notRuntimeMeasured` = 1 AND _f_.`notRuntimeMeasured` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-runtime-meas` = FALSE).

> > If _m_.`notRuntimeMeasured` = 1 AND _f_.`notRuntimeMeasured` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-runtime-meas` = TRUE).

> > If _m_.`notImmutable` = 1 AND _f_.`notImmutable` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-immutable` = FALSE).

> > If _m_.`notImmutable` = 1 AND _f_.`notImmutable` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-immutable` = TRUE).

> > If _m_.`notTcb` = 1 AND _f_.`notTcb` = 1; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-tcb` = FALSE).

> > If _m_.`notTcb` = 1 AND _f_.`notTcb` = 0; **set**(ECT.`element-list`.`element-map`.`measurement-values-map`.`flags`.`is-tcb` = TRUE).

{: dtt-enum}
* The ECT.`authority` field is set up based on the signer of the certificate containing DTI as described in {{sec-authority}}.

The completed ECT is added to the `ae` list.

## Authority field in DICE/SPDM ECTs {#sec-authority}

The ECT authority field is an array of `$crypto-keys-type-choice`s.

When adding Evidence to the ACS, the Verifier SHALL add the public key representing the signer of that Evidence (for example the DICE certificate or SPDM MEASUREMENTS response) to the ECT authority field.
The Verifier SHALL also add the signer of each certificate which has authorized the signer of the signing key.

Having each authority in a certificate path in the ECT `authority` field lets conditional endorsement conditions match multiple authorities or match an authority that is scoped more broadly than the immediate signer of the Evidence artifact.

Each signer authority value MUST be represented using `tagged-cose-key-type`.

# Transforming TCG Concise Evidence {#sec-ce-trans}

This section defines how Evidence from TCG {{-ce}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the TCG Concise Evidence format SHOULD implement this transformation.

Concise evidence may be recognized by any of the following registered types:

| CBOR tag    | C-F ID    | TN Tag    | Media Type         |
|-------------|-----------|-----------|--------------------|
| #6.571 | 10571 | #6.1668557429 | "application/ce+cbor" |

A Concise Evidence entry is converted to a CoRIM ECT (see {{Section 8.2.1 of -corim}}) using the transformation steps in this section.
A list of Evidence ECTs (i.e., `ae = [ + ECT]`) is constructed using CoRIM attestation evidence internal representation (see {{Section 8.2.1.1 of -corim}}).
The Concise Evidence scheme uses CoRIM CDDL definitions to define several Evidence representations called _triples_.
Cases where Concise Evidence CDDL is identical to CoRIM CDDL the transformation logic uses the structure names in common.

## Transforming the ce.evidence-triples {#sec-evidence-triple}

The `ce.evidence-triples` structure is a list of `evidence-triple-record`.
An `evidence-triple-record` consists of an `environment-map` and a list of `measurement-map`.
For each `evidence-triple-record` and `ae` ECT is constructed.

{:cet-enum: counter="cet" style="format Step %d."}

{: cet-enum}
* An `ae` ECT entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The Concise Evidence (CE) entry populates the `ae` ECT `environment` fields.

> > **copy**(CE.`evidence-triple-record`.`environment-map`, ECT.`environment`.`environment-map`).

{:cet2-enum: counter="cet2" style="format %i"}

{: cet2-enum}

* For each e in CE.`[ + measurement-map]`:

> > **copy**(e.`measurement-map`.`mkey`, ECT.`element-list`.`element-map`.`element-id`)

> > **copy**(e.`measurement-map`.`mval`, ECT.`element-list`.`element-map`.`element-claims`)

{: cet-enum}
* The signer of the envelope containing CE is copied to the ECT.`authority` field as described in {{sec-authority}.
For example, a CE may be wrapped by an EAT token {{-eat}} or DICE certificate {{-dice-attest}}.
The signer identity MUST be expressed using `$crypto-key-type-choice`.
A profile or other arrangement is used to coordinate which `$crypto-key-type-choice` is used for both Evidence and Reference Values.

* If CE has a profile, the profile is converted to a `$profile-type-choice` then copied to the ECT`.`profile` field.

The completed ECT is added to the `ae` list.

## Transforming the ce.identity-triples {#sec-identity-triple}

## Transforming the ce.attest-key-triples {#sec-attest-key-triple}

# Transforming SPDM Evidence {#sec-spdm-trans}

This section defines how Evidence from SPDM {{-spdm}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the SPDM Evidence format SHOULD implement this transformation.

The SPDM measurements are converted to `concise-evidence` which has a format that is similar to CoRIM `triples-map` (their semantics follows the matching rules described above).
The TCG DICE Concise Evidence Binding for SPDM specification {{-ce}} describes a process for converting the SPDM Measurement Block to Concise Evidence.
Subsequently the transformation steps defined in {{sec-ce-trans}}.

The keys provided in the ECT.`authority` field SHOULD include the key which signed the SPDM MEASUREMENTS response carrying the Evidence as described in {{sec-authority}}, the DeviceID key which authorized that key and keys which authorized the DeviceID key.```

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

