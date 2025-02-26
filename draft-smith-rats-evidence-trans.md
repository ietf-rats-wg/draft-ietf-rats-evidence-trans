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
  STD96:
    -: cose
    =: RFC9052
  STD94:
    -: cbor
    =: RFC8949
  I-D.ietf-rats-eat: eat
  RFC5280: x509
  RFC7942:

entity:
  SELF: "RFCthis"

--- abstract

Remote Attestation Procedures (RATS) enable Relying Parties to assess the trustworthiness of a remote Attester to decide if continued interaction is warrented.
Evidence structures can vary making appraisals challenging for Verifiers.
Verifiers need to understand Evidence encoding formats and some of the Evidence semantics to appraise it.
Consequently, Evidence may require format transformation to an internal representation that preserves original semantics.
This document specifies Evidence transformation methods for DiceTcbInfo, concise evidence, and SPDM measurements block structures.
These Evidence structures are converted to the CoRIM internal representation and follow CoRIM defined appraisal procedures.

--- middle

# Introduction {#sec-intro}

Remote Attestation Procedures (RATS) {{-rats-arch}} enable Relying Parties to assess the trustworthiness of a remote Attester to decide if continued interaction is warrented.
Evidence structures can vary making appraisals challenging for Verifiers.
Verifiers need to understand Evidence encoding formats and some of the Evidence semantics to appraise it.
Consequently, Evidence may require format transformation to an internal representation that preserves original semantics.
This document specifies Evidence transformation methods for DiceTcbInfo {{-dice-attest}}, concise evidence {{-ce}}, and SPDM measurements block {{-spdm}} structures.
These Evidence structures are converted to the CoRIM internal representation (Section 2.1 {{-corim}}) and follow CoRIM defined appraisal procedures (Section 8 {{-corim}}).

## Terminology

This document uses terms and concepts defined by the RATS architecture.
For a complete glossary. See {{Section 4 of -rats-arch}}.
Addintional RATS architecture and terminology is found in {{-rats-endorsements}}.
RATS architecture terms and concepts are always referenced as proper nouns, i.e., with Capital Letters.
Additional terminology from CoRIM {{-corim}}, {{-dice-corim}}, CBOR {{-cbor}}, CDDL {{-cddl}} and COSE {{-cose}} may also apply.

In this document, Evidence structures are expressed in their respective "external representations".
There are many possible Evidence structures including those mentioned above.

The CoRIM specification defines an "internal representation" for Evidence (Section 8.2.1.3 {{-corim}}).
This document defines mapping operations that convert from an external representation to an internal representation.
The conversion steps are also known as "transformation".

{::boilerplate bcp14}

# Verifier Reconciliation {#sec-verifier-rec}

This document assumes the reader is familiar with Verifier reconciliation as described in {{Section 2 of -corim}}.
It describes how a Verifier should process the CoRIM to enable CoRIM authors to convey their intended meaning and how a Verifier reconciles its various inputs.
Evidence is one of its inputs.
The Verifier is expected to create an internal representation from an external representation.
By using an internal representation, the Verifier processes Evidence inputs such that they can be appraised consistently.

This document defines format transformations for Evidence in DICE {{-dice-attest}}, SPDM {{-spdm}}, and concise evidence {{-ce}} formats that are transformed into a Verifier's internal representation.
This document uses the CoMID internal representation ({{Section 8.2.1 of -corim}}) as the transformation target.
Other internal representations are possible but out of scope for this document.

# Transforming DICE Certificate Extension Evidence {#sec-dice-trans}

