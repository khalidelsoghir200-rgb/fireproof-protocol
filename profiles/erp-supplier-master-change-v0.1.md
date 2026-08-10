# FireProof Receiver Acceptance Profile — ERP Supplier Master Change v0.1

Status: R2 normative contract before implementation.

## 1. Scope

This profile governs one synthetic, owner-controlled operation:

    operation_ref = supplier.payment-destination.replace

It models a sensitive ERP supplier-master change without contacting a bank,
payment rail, customer tenant, or external ERP. The profile proves only the
signed, linked claim that the R2 receiver accepted or rejected a synthetic
local state transition. It does not prove a real supplier, bank account,
payment, legal outcome, or external-world effect.

## 2. Versioning and R1 compatibility

R2 is additive to R1:

- R1 bundle format fireproof.evidence-bundle/v0.1 remains unchanged.
- R1 receipt profile fireproof.action-receipt/v0.2 remains unchanged.
- R2 bundle format is fireproof.evidence-bundle/v0.2.
- R2 profile identifier is
  fireproof.receiver-acceptance.erp-supplier-master/v0.1.
- R2 Authority payload type is fireproof.receiver-authority/v0.1.
- R2 Fulfillment payload type is fireproof.receiver-fulfillment/v0.1.
- R2 receiver manifest payload type is
  fireproof.receiver-trust-manifest/v0.1.
- R2 uses standard DSSE v1 PAE, RFC 8785 JCS, and Ed25519. It introduces no
  COSE, custom PAE, network verifier, or signing-algorithm change.

R2 defines a separately buildable verifier and schema family. The R1
fireproof-verify CLI and package remain R1-only. They MUST return unsupported
for an R2 bundle and MUST NOT treat it as valid by reinterpreting it under R1.

No R1 schema is widened with optional R2 fields. No R2 semantic field is
smuggled into an R1 result digest, policy reference, or HTTP/PATCH profile.

## 3. Canonical bytes and digest notation

Every SHA-256 value in this profile is lower-case hexadecimal encoding of the
32-byte digest. The operator || means byte concatenation. Literal domain tags
are UTF-8 bytes including the terminating zero byte. JCS(object) means the
UTF-8 bytes of RFC 8785 canonical JSON. No parser may substitute a
reserialized or semantically similar object where the exact byte rule below is
required.

For a verified DSSE envelope, payload_sha256 means SHA-256 of the decoded raw
payload bytes exactly as verified by DSSE. It is not SHA-256 of a later JSON
reserialization.

## 4. Receiver-owned trust model

The ERP receiver owns a local root file and a local receiver configuration
record. Neither is received from the sender, inferred from DSSE keyid, fetched
over a network, or controlled by FireProof.

The local configuration contains exactly one active receiver-manifest payload
digest and version for this profile, receiver, tenant, and environment. A
portable receiver manifest is admissible for a new write only when all of the
following hold:

1. its DSSE signature verifies under the local root;
2. its payload type and scope match this profile;
3. its verified raw payload digest equals the locally pinned active manifest
   digest; and
4. the Authority and Fulfillment signer relations match that manifest.

A root signature alone is insufficient. If the receiver supersedes or revokes a
manifest locally, an older portable manifest cannot authorize a new write even
if it remains correctly signed. Offline verification may validate a historical
bundle against the root and manifest supplied for that bundle; it must not
mistake that historical validity for permission to create a new receiver write.

This v0.1 profile has no online revocation, distributed manifest expiry, or
root/key transparency-log semantics. Rotation is configuration-controlled: the
receiver changes its locally pinned active manifest digest. A signer is trusted
for new acceptance only because its exact key identifier and role are listed in
that active manifest. The deterministic expired-proof case in R2 applies to
the Authority validity window, not to an invented manifest/key expiry rule.

The receiver manifest MUST admit:

- Authority issuer keys for the exact profile, receiver, tenant, environment,
  operation, and permitted field; and
- receiver Fulfillment keys for the same relation.

The Authority signer, receiver Fulfillment signer, and root signer are
distinct roles. The Authority signer cannot issue a receiver success claim.

