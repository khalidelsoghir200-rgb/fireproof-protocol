# R2 Receiver Acceptance Schemas v0.1

This schema family belongs only to the ERP supplier-master R2 profile. It is
additive to R1: none of these files widen, replace, or reinterpret an R1
schema.

## Files

- trust-root-set.schema.json — explicit verifier root input.
- receiver-trust-manifest.schema.json — root-signed receiver admission payload.
- authority.schema.json — Authority payload.
- fulfillment.schema.json — receiver Fulfillment payload.
- dsse-envelope.schema.json — strict one-signature R2 DSSE envelope.
- evidence-bundle.schema.json — strict R2 bundle inventory.
- boundary-request.schema.json — private local request shape only.
- common.schema.json — shared scalar definitions.

## What JSON Schema does not prove

Schema validation is necessary but not sufficient. The R2 verifier and receiver
must separately enforce all semantic rules that require bytes, cryptography,
cross-artifact equality, or durable state, including:

- canonical padded standard base64 decoding, JCS byte equality, DSSE v1 PAE,
  Ed25519 verification, and JWK-thumbprint recomputation;
- root-to-manifest and manifest-to-signer admission, including duplicate logical
  identifiers and key IDs;
- Authority/Fulfillment digest and field linkage, time ordering, and outcome
  state relations;
- receiver-local active-manifest pinning, configured-supplier scope, replay
  fencing, conditional state mutation, and post-commit signing; and
- request byte-size, UTF-8/BOM, and duplicate-JSON-name rejection.

The schemas intentionally reject unknown fields. They define no generic policy
language, transport proxy, broker, or external ERP connector.

## Offline schema resolution

The `https://schemas.fireproof.dev/receiver-acceptance/v0.1/` values used in
`$id` and `$ref` are canonical schema identifiers. They are not a requirement
that a verifier or schema consumer fetches a network resource.

For an offline distribution, load every `*.schema.json` file in this directory
into the resolver under its declared `$id`, then resolve absolute `$ref` values
through that in-memory catalog. The R2 verifier itself does not fetch schema
URLs. A network-hosted schema namespace is outside this profile until it is
deliberately operated with valid TLS and byte-matched content.
