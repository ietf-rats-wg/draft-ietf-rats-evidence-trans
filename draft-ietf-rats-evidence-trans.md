---
v: 3

title: Evidence Transformations
abbrev: EvTrans
docname: draft-ietf-rats-evidence-trans-latest
category: std
consensus: true
submissiontype: IETF
date: 2026-10-30
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
- ins: F. D'Amato
  name: Fabrizio D'Amato
  org: AMD
  email: fabrizio.damato@amd.com
- ins: A. Draper
  name: Andrew Draper
  org: Independent
  email: andrew.draper@freeside.org.uk
- ins: N. Smith
  name: Ned Smith
  org: Independent
  email: ned.smith.ietf@outlook.com

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
  RFC9711: eat
  RFC5280: x509
  RFC7942: running-code

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

# Requirements Language and Terminology

{::boilerplate bcp14}

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

The transformations are specified in pseudocode using the notation defined in {{Section 8.1 of -corim}}.
The pseudocode is meant for readability and not for execution.
Type names are taken from the CDDL of CoRIM and Concise Evidence; the names of structures defined by {{-dice-attest}} (e.g., `DiceTcbInfo`, `FWID`) and {{-spdm}} (e.g., `MeasurementBlock`) are used as bare type names.

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

## DiceUeid Transformation {#sec-ueid}

This section defines the transformation method for the DiceUeid certificate extension.
This extension is identified by the following object identifier:

* tcg-dice-Ueid - "2.23.133.5.4.4"

The DiceUeid extension does not create evidence directly.
The `ueid` OCTET STRING within this extension is used to populate the `instance-id` field within evidence created by other extensions.
Section {{sec-tcb-info}} describes extensions which use this value.

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

Each DiceTcbInfo (DTI) entry in a DiceMultiTcbInfo is transformed as described by the pseudocode in {{algo-dti-transform}}.
The pseudocode uses the notation defined in {{Section 8.1 of -corim}}.
The `ueid` parameter is the `ueid` OCTET STRING of the DiceUeid extension, if present.
The `signer` parameter is set up as described in {{sec-authority}}.

~~~ pseudocode
FUNC fwids_to_digests(fwids: [ + FWID ]) -> digests-type {
    digests := digests-type::NEW()

    FOREACH fwid IN fwids:
        d := eatmc.digest::NEW()
        d.alg = fwid.hashAlg
        d.val = fwid.digest
        digests::APPEND(d)

    RETURN digests
}

FUNC dti_flags_to_flags_map(
    f: OperationalFlags,
    m: OperationalFlagsMask
) -> flags-map {
    fm := flags-map::NEW()

    IF m.notConfigured == 1:
        fm.is-configured = (f.notConfigured == 0)
    IF m.notSecure == 1:
        fm.is-secure = (f.notSecure == 0)
    IF m.recovery == 1:
        fm.is-recovery = (f.recovery == 0)
    IF m.debug == 1:
        fm.is-debug = (f.debug == 0)
    IF m.notReplayProtected == 1:
        fm.is-replay-protected = (f.notReplayProtected == 0)
    IF m.notIntegrityProtected == 1:
        fm.is-integrity-protected = (f.notIntegrityProtected == 0)
    IF m.notRuntimeMeasured == 1:
        fm.is-runtime-meas = (f.notRuntimeMeasured == 0)
    IF m.notImmutable == 1:
        fm.is-immutable = (f.notImmutable == 0)
    IF m.notTcb == 1:
        fm.is-tcb = (f.notTcb == 0)

    RETURN fm
}