Before the receiver begins a mutation transaction, it MUST select one admitted
receiver Fulfillment signer from the pinned manifest, verify the private key is
available, and pin that signer key identifier in the immutable fulfillment
basis. Recovery MUST use that exact key and payload or remain in bounded
recovery-pending state; it MUST NOT substitute a rotated or different signer.

### 4.1 Root, receiver manifest, and signer shapes

The explicit verifier root is a strict JSON object:

    {
      "format": "fireproof.trust-root-set/v0.1",
      "keys": [
        {
          "jwk_thumbprint_sha256": "<base64url JWK thumbprint>",
          "public_jwk": {"kty": "OKP", "crv": "Ed25519", "x": "<base64url 32 bytes>"}
        }
      ]
    }

It contains one to sixteen distinct root keys and no other fields. Every
Ed25519 JWK has exactly kty, crv, and x. Its keyid is the RFC 7638 thumbprint
over the JCS-compatible object containing crv, kty, and x, encoded as
unpadded base64url SHA-256. The declared thumbprint MUST equal that value.

The root-signed receiver manifest payload is a strict object with these exact
fields:

| Field | Requirement |
| --- | --- |
| format | fireproof.receiver-trust-manifest/v0.1 |
| profile | Exact R2 profile identifier |
| kind | receiver_trust_manifest |
| receiver_ref | Exact synthetic receiver identity |
| tenant_ref | Exact tenant scope |
| environment_ref | Exact sandbox environment scope |
| manifest_version | Positive integer selected by receiver configuration |
| authority_issuers | One to sixteen authority issuer entries |
| fulfillment_signers | One to sixteen receiver signer entries |

Each authority_issuers entry is a strict object containing issuer_id, keyid,
public_jwk, operation_ref, and permitted_field. It admits exactly the R2
operation and payment_destination field. Each fulfillment_signers entry is a
strict object containing signer_id, keyid, and public_jwk. In either entry,
keyid MUST equal the JWK thumbprint of public_jwk. Duplicate keyid or logical
identifier values are invalid. No signer status, network revocation URL, or
unstated lifetime field is permitted in v0.1; local active-manifest selection
is the rotation mechanism.

### 4.2 DSSE envelope shape

Every manifest, Authority, and Fulfillment artifact is exactly one DSSE v1
envelope:

    {
      "payloadType": "<role-specific media type>",
      "payload": "<canonical standard base64 of JCS payload bytes>",
      "signatures": [{"keyid": "<JWK thumbprint>", "sig": "<canonical standard base64 Ed25519 signature>"}]
    }

The envelope has no extension fields and exactly one signature. payload and sig
use canonical padded standard base64, not base64url. The decoded payload bytes
MUST equal their RFC 8785 JCS serialization. The signature is Ed25519 over the
standard DSSE v1 PAE bytes.

The exact payloadType by role is:

| Role | payloadType |
| --- | --- |
| manifest | application/vnd.fireproof.receiver-trust-manifest.v0+json |
| authority | application/vnd.fireproof.receiver-authority.v0+json |
| fulfillment | application/vnd.fireproof.receiver-fulfillment.v0+json |

## 5. Authority artifact

A sender presents a decision bundle containing a root-signed receiver manifest
and an Authority signed by an issuer admitted by the locally pinned manifest.
It contains no Fulfillment.

The Authority payload MUST bind all fields below and reject unknown fields
unless a later profile version says otherwise.

