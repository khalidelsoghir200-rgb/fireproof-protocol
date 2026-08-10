# FireProof Protocol

Evidence format specification, schemas, canonical serialization rules, signing vectors, and safe test fixtures for the FireProof action-evidence format.

## What is in this repo

| Path | Contents |
|------|----------|
| `evidence-bundle-v0.1.md` | R1 evidence bundle contract |
| `fireproof-action-receipt-v0.md` | Action receipt wire format |
| `profiles/` | Operation-specific profiles (e.g. `erp-supplier-master-change-v0.1.md`) |
| `schemas/` | JSON Schema definitions |
| `vectors/` | Commitment byte vectors for cross-runtime conformance |
| `validation-matrix-v0.2.md` | Full validation matrix for R2 |
| `fixtures/` | Safe synthetic test fixtures (no real keys, no real data) |

## Design principles

**Vendor-neutral.** The format uses JCS (RFC 8785) for canonical serialization, DSSE v1 for envelope signing, and Ed25519 for signatures. These are stable, open standards. No FireProof service is required to verify a bundle.

**Minimal disclosure.** Evidence proves what is necessary. Private request data, raw state values, and supplier credentials are represented as commitments (SHA-256 hashes) in the portable bundle, not as plaintext.

**Long-lived.** A bundle produced today must be verifiable years later without calling back to any live service. The format is explicitly versioned and designed to outlive specific agent platforms, cloud providers, or integrators.

**Separation of roles.** Three distinct signing roles: Authority (upstream authorization), Receiver (acceptance decision), Fulfillment (post-commit attestation). Each uses a separate key and scope.

## Commitment vectors

`vectors/` contains the exact byte formulas for every commitment field. These are published so any implementation in any language can reproduce the same canonical bytes independently. Cross-runtime agreement on commitment bytes is a falsifiable correctness test.

## Fixtures

All fixtures are synthetic. No real supplier data, no real keys, no real payment destinations. The `signed/` directory contains test-only Ed25519 vectors with `"private_key_persisted": false`.

## License

Apache 2.0
