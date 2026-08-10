# R1 Evidence Bundle fixtures

`valid/action` is a public synthetic directory bundle generated from the
existing v0.2 canonical payload fixtures and their detached Ed25519 test
signatures. It has no private key, credential, customer payload, or live
service record.

Regenerate it on the VM with:

```text
.venv-contract/bin/python scripts/generate-r1-fixtures.py
```

The regeneration script must leave the verifier result unchanged at
`2026-07-15T12:01:00Z`. It is not a signer and cannot create a new valid
production receipt.