| Field | Requirement |
| --- | --- |
| format | fireproof.receiver-authority/v0.1 |
| profile | Exact R2 profile identifier |
| receiver_ref | Exact synthetic ERP receiver identity |
| tenant_ref | Exact local tenant scope |
| environment_ref | Exact sandbox environment scope |
| kind | Exact string authority |
| authority_issuer_id | Exact issuer_id admitted by the manifest entry matching DSSE keyid |
| actor_ref_commitment | Commitment to the Authority-bound claimed actor reference; not an authenticated receiver caller identity in v0.1 |
| operation_ref | Exact operation in section 1 |
| supplier_ref_commitment | Commitment to the requested supplier reference |
| permitted_field | Exact string payment_destination |
| payment_destination_commitment | Commitment to the proposed replacement value |
| expected_record_version | Positive integer expected by the sender |
| expected_state_commitment | Commitment to the expected receiver state |
| action_instance_id | Single-use 1-128 character ASCII identifier matching `[A-Za-z0-9][A-Za-z0-9._:-]*` |
| use_nonce_commitment | Commitment to one 32-byte use nonce |
| receiver_manifest_payload_sha256 | Digest of the admitted receiver manifest |
| issued_at | RFC 3339 UTC time |
| expires_at | RFC 3339 UTC time later than issued_at |
| decision | One of allow, deny, approval_required |

The Authority DSSE keyid MUST equal the keyid of exactly one matching
authority_issuers entry, and authority_issuer_id MUST equal that entry's
issuer_id. No unknown Authority fields are permitted.

Only decision = allow can reach the receiver write transaction. A valid deny or
approval_required artifact causes no mutation and consumes neither action nor
nonce. It is an upstream decision, not receiver acceptance.

R2 v0.1 is a bearer-proof sandbox. Possession of a valid bundle, matching
private request, and one-use nonce is sufficient to present a mutation
candidate. The receiver validates that actor_ref matches the Authority-bound
claim, but it does not authenticate a live caller principal. Therefore a
Fulfillment never claims that a particular network caller was authenticated;
it claims only the Authority-bound actor reference. Receiver-authenticated
caller identity is a future profile decision, not an implicit property.

## 6. Boundary request and commitments

The sender supplies private request material only over the local demonstrator
boundary. It contains synthetic values:

    {
      "actor_ref": "actor:synthetic-001",
      "supplier_ref": "supplier:synthetic-001",
      "expected_record_version": 1,
      "payment_destination": "synthetic-payment-destination-v2",
      "use_nonce_base64url": "<32-byte unpadded base64url nonce>"
    }

The receiver MUST strictly parse this object, reject extra fields, decode an
exactly 32-byte unpadded base64url nonce, and recompute every Authority-bound
commitment before opening a transaction. It MUST also require the direct
boundary_request.expected_record_version integer to equal the Authority
expected_record_version before opening a transaction.

The private request is UTF-8 without a byte-order mark, at most 4 KiB, and
uses a duplicate-name-rejecting JSON parser. actor_ref and supplier_ref are
1-128 character ASCII strings matching [A-Za-z0-9][A-Za-z0-9._:-]*.
payment_destination is a 1-128 character synthetic ASCII string matching the
same grammar. JSON floating point values, numeric constants, nested objects,
and arrays are not admitted.

The v0.1 receiver is configured for exactly one synthetic supplier reference.
If the private supplier_ref is not that configured reference, the receiver
returns target_not_admitted before opening a transaction and consumes neither
action nor nonce. It does not create a record_absent terminal outcome.

The formulas are:

    actor_ref_commitment =
      SHA-256("FIREPROOF-R2-ACTOR-REF-V1\0" ||
              JCS({"actor_ref": value}))

    supplier_ref_commitment =
      SHA-256("FIREPROOF-R2-SUPPLIER-REF-V1\0" ||
              JCS({"supplier_ref": value}))

    payment_destination_commitment =
      SHA-256("FIREPROOF-R2-PAYMENT-DESTINATION-V1\0" ||
              JCS({"payment_destination": value}))

    use_nonce_commitment =
      SHA-256("FIREPROOF-R2-USE-NONCE-V1\0" || raw_32_byte_nonce)

The relevant supplier-state commitment is:

    supplier_state_commitment =
      SHA-256("FIREPROOF-R2-SUPPLIER-STATE-V1\0" ||
              JCS({
                "profile": "fireproof.receiver-acceptance.erp-supplier-master/v0.1",
                "receiver_ref": receiver_ref,
                "tenant_ref": tenant_ref,
                "supplier_ref_commitment": supplier_ref_commitment,
                "record_version": record_version,
                "payment_destination_commitment": payment_destination_commitment
              }))

