# R2 Receiver Acceptance Commitment Vectors v0.1

These are deterministic, synthetic-only interoperability vectors for the
commitment formulas in the R2 profile, sections 6 and 8.

They are neither an Evidence Bundle nor a receiver outcome. They make no claim
that an ERP write occurred.

## Safety boundary

Every reference, destination, nonce, and state in commitment-vectors.json is
fixed test data. It MUST NOT be copied into a production Authority, request,
state store, log, or bundle. In particular, raw request values and the raw
nonce remain forbidden from portable R2 evidence; they are exposed here only so
independent implementations can reproduce the hashes.

This directory is the narrowly scoped synthetic-vector exception described in
profile section 6. It is not an Evidence Bundle, an Authority, a Fulfillment,
or a production fixture; it contains no real, production, or customer-derived
value.

## Reproduction rule

For each JSON commitment:

1. Serialize the named jcs_object with RFC 8785 JCS and UTF-8.
2. Encode domain_tag_utf8_with_nul as UTF-8, including its terminating NUL
   character.
3. Concatenate the domain-tag bytes, the JCS bytes, and no delimiter other
   than the NUL already present in the tag.
4. Apply SHA-256 and render the 32-byte result as lower-case hexadecimal.

For use_nonce_commitment, concatenate the domain-tag bytes with the decoded
32-byte nonce instead of JCS bytes. The nonce must be unpadded base64url and
decode to exactly 32 bytes.

The vector objects deliberately contain only ASCII strings and integer record
versions. That makes their expected JCS output unambiguous; it does not license
an implementation to replace JCS with an arbitrary JSON serializer for other
inputs.

## Required checks

An implementation claiming this profile should reproduce every expected digest
exactly and should additionally reject a boundary nonce which is padded,
contains a non-base64url character, or does not decode to exactly 32 bytes.
Those negative parser cases are protocol requirements, not portable evidence
fixtures.

The state vectors are linked intentionally: the before-state result is the
Authority expected-state commitment for record version 1, and the after-state
result uses the same receiver, tenant, supplier commitment, and replacement
destination at record version 2.