FUNC dti_to_ae_item(
    dti: DiceTcbInfo,
    ueid: ueid-type,
    signer: [ + $crypto-key-type-choice ]
) -> ae-item {
    item := ae-item::NEW()

    item.addition.cmtype = evidence

    // environment
    IF dti.type:
        item.addition.environment.class.class-id = dti.type
    IF dti.vendor:
        item.addition.environment.class.vendor = dti.vendor
    IF dti.model:
        item.addition.environment.class.model = dti.model
    IF dti.layer:
        item.addition.environment.class.layer = dti.layer
    IF dti.index:
        item.addition.environment.class.index = dti.index
    IF ueid:
        item.addition.environment.instance = #6.550(ueid)

    // element-list
    em := element-map::NEW()

    IF dti.version:
        em.element-claims.version.version = dti.version
    IF dti.svn:
        em.element-claims.svn = dti.svn
    IF dti.vendorInfo:
        em.element-claims.raw-value = #6.560(dti.vendorInfo)
    IF dti.FWIDLIST:
        em.element-claims.digests = fwids_to_digests(dti.FWIDLIST)
    FOREACH ir IN dti.IRLIST:
        em.element-claims.integrity-registers[ir.registerNum] =
            fwids_to_digests(ir.FWIDLIST)
    IF dti.OperationalFlags:
        em.element-claims.flags =
            dti_flags_to_flags_map(dti.OperationalFlags,
                                   dti.OperationalFlagsMask)

    item.addition.element-list::APPEND(em)

    item.addition.authority = signer

    RETURN item
}