All properties in the JCS object are mandatory. Values are represented exactly
as their JSON types above; record_version is an integer, and every commitment
is its lower-case hexadecimal string. The implementation MUST publish safe
vectors for each formula before making an interoperability claim.

Raw request values MUST NOT enter an evidence bundle, Fulfillment payload,
console report, or exception message. They also MUST NOT enter a public
fixture, except the fixed synthetic R2 commitment-formula vectors. That narrow
exception is not
portable evidence and exists only so an independent implementation can
reproduce the specified hashes. It MUST contain no real, production, or
customer-derived value and MUST NOT be copied into a production Authority,
request, state store, log, or bundle.

## 7. Time rule

The receiver records acceptance time and a pre-commit marker using its local
UTC clock. That clock is a receiver-attested observation, not a trusted global
clock.

Every R2 timestamp is the canonical whole-second UTC RFC 3339 form
`YYYY-MM-DDTHH:MM:SSZ`. Fractional seconds, offsets other than `Z`, and
semantically equivalent alternate renderings are not admitted.

For every durable receiver terminal basis, including a state_changed rejection,
the receiver MUST establish:

    issued_at <= accepted_at <= commit_prepared_at <= expires_at

inside the documented transaction path. commit_prepared_at is a
receiver-attested marker captured under the transaction lock immediately before
the commit attempt; it is not a claim to measure the physical storage-engine
commit instant. The canonical Fulfillment payload stores accepted_at and
commit_prepared_at before commit. Its DSSE signature may be created after a
successful commit from those immutable persisted bytes, even if the act of
signing occurs after expires_at. This does not extend Authority validity.

## 8. Action identity, replay, and transaction

A terminal action key is:

    terminal_action_key =
      SHA-256("FIREPROOF-R2-TERMINAL-ACTION-V1\0" ||
              JCS({
                "receiver_ref": receiver_ref,
                "tenant_ref": tenant_ref,
                "action_instance_id": action_instance_id
              }))

The receiver database MUST enforce both unique scopes:

    (receiver_ref, tenant_ref, action_instance_id)
    (receiver_ref, tenant_ref, use_nonce_commitment)

A nonce or action_instance_id is a receiver resource, not a retry hint. The
single-use scope of action_instance_id is the whole receiver/tenant namespace,
not one Authority payload. A retry or a later Authority carrying the same
action_instance_id does not create a second terminal Fulfillment.

After all cryptographic, local-manifest, scope, decision, private-request, and
time preconditions pass, the receiver opens the documented SQLite transaction
with BEGIN IMMEDIATE. Within that same transaction it MUST:

1. check both durable uniqueness constraints;
2. read the synthetic supplier row;
3. compute and compare the actual supplier_state_commitment to the Authority
   expected_state_commitment and compare the exact expected record version;
4. record accepted_at and require it to be in the Authority validity interval;
5. for a matching state, conditionally update exactly one row using the
   expected version in the update predicate;
6. compute before and after state commitments from the receiver's own state;
7. create the immutable canonical Fulfillment payload bytes, including the
   pinned fulfillment_signer_keyid, and durable action journal basis; and
8. record commit_prepared_at, require it to be within the Authority validity
   interval, then attempt commit.

For a matching state, the conditional update MUST change exactly one supplier
row and increment its version exactly once. A zero-row update is a stale-state
result, never a success.

For a succeeded Fulfillment, the following semantic relations are mandatory:

    Authority.expected_state_commitment == Fulfillment.before_state_commitment
    Authority.expected_record_version == Fulfillment.record_version_before
    Fulfillment.record_version_after == Fulfillment.record_version_before + 1
    Fulfillment.payment_destination_commitment ==
        Authority.payment_destination_commitment
    Fulfillment.after_state_commitment != Fulfillment.before_state_commitment

The after-state commitment is the section 6 formula using the same receiver,
tenant, and supplier commitment, the incremented record version, and the
Authority-bound payment-destination commitment. The receiver recomputes this
private preimage inside its transaction; the offline verifier checks the
required signed field equalities and shape, not an absent raw preimage.