This section defines how Evidence from an X.509 certificate {{-x509}} containing a DICE certificate extension {{-dice-attest}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the DICE certificate Evidence extensions SHOULD implement this transformation.

## DiceTcbInfo Transformation {#sec-tcb-info}

This section defines transformation methods for DICE certificate extensions DiceTcbInfo, DiceMultiTcbInfo, and DiceMultiTcbInfoComp.

These extensions are identified by the following object identifiers:

* tcg-dice-TcbInfo - "2.23.133.5.4.1"

* tcg-dice-MultiTcbInfo - "2.23.133.5.4.5"

* tcg-dice-MultiTcbInfoComp - "2.23.133.5.4.8"

Each DiceTcbInfo entry in a MultiTcbInfo is converted to a CoRIM ECT (see {{Section 8.2.1 of -corim}}) using the transformation steps in this section.
Each DiceMultiTcbInfo entry is independent of the others such that each is transformed to a separate ECT entry.
A list of Evidence ECTs (i.e., `ae = [ + ECT]`) is constructed using CoRIM attestation evidence internal representation (see {{Section 8.2.1.1 of -corim}}).
Each DiceMultiTcbInfoComp entry is converted to a DiceMultiTcbInfo entry then processed as a DiceMultiTcbInfo.

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

## DiceUeid Transformation {#sec-ueid}

This section defines the transformation method for the DiceUeid certificate extension.
This extension is identified by the following object identifier:

* tcg-dice-Ueid - "2.23.133.5.4.4"

{:ueid-enum: counter="ueid" style="format Step %d."}

{: ueid-enum}
* An `ae` entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The DiceUeid entry populates the `ae` ECT `environment-map`.`instance-id`.`tagged-ueid-type`.
The CBOR tag #6.550 is prepended to the DiceUeid OCTET STRING then copied to `ae`.`environment-map`.`instance-id`.

* The ECT.`authority` field is set up based on the signer of the certificate containing DiceUeid as described in {{sec-authority}}.

The completed ECT is added to the `ae` list.

## DiceConceptualMessageWrapper Transformation {#sec-cmw}

This section defines the transformation method for the DiceConceptualMessageWrapper certificate extension.
This extension is identified by the following object identifier:

* tcg-dice-Ueid - "2.23.133.5.4.9"

The DiceConceptualMessageWrapper entry OCTET STRING may contain a CBOR array, JSON array, or CBOR tagged value.
If the entry contains a CBOR tag value of #6.571 or #6.1668557429, or a Content ID of 10571, or a Media Type of "application/ce+cbor",
the contents are transformed according to {{sec-ce-trans}}.

## Authority field in DICE/SPDM ECTs {#sec-authority}

The ECT authority field is an array of `$crypto-keys-type-choice` values.

When adding Evidence to the ACS, the Verifier SHALL add the public key representing the signer of that Evidence (for example the DICE certificate or SPDM MEASUREMENTS response) to the ECT authority field.
The Verifier SHALL add the authority of the signers of each certificate in the certificate path of the end-entity signing key to the ECT `authority` list.
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
For each `evidence-triple-record` an `ae` ECT is constructed.

{:cet-enum: counter="cet" style="format Step %d."}

{: cet-enum}
* An `ae` ECT entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The Concise Evidence (CE) entry populates the `ae` ECT `environment` fields.

> > **copy**(CE.`evidence-triple-record`.`environment-map`, ECT.`environment`.`environment-map`).

{:cet2-enum: counter="cet2" style="format %i"}

{: cet2-enum}

* For each ce in CE.`[ + measurement-map]`; and each ect in ECT.`[ + element-list]`:

> > **copy**(ce.`mkey`, ect.`element-map`.`element-id`)

> > **copy**(ce.`mval`, ect`.`element-map`.`element-claims`)

{: cet-enum}
* The signer of the envelope containing CE is copied to the ECT.`authority` field as described in {{sec-authority}.
For example, a CE may be wrapped by an EAT token {{-eat}} or DICE certificate {{-dice-attest}}.
The signer identity MUST be expressed using `$crypto-key-type-choice`.
A profile or other arrangement is used to coordinate which `$crypto-key-type-choice` is used for both Evidence and Reference Values.

* If CE has a profile, the profile is converted to a `$profile-type-choice` then copied to the ECT`.`profile` field.

The completed ECT is added to the `ae` list.

## Transforming the ce.identity-triples {#sec-identity-triple}

The `ce.identity-triples` structure is a list of `ev-identity-triple-record`.
An `ev-identity-triple-record` consists of an `environment-map` and a list of `$crypto-key-type-choice`.
For each `ev-identity-triple-record` an `ae` ECT is constructed where the `$crypto-key-type-choice` values are copied as ECT Evidence measurement values.
The ECT internal representation accommodates keys as a type of measurement.
In order for the `$crypto-key-type-choice` keys to be verified a CoRIM `identity-triples` claim MUST be asserted.

{:ikt-enum: counter="ikt" style="format Step %d."}

{: ikt-enum}
* An `ae` ECT entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The Concise Evidence (CE) entry populates the `ae` ECT `environment` fields.

> > **copy**(CE.`ce-identity-triple-record`.`environment-map`, ECT.`environment`.`environment-map`).

> > **copy**(_null_, ECT.`element-list`.`element-map`.`element-id`).

{:ikt2-enum: counter="ikt2" style="format %i"}

{: ikt2-enum}

* For each cek in CE.`[ + $crypto-key-type-choice ]`; and each ect in ECT.`element-list`.`element-map`.`element-claims`.`intrep-keys`.`[ + typed-crypto-key ]`:

> > **copy**(cek, ect.`key`)

> > **set**( &(identity-key: 1), ect.`key-type`)

{: ikt-enum}
* The signer of the envelope containing CE is copied to the ECT.`authority` field.
For example, a CE may be wrapped by an EAT token {{-eat}} or DICE certificate {{-dice-attest}}.
The signer identity MUST be expressed using `$crypto-key-type-choice`.
A profile or other arrangement is used to coordinate which `$crypto-key-type-choice` is used for both Evidence and Reference Values.

* If CE has a profile, the profile is converted to a `$profile-type-choice` then copied to the ECT`.`profile` field.

The completed ECT is added to the `ae` list.

## Transforming the ce.attest-key-triples {#sec-attest-key-triple}

The `ce.attest-key-triples` structure is a list of `ev-attest-key-triple-record`.
An `ev-attest-key-triple-record` consists of an `environment-map` and a list of `$crypto-key-type-choice`.
For each `ev-attest-key-triple-record` an `ae` ECT is constructed where the `$crypto-key-type-choice` values are copied as ECT Evidence measurement values.
The ECT internal representation accommodates keys as a type of measurement.
In order for the `$crypto-key-type-choice` keys to be verified a CoRIM `attest-key-triples` claim MUST be asserted.

{:akt-enum: counter="akt" style="format Step %d."}

{: akt-enum}
* An `ae` ECT entry is allocated.

* The `cmtype` of the ECT is set to `evidence`.

* The Concise Evidence (CE) entry populates the `ae` ECT `environment` fields.

> > **copy**(CE.`ce-attest-key-triple-record`.`environment-map`, ECT.`environment`.`environment-map`).

> > **copy**(_null_, ECT.`element-list`.`element-map`.`element-id`).

{:akt2-enum: counter="akt2" style="format %i"}

{: akt2-enum}

* For each cek in CE.`[ + $crypto-key-type-choice ]`; and each ect in ECT.`element-list`.`element-map`.`element-claims`.`intrep-keys`.`[ + typed-crypto-key ]`:

> > **copy**(cek, ect.`key`)

> > **set**( &(attest-key: 0), ect.`key-type`)

{: akt-enum}
* The signer of the envelope containing CE is copied to the ECT.`authority` field.
For example, a CE may be wrapped by an EAT token {{-eat}} or DICE certificate {{-dice-attest}}.
The signer identity MUST be expressed using `$crypto-key-type-choice`.
A profile or other arrangement is used to coordinate which `$crypto-key-type-choice` is used for both Evidence and Reference Values.

* If CE has a profile, the profile is converted to a `$profile-type-choice` then copied to the ECT`.`profile` field.

The completed ECT is added to the `ae` list.

# Transforming SPDM Evidence {#sec-spdm-trans}

This section defines how Evidence from SPDM {{-spdm}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the SPDM Evidence format SHOULD implement this transformation.

The SPDM measurements are converted to `concise-evidence` which has a format that is similar to CoRIM `triples-map` (their semantics follows the matching rules described above).
The TCG DICE Concise Evidence Binding for SPDM specification {{-ce}} describes a process for converting the SPDM Measurement Block to Concise Evidence.
Subsequently the transformation steps defined in {{sec-ce-trans}}.

The keys provided in the ECT.`authority` field SHOULD include the key which signed the SPDM MEASUREMENTS response carrying the Evidence and keys which authorized that key as described in {{sec-authority}}.```

# Implementation Status

This section records the status of known implementations of the protocol defined by this specification at the time of posting of this Internet-Draft,
and is based on a proposal described in {{RFC7942}}.
The description of implementations in this section is intended to assist the IETF in its decision processes in progressing drafts to RFCs.
Please note that the listing of any individual implementation here does not imply endorsement by the IETF.
Furthermore, no effort has been spent to verify the information presented here that was supplied by IETF contributors.
This is not intended as, and must not be construed to be, a catalogue of available implementations or their features.
Readers are advised to note that other implementations may exist.

According to {{RFC7942}}, "this will allow reviewers and working groups to assign due consideration to documents that have the benefit of running code,
which may serve as Evidence of valuable experimentation and feedback that have made the implemented protocols more mature.
It is up to the individual working groups to use this information as they see fit".

# Security and Privacy Considerations {#sec-sec}

Evidence appraisal is at the core of any RATS protocol flow, mediating all interactions between Attesters and their Relying Parties.
The Verifier is effectively part of the Attesters' and Relying Parties' trusted computing base (TCB).
Any mistake in the appraisal process could have security implications.
For instance, it could lead to the subversion of an access control function, which creates a chance for privilege escalation.

Therefore, the Verifier’s code and configuration, especially those of the CoRIM processor, are primary security assets that must be built and maintained as securely as possible.

The protection of both the Attester and Verifier systems should be considered throughout their entire lifecycle, from design to operation.
This includes the following aspects:

- Minimizing implementation complexity (see also {{Section 6.1 of -rats-endorsements}});
- Using memory-safe programming languages;
- Using secure defaults;
- Minimizing the attack surface by avoiding unnecessary features that could be exploited by attackers;
- Applying the principle of least privilege to the system's users;
- Minimizing the potential impact of security breaches by implementing separation of duties in both the software and operational architecture;
- Conducting regular, automated audits and reviews of the system, such as ensuring that users' privileges are correctly configured and that any new code has been audited and approved by independent parties;
- Failing securely in the event of errors to avoid compromising the security of the system.

The appraisal process should be auditable and reproducible.
The integrity of the code and data during execution should be made an explicit objective, for example ensuring that the appraisal functions are computed in an attestable trusted execution environment (TEE).

The integrity of public and private key material and the secrecy of private key material must be ensured at all times.
This includes key material carried in attestation key triples and key material used to assert or verify the authority of triples (such as public keys that identify trusted supply chain actors).
For more detailed information on protecting Trust Anchors, refer to {{Section 12.4 of -rats-arch}}.

The Verifier should use cryptographically protected, mutually authenticated secure channels to all its trusted input sources (i.e., Attesters, Endorsers, RVPs, Verifier Owners).
The Attester should use cryptographically protected, mutually authenticated secure channels to all its trusted input sources (i.e., Verifiers, Relying Parties).
These links must reach as deep as possible - possibly terminating within the Attesting Environment of an Attester or within the appraisal session context of a Verifier - to avoid man-in-the-middle attacks.
Also consider minimizing the use of intermediaries: each intermediary becomes another party that needs to be trusted and therefore factored in the Attesters and Relying Parties' TCBs.
Refer to {{Section 12.2 of -rats-arch}} for information on Conceptual Messages protection.

# IANA Considerations {#sec-iana-cons}

There are no IANA considerations.

--- back


# Contributors
{:unnumbered}

The authors would like to thank the following people for their valuable contributions to the specification.

Henk Birkholz

Email: henk.birkholz@ietf.contact

Fabrizio D'Amato

Email: fabrizio.damato@amd.com

Yogesh Deshpande

Email: yogesh.deshpande@arm.com

Thomas Fossati

Email: Thomas.Fossati@linaro.org

Dionna Glaze

Email: dionnaglaze@google.com

# Acknowledgments
{:unnumbered}

The authors would like to thank James D. Beaney, Francisco J. Chinchilla, Vincent R. Scarlata, and Piotr Zmijewski for review feedback.