FUNC dice_to_ae(
    dtis: [ + DiceTcbInfo ],
    ueid: ueid-type,
    signer: [ + $crypto-key-type-choice ]
) -> ae {
    result := ae::NEW()

    FOREACH dti IN dtis:
        result::APPEND(dti_to_ae_item(dti, ueid, signer))

    RETURN result
}
~~~
{: #algo-dti-transform title="DiceTcbInfo Transformation"}

The binary representation of DTI.`type` MUST be equivalent to the binary representation of `class-id` without the CBOR tag.

The CBOR tag #6.550 is prepended to the DiceUeid OCTET STRING before it is copied to the `instance` field of the `environment-map`.

The DTI.`vendorInfo` OCTET STRING is wrapped in the CBOR tag #6.560 (`tagged-bytes`) as required by `$raw-value-type-choice`.

<cref>The flag-by-flag mapping above follows the previous text of this section.
For `recovery` and `debug`, DiceTcbInfo does not use the "not" prefix, so the polarity of `is-recovery` and `is-debug` looks inverted.
Confirm against {{-dice-attest}}.</cref>

## DiceConceptualMessageWrapper Transformation {#sec-cmw}

This section defines the transformation method for the DiceConceptualMessageWrapper certificate extension.
This extension is identified by the following object identifier:

* tcg-dice-Ueid - "2.23.133.5.4.9"

The DiceConceptualMessageWrapper entry OCTET STRING may contain a CBOR array, JSON array, or CBOR tagged value.
If the entry contains a CBOR tag value of #6.571 or #6.1668557429, or a Content ID of 10571, or a Media Type of "application/ce+cbor",
the contents are transformed according to {{sec-ce-trans}}.

~~~ pseudocode
FUNC dice_cmw_to_ae(
    cmw: DiceConceptualMessageWrapper,
    signer: [ + $crypto-key-type-choice ]
) -> ae {
    IF TYPEOF(cmw) == tagged-concise-evidence:
        RETURN transform_ce(~cmw, signer)
    ELIF TYPEOF(cmw) == concise-evidence-map:
        // identified by Content ID or Media Type
        RETURN transform_ce(cmw, signer)

    // other content is not transformed
    RETURN ae::NEW()
}
~~~
{: #algo-dice-cmw-transform title="DiceConceptualMessageWrapper Transformation"}

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

The pseudocode in {{algo-ce-transform}} transforms a Concise Evidence (CE) entry.
The `signer` parameter is the signer of the envelope containing the CE, set up as described in {{sec-authority}}.
For example, a CE may be wrapped by an EAT token {{-eat}} or DICE certificate {{-dice-attest}}.
The signer identity MUST be expressed using `$crypto-key-type-choice`.
A profile or other arrangement is used to coordinate which `$crypto-key-type-choice` is used for both Evidence and Reference Values.
If the CE has a profile, the profile is converted to a `$profile-type-choice` and copied to the `profile` field of every resulting ECT.
(The pseudocode reuses the `mms_to_ems` function defined in {{Section 8.3.2.1.1 of -corim}}.)

~~~ pseudocode
FUNC transform_ce(
    ce: concise-evidence-map,
    signer: [ + $crypto-key-type-choice ]
) -> ae {
    result := ae::NEW()
    profile := ce.profile

    FOREACH T IN ce.evidence-triples:
        result::APPEND(transform_ce_evidence(T, signer, profile))

    FOREACH T IN ce.identity-triples:
        result::APPEND(transform_ce_keys(T, signer, profile))

    FOREACH T IN ce.attest-key-triples:
        result::APPEND(transform_ce_keys(T, signer, profile))

    RETURN result
}
~~~
{: #algo-ce-transform title="Concise Evidence Transformation"}

## Transforming the ce.evidence-triples {#sec-evidence-triple}

The `ce.evidence-triples` structure is a list of `evidence-triple-record`.
An `evidence-triple-record` consists of an `environment-map` (`environment`) and a list of `measurement-map` (`measurements`).
For each `evidence-triple-record` an `ae` ECT is constructed.

~~~ pseudocode
FUNC transform_ce_evidence(
    T: evidence-triple-record,
    signer: [ + $crypto-key-type-choice ],
    profile: $profile-type-choice
) -> ae-item {
    item := ae-item::NEW()

    item.addition.cmtype = evidence
    item.addition.environment = T.environment
    item.addition.element-list = mms_to_ems(T.measurements)
    item.addition.authority = signer

    IF profile:
        item.addition.profile = profile

    RETURN item
}
~~~
{: #algo-ce-evidence-transform title="CE Evidence Triple Transformation"}

## Transforming the ce.identity-triples {#sec-identity-triple}

The `ce.identity-triples` structure is a list of `ev-identity-triple-record`.
An `ev-identity-triple-record` consists of an `environment-map` (`environment`) and a list of `$crypto-key-type-choice` (`key-list`).
For each `ev-identity-triple-record` an `ae` ECT is constructed where the `$crypto-key-type-choice` values are copied as ECT Evidence measurement values.
The ECT internal representation accommodates keys as a type of measurement.
In order for the `$crypto-key-type-choice` keys to be verified a CoRIM `identity-triples` claim MUST be asserted.

~~~ pseudocode
FUNC transform_ce_keys(
    T: ev-identity-triple-record / ev-attest-key-triple-record,
    signer: [ + $crypto-key-type-choice ],
    profile: $profile-type-choice
) -> ae-item {
    item := ae-item::NEW()

    item.addition.cmtype = evidence
    item.addition.environment = T.environment

    em := element-map::NEW()
    em.element-claims.cryptokeys = T.key-list
    item.addition.element-list::APPEND(em)

    item.addition.authority = signer

    IF profile:
        item.addition.profile = profile

    RETURN item
}
~~~
{: #algo-ce-keys-transform title="CE Identity and Attest Key Triples Transformation"}

<cref>The previous text set a `key-type` (`identity-key` or `attest-key`) next to the keys.
`E-ECT` and `measurement-values-map` in {{-corim}} have no such member: `key-type` belongs to the `keys` relation, which is staged from the CoRIM triple.
As a result, the two kinds of CE triple are transformed identically.
Confirm that this is the intent.</cref>

## Transforming the ce.attest-key-triples {#sec-attest-key-triple}

The `ce.attest-key-triples` structure is a list of `ev-attest-key-triple-record`.
An `ev-attest-key-triple-record` consists of an `environment-map` (`environment`) and a list of `$crypto-key-type-choice` (`key-list`).
For each `ev-attest-key-triple-record` an `ae` ECT is constructed where the `$crypto-key-type-choice` values are copied as ECT Evidence measurement values.
The ECT internal representation accommodates keys as a type of measurement.
In order for the `$crypto-key-type-choice` keys to be verified a CoRIM `attest-key-triples` claim MUST be asserted.

The transformation is the same as for `ce.identity-triples`, i.e., {{algo-ce-keys-transform}}.

# DMTF SPDM Structure Definitons

This section defines how a Verifier shall parse a DMTF Measurement Block.

DMTF Measurement Block Definition:

- Byte Offset 0: DMTFSpecMeasurementValueType
  - Bit 7     = 0b Digest / 1b Raw bit stream
  - Bit \[6:0\] = Indicate what is measured
    - 0x0 Immutable Rom
    - 0x1 Mutable FW
    - 0x2 HW Config
    - 0x3 FW config
    - 0x4 Freeform Manifest
    - 0x5 Structured Representation of debug and device mode
    - 0x6 Mutable FW Version Number
    - 0x7 Mutable FW Secure Version Number
    - 0x8 Hash-Extend Measurement (new in SPDM 1.3)
    - 0x9 Informational (new in SPDM 1.3)
    - 0xA Structured Measurement Manifest (new in SPDM 1.3)
- Byte Offset 1: DMTFSpecMeasurementValueSize
- Byte Offset 3: DMTFSpecMeasurementValue

Structured Manifest Block Definition (only for >=SPDM 1.3):

- Byte Offset 0: Standard Body or Vendor Defined Header (SVH)
- Byte Offset 2 + VendorIdLen: Manifest

Standard Body or Vendor Defined Header (SVH) Definition (only for >=SPDM 1.3):

- Byte Offset 0: ID
- Byte Offset 1: VendorIdLen
- Byte Offset 2: VendorId


DMTF Header for Concise Evidence Manifest Block:

If SPDM Version 1.2:

- DMTFSpecMeasurementValueType = 0x84 (Raw Bit / Freeform Manifest)
- DMTFSpecMeasurementValueSize = Size of tagged-spdm-toc CBOR Tag
- DMTFSpecMeasurementValue     = tagged-spdm-toc CBOR Tag

if SPDM >=Version 1.3:

- DMTFSpecMeasurementValueType = 0x8A (Raw Bit / Structured Manifest)
- DMTFSpecMeasurementValueSize = Size of Structured Manifest
- DMTFSpecMeasurementValue     = Structured Manifest

SVH for Concise Evidence Manifest Block:

- ID          = 0xA (IANA CBOR)
- VendorIdLen = 3
- VendorId    = (0xd9 0x02 0x3a) #6.570(spdm-toc-map)

Structured Manifest Block Definition for Concise Evidence:

- SVH      = SVH for Concise Evidence Manifest Block
- Manifest = tagged-spdm-toc CBOR Tag Payload

DMTF Header for signed CBOR Web Token (CWT):

If SPDM Version 1.2:

- DMTFSpecMeasurementValueType = 0x84 (Raw Bit / Freeform Manifest)
- DMTFSpecMeasurementValueSize = Size of CWT
- DMTFSpecMeasurementValue     = CWT # COSE_Sign1

if SPDM = Version 1.3:

- DMTFSpecMeasurementValueType = 0x8A (Raw Bit / Structured Manifest)
- DMTFSpecMeasurementValueSize = Size of Structured Manifest
- DMTFSpecMeasurementValue     = Structured Manifest

SVH for signed CBOR Web Token (CWT):

- ID          = 0xA (IANA CBOR)
- VendorIdLen = 1
- VendorId    = 0xd2 #6.18(COSE_Sign1)

Structured Manifest Block Definition for CBOR Web Token (CWT):

- SVH      = SVH for CBOR Web Token (CWT)
- Manifest = COSE_Sign1 Payload

# Transforming SPDM Measurement Block Digest

~~~ pseudocode
FUNC spdm_digest_to_claims(
    mb: MeasurementBlock,
    claims: measurement-values-map
) -> measurement-values-map {
    t := mb.DMTFSpecMeasurementValueType

    IF t >= 0x80 AND t <= 0x83:
        claims.digests = mb.DMTFSpecMeasurementValue
    ELIF t == 0x88:
        claims.integrity-registers = mb.DMTFSpecMeasurementValue

    RETURN claims
}
~~~
{: #algo-spdm-digest title="SPDM Measurement Block Digest Transformation"}

# Transforming SPDM Measurement Block Raw Value

~~~ pseudocode
FUNC spdm_raw_to_claims(
    mb: MeasurementBlock,
    claims: measurement-values-map
) -> measurement-values-map {
    t := mb.DMTFSpecMeasurementValueType

    IF t == 0x07:
        claims.svn = mb.DMTFSpecMeasurementValue

    RETURN claims
}
~~~
{: #algo-spdm-raw title="SPDM Measurement Block Raw Value Transformation"}

<cref>Per the Measurement Block definition above, bit 7 of DMTFSpecMeasurementValueType is 0 for a digest and 1 for a raw bit stream.
The ranges used in {{algo-spdm-digest}} and {{algo-spdm-raw}} are those of the previous text, and appear to have bit 7 swapped.
Also, `digests` requires a hash algorithm identifier, which the Measurement Block does not carry.</cref>

# Transforming EAT Conveyed by SPDM

The EAT shall be reported in Measurement Block 0xFD.
The Concise Evidence CBOR Tag is serialized inside eat-measurements (273) claim ($measurements-body-cbor /= bytes .cbor concise-evidence-map)
Subsequently the transformation steps defined in {{sec-ce-trans}}.

~~~ pseudocode
FUNC spdm_eat_to_ae(
    mb: MeasurementBlock,
    signer: [ + $crypto-key-type-choice ]
) -> ae {
    eat := mb.DMTFSpecMeasurementValue
    // $measurements-body-cbor /= bytes .cbor concise-evidence-map
    ce := eat.eat-measurements

    RETURN transform_ce(ce, signer)
}
~~~
{: #algo-spdm-eat title="EAT Conveyed by SPDM Transformation"}

# Transforming SPDM Evidence {#sec-spdm-trans}

This section defines how Evidence from SPDM {{-spdm}} is transformed into an internal representation that can be processed by Verifiers.

Verifiers supporting the SPDM Evidence format SHOULD implement this transformation.
SPDM Responders SHALL support a minimum version of 1.2

The SPDM Requestor SHALL retrieve the measurement Manifest at Block 0xFD (Manifest Block) and send its payload to the Verifier.
The Verifier SHALL decode the payload as a tagged-spdm-toc CBOR tag and extract the tagged-concise-evidence CBOR tag from it.

The `concise-evidence` has a format that is similar to CoRIM `triples-map` (their semantics follows the matching rules described above).

For every `spdm-indirect` measurement the Verifier shall ask the SPDM Requestor to retrieve the measurement block indicated by the index.

~~~ pseudocode
FUNC transform_spdm(
    mb-fd: MeasurementBlock,
    requestor: SPDM-Requestor,
    signer: [ + $crypto-key-type-choice ]
) -> ae {
    // mb-fd is Measurement Block 0xFD (Manifest Block)
    toc := mb-fd.DMTFSpecMeasurementValue    // tagged-spdm-toc
    ce := ~toc.tagged-concise-evidence

    result := transform_ce(ce, signer)
    eats := ae::NEW()

    FOREACH item IN result:
        FOREACH em IN item.addition.element-list:
            IF em.element-claims.spdm-indirect:
                index := em.element-claims.spdm-indirect
                mb := requestor::GET_MEASUREMENT_BLOCK(index)

                IF index >= 0x01 AND index <= 0xEF:
                    em.element-claims =
                        spdm_digest_to_claims(mb, em.element-claims)
                    em.element-claims =
                        spdm_raw_to_claims(mb, em.element-claims)
                ELIF index == 0xFD:
                    eats::APPEND(spdm_eat_to_ae(mb, signer))

    result::APPEND(eats)

    RETURN result
}
~~~
{: #algo-spdm-transform title="SPDM Evidence Transformation"}

The `signer` parameter is set up as described in {{sec-authority}}: its keys SHOULD include the key which signed the SPDM MEASUREMENTS response carrying the Evidence and keys which authorized that key.

The TCG DICE Concise Evidence Binding for SPDM specification {{-ce}} describes a process for converting the SPDM Measurement Block to Concise Evidence.
Subsequently the transformation steps defined in {{sec-ce-trans}}.

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

# Security Considerations {#sec-sec}

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

Yogesh Deshpande

Email: yogesh.deshpande@arm.com

Thomas Fossati

Email: Thomas.Fossati@linaro.org

Dionna Glaze

Email: dionnaglaze@google.com

# Acknowledgments
{:unnumbered}

The authors would like to thank James D. Beaney, Francisco J. Chinchilla, Vincent R. Scarlata, and Piotr Zmijewski for review feedback.