For a valid allow Authority whose expected state does not match, the receiver
MUST consume the terminal action and nonce, persist one rejected Fulfillment
basis with reason_code = state_changed, and commit without changing the
supplier. That rejection attests only that the receiver observed the mismatched
state inside its durable acceptance transaction and made no supplier update in
that transaction. It does not claim the record can never change elsewhere.

If the expected state matches but the current private payment destination
already has the Authority payment_destination_commitment, the receiver MUST
consume the terminal action and nonce, persist one rejected Fulfillment basis
with reason_code = no_effect, and commit without changing the supplier or its
record version. This is not a succeeded write and it must not manufacture a
version increment merely to create a receipt.

Malformed, tampered, untrusted, expired, wrong-scope, non-allow, or
request-mismatched input is rejected before a durable action record is made and
does not consume an action or nonce.

The fulfillment basis has internal state committed_unattested after transaction
commit. Only after commit may the receiver sign the exact persisted canonical
payload and transition the journal to its published terminal result. A process
crash after commit but before signing is recovered by signing that immutable
basis; it MUST NOT apply a second supplier update or generate a second
Fulfillment payload. This v0.1 local SQLite profile defines no ordinary
indeterminate terminal result. A future irrecoverable durable-state condition
requires a new profile version and an explicit fault model; it may not be
invented from caller timeout or reply loss.

If the local clock moves backward such that commit_prepared_at is earlier than
accepted_at, the receiver MUST roll back before attempting a mutation commit
and return a non-terminal clock_error. If SQLite COMMIT raises an error, the
receiver MUST close the connection and reopen its durable store before reporting
an outcome. A readable durable matching basis means the transaction committed
and recovery may publish that exact basis. No matching basis means a safe
non-terminal storage_error. If durable state cannot be read reliably, the
receiver fails closed with a non-terminal recovery_error. It MUST NOT retry the
supplier mutation automatically or emit a succeeded Fulfillment in any of
these paths.

On replay, the receiver returns the already published terminal Fulfillment. If
recovery is still pending, it returns a bounded pending/recovery response until
the one durable basis is signed. It never writes a second terminal outcome.
Authority expiry applies to first acceptance only; it does not erase or prevent
safe retrieval of a previously published terminal result for the same durable
action or nonce.

## 9. Fulfillment

An action bundle contains the same receiver manifest and Authority plus a
receiver-signed Fulfillment. Only a receiver key admitted by the locally pinned
manifest may issue a succeeded Fulfillment.

Every Fulfillment MUST bind:

- format = fireproof.receiver-fulfillment/v0.1;
- exact R2 profile, receiver_ref, tenant_ref, and environment_ref;
- kind = fulfillment;
- authority_payload_sha256 from the exact verified Authority payload bytes;
- receiver_manifest_payload_sha256;
- terminal_action_key, action_instance_id, and use_nonce_commitment;
- actor_ref_commitment, supplier_ref_commitment, operation_ref, and
  permitted_field exactly as present in the Authority;
- payment_destination_commitment, expected_record_version, and
  expected_state_commitment exactly as present in the Authority;
- receiver_transaction_id;
- accepted_at and commit_prepared_at;
- receiver_signer_id and fulfillment_signer_keyid pinned before the transaction
  commit; and
- outcome status and bounded reason code.

receiver_transaction_id, receiver_signer_id, and action_instance_id are each 1-128
character ASCII strings matching [A-Za-z0-9][A-Za-z0-9._:-]*. The DSSE
signature keyid MUST equal fulfillment_signer_keyid, and receiver_signer_id
MUST identify exactly one fulfillment_signers entry whose keyid matches it.
No unknown fields are permitted. The exact outcome object is either:

    {"status": "succeeded", "reason_code": "committed"}

or:

    {"status": "rejected", "reason_code": "state_changed"}

or:

    {"status": "rejected", "reason_code": "no_effect"}

A succeeded Fulfillment MUST additionally bind:

- before_state_commitment and after_state_commitment;
- record_version_before and record_version_after; and
- outcome.status = succeeded.

