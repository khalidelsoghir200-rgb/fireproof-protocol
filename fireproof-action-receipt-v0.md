# FireProof Action Receipt v0

- Status: internal candidate; not a public release
- Schema version: `0.1`
- DSSE payload type: `application/vnd.fireproof.action-receipt.v0+json`
- Cryptographic profile: Ed25519 over DSSE v1 PAE

## Purpose and boundary

An Action Receipt is signed evidence about an agent action. It has two payload
kinds:

1. `authority-decision`: a time-bounded decision made before a consequential
   action; and
2. `fulfillment-receipt`: a result a signer attests after an attempted action
   that references the authority payload.

The receipt can prove only the verification level documented in ADR-0007. A
valid signature does not by itself prove a real-world side effect, delegation
truth, policy compliance, key ownership, or legal non-repudiation.

## Envelope and bytes

An Action Receipt is carried in a DSSE JSON envelope with:

- exact `payloadType` of
  `application/vnd.fireproof.action-receipt.v0+json`;
- `payload`: base64 encoding of the exact UTF-8 payload bytes;
- `signatures`: one or more signature entries, each with non-empty `sig` and
  optional `keyid`.

Producers emit padded base64url. A compatible verifier accepts standard or
URL-safe base64 but rejects whitespace, malformed padding, mixed alphabets,
non-canonical pad bits, duplicate JSON object names, unsupported payload type,
or oversized input. Unknown DSSE fields have no security meaning.

The signer signs the DSSE PAE of the raw decoded payload bytes. The verifier
MUST verify those bytes before parsing them, and MUST pass that same verified
byte sequence to payload parsing. It MUST NOT parse an envelope a second time to
retrieve another payload.

## Payload serialization and digest

Producers serialize payload objects using RFC 8785 JCS to UTF-8 bytes. This is
a producer/interoperability rule; DSSE authentication always covers the actual
raw payload bytes.

`sha256` values are lower-case hexadecimal SHA-256 digests. A Fulfillment
Receipt's `authority.payload_sha256` is SHA-256 over the exact raw authority
payload bytes before base64 encoding, not over the envelope, decoded object, or
an arbitrary reserialization.

## Authority Decision payload

Authority decisions use
`schemas/authority-decision-v0.schema.json`. Required semantics beyond JSON
Schema are:

- `issued_at` and `expires_at` are UTC RFC 3339 timestamps with second
  precision, and `expires_at` MUST be later than `issued_at`;
- `receipt_id` is a lower-case UUIDv7;
- the action carries a non-secret `intent_digest`, not raw customer input;
- `policy` binds policy identifier, version, and bytes digest;
- a decision is `allow` or `deny`; and
- a fulfillment can link only to an `allow` decision that is not expired at its
  `issued_at`.

## Fulfillment Receipt payload

Fulfillment receipts use
`schemas/fulfillment-receipt-v0.schema.json`. A verifier that has the linked
authority payload MUST fail closed unless:

1. `authority.decision_id` equals the Authority Decision `receipt_id`;
2. `authority.payload_sha256` equals the digest of the raw verified authority
   payload bytes;
3. `action.intent_digest` equals the Authority Decision action digest;
4. the authority decision is `allow`; and
5. authority `issued_at` <= fulfillment `issued_at` <= authority `expires_at`.

Equality at either time boundary is allowed because v0 timestamps have only
second precision. It does not by itself prove causal ordering within that
second.

`result_digest` and evidence digests identify result/evidence bytes held outside
the receipt. They do not make that material available or prove the material is
complete.

## Trust and key selection

The envelope contains no algorithm declaration. A trust manifest outside the
receipt maps allowed key fingerprints to Ed25519 public keys, receipt types, and
optional policy/tenant constraints. `keyid` can select candidate keys but cannot
be trusted until the signature verifies against the manifest.

Network key lookup, key revocation, transparency logging, timestamp authority,
and private policy evaluation are out of scope for v0. Missing required evidence
produces `unverifiable`, not an optimistic pass.

## Current tool-write boundary

This internal candidate does not yet define a versioned preimage for
`intent_digest`, a signed tenant/environment/target scope, an authority-minted
action instance, signed issuer identity, or a trust-manifest schema. Therefore
its current Authority-to-Fulfillment linkage proves only the stated bytes and
field equality; it cannot authorize a real tool write or prove that the action
occurred in a particular customer, endpoint, or resource context. These are
mandatory Gate 2 additions before a tool-write demo or a stronger authorization
claim.

## Fixtures and validation

The `fixtures/` directory contains synthetic data only. Every schema needs a
positive fixture and explicit negative cases. Structural validity, semantic
linkage, base64 behavior, signature verification, and policy evaluation are
separate test categories; a pass in one category must not be reported as a pass
in another.

The signed test vector contains a synthetic public key and signatures, not
duplicated envelope payloads or a private key. The validator derives its DSSE
envelopes from the JCS-canonical positive fixtures, which remain the sole
payload source of truth.
