# Signed Test Vector

`ed25519-dsse-vector.json` is synthetic test material for the v0 DSSE profile.
It contains a public Ed25519 key, a key identifier, and signatures for the
canonical authority and fulfillment payload fixtures.

It deliberately does not contain a private key or static copies of signed
payload bytes. `scripts/validate-contract-schema.py` derives each envelope
from `fixtures/valid/` using RFC 8785 JCS, then verifies the stored signature
over the resulting DSSE PAE bytes.

This vector proves only that the stated test key verifies the stated synthetic
fixture bytes. It is not a production trust manifest, issuer identity, key
custody mechanism, or evidence of an external action.