A state_changed rejected Fulfillment MUST bind:

- observed_state_commitment and observed_record_version; and
- outcome.status = rejected and reason_code = state_changed.

A succeeded payload contains the success fields and omits observed_state_commitment
and observed_record_version. A rejected state_changed or no_effect payload
contains the observed fields and omits before_state_commitment,
after_state_commitment, record_version_before, and record_version_after.

For state_changed, at least one of the following MUST be true:

    observed_state_commitment != Authority.expected_state_commitment
    observed_record_version != Authority.expected_record_version

The R2 verifier MUST reject a rejected state_changed Fulfillment that presents
the Authority expected state and version unchanged.

A no_effect rejected Fulfillment MUST bind observed_state_commitment and
observed_record_version, set outcome.status = rejected and reason_code =
no_effect, and satisfy both:

    observed_state_commitment == Authority.expected_state_commitment
    observed_record_version == Authority.expected_record_version

The R2 verifier MUST reject a no_effect Fulfillment whose observed state or
version differs from the Authority expectation.

The Fulfillment payload bytes are constructed and stored before commit; the
signature is applied only after commit. A signed success therefore cannot
escape a rolled-back write. The profile does not allow an Authority issuer,
agent, or intermediary to create a succeeded Fulfillment.

## 10. Bundle and offline verification

An R2 bundle has a strict JCS bundle.json inventory with exactly format,
bundle_kind, profile, and entries. Its format is
fireproof.evidence-bundle/v0.2 and its profile is the exact R2 profile. Each
entries item has exactly role, path, media_type, and sha256. sha256 is lower
case hexadecimal. media_type is
application/vnd.fireproof.dsse-envelope.v1+json for every role.

The only role/path pairs are:

| Role | Exact path |
| --- | --- |
| manifest | receiver-manifest.dsse.json |
| authority | authority.dsse.json |
| fulfillment | fulfillment.dsse.json |

Paths are plain nonempty POSIX filenames with no directory, slash, backslash,
or traversal component. The bundle directory contains bundle.json and exactly
the artifact files declared by entries; symlinks, hard links, extra files, and
non-regular files are invalid.

A decision bundle has bundle_kind = decision and exactly manifest plus
authority entries. An action bundle has bundle_kind = action and exactly
manifest, authority, and fulfillment entries. A decision bundle never claims
receiver acceptance or success.

A decision bundle contains:

    bundle.json
    receiver-manifest.dsse.json
    authority.dsse.json

An action bundle additionally contains:

    fulfillment.dsse.json

The receiver root set is an explicit verifier input, not a root selected by the
sender at verification time. The synthetic demonstrator may export a safe root
file beside a bundle for convenience, but the verifier invocation must name it
explicitly.

The separately buildable R2 verifier MUST validate:

1. strict directory inventory, file digests, and bundle/profile version;
2. DSSE/JCS/Ed25519 validity;
3. root-to-manifest, manifest-to-authority, and manifest-to-receiver signer
   relations;
4. Authority-to-Fulfillment linkage and equality of their signed receiver,
   manifest, action, nonce, target, and state-commitment fields;
5. Authority time versus accepted_at and commit_prepared_at;
6. commitment encoding, profile-required field shape, and terminal outcome
   shape, including the succeeded and state_changed semantic relations in
   section 8 and the no_effect relation in section 9; and
7. that a decision bundle has no claimed receiver success and an action bundle
   has exactly one required receiver Fulfillment.

Valid means only that these signed and linked claims are valid under the
explicit supplied root. It does not prove a real supplier, bank account,
payment, organization policy, signer key custody, legal non-repudiation, or
real-world time. The offline verifier cannot recompute private request, nonce,
or state preimages because those values are deliberately absent from the
bundle; their recomputation is a receiver-side acceptance check. Public safe
vectors exercise those formulas separately.

## 11. Explicit non-goals

This profile does not define a generic ERP connector, SAP integration, banking
payment protocol, credential brokerage, generic HTTP forwarding, distributed
exactly-once behavior, high availability, public API, public deployment, or a
generic policy language.
