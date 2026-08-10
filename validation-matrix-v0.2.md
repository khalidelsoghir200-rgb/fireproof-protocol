# Action Receipt v0.2 Validation Matrix

This matrix is the bounded test contract for the v0.2 synthetic-vector
verifier. It verifies signed receipts and a supplied manifest chain; the
verifier alone does not prove replay prevention, current revocation status, or
an independently observed external side effect.

| Check | Authority | Fulfillment | Trust Manifest / DSSE | Required result |
| --- | --- | --- | --- | --- |
| Strict JSON parsing | yes | yes | yes | duplicate names and `NaN`/`Infinity` reject before schema or JCS |
| JSON Schema | v0.2 | v0.2 | manifest v0 / envelope v0.2 | invalid shape rejects |
| Canonical payload | JCS bytes | JCS bytes | JCS bytes | raw signed bytes must equal JCS of parsed payload |
| DSSE payload type | receipt v0.2 only | receipt v0.2 only | manifest v0 only | mismatched type rejects |
| Trust root | n/a | inherited through Authority | locally supplied root only | manifest verifies before its keys are considered; root key cannot be a receipt key |
| JWK identity | authority role only | fulfillment role only | Ed25519 `OKP` public JWK | 32-byte canonical `x` and RFC 7638 thumbprint must match |
| `keyid` handling | hint only | hint only | hint only | changing it cannot select or authorize a key |
| Signature counting | one qualified match | one qualified match | one synthetic-root signature | unqualified extras grant nothing; multiple qualified matches are ambiguous |
| Scope and relation | exact manifest rule | inherited Authority relation | tenant-scoped rule set | tenant, environment, target, `PATCH`, and arguments profile match exactly |
| Authority linkage | manifest ID/revision/raw digest | n/a | source manifest | fail closed on any mismatch |
| Fulfillment linkage | source Authority | ID/raw digest/profile/commitment/action instance | n/a | fail closed on mismatch or denied Authority |
| Time linkage | issue before expiry | inclusive Authority issue/expiry window | key and manifest validity | fail closed outside permitted intervals |
| Intent disclosure | source commitment | n/a | n/a | with disclosure, reproduce both commitments; without it, report `intent_unverifiable` |
| Result meaning | signed decision | signer-attested outcome | n/a | never promote to proof of an external side effect |

The reference VM command is:

```bash
bash scripts/verify-contract-v0.2.sh
```

The fixture disclosure under `fixtures/v0.2/test-only-disclosure/` is public,
synthetic test data. Production receipts must never carry its private scope,
blinding nonce, or use nonce.

## Isolated action-instance replay-fence reference

The separate state-machine harness accepts only a `VerifiedActionContext` that
a production integration must derive from a successful verifier result retaining
the raw canonical Authority payload bytes. In the harness, that provenance is a
fixture precondition, not an authentication boundary.

| Check | Required result |
| --- | --- |
| Durable identity | primary identity is verified Authority raw-payload SHA-256 plus signed UUIDv7 action-instance ID; `(tenant_ref, action_instance_id)` reuse also rejects |
| Pre-consume binding | `allow`, nonce commitment, authenticated tenant/principal/environment/executor context, Authority time window, `max_uses: 1`, and exact tenant/environment registered adapter profile/target/`PATCH`/arguments profile/idempotency support must all match before mutation |
| Durable transition | one transaction records `reserved` and `consumed_before_dispatch`; a second committed transaction records `dispatch_started` before any modeled callback |
| Terminality | only a started, locally invoked attempt can become a definite `fulfilled` outcome or terminal `indeterminate`; duplicate start, callback, terminal record, or redispatch rejects |
| Restart and rollback | reopening observes the durable record while ephemeral permits and raw idempotency keys do not survive; injected write failures leave no partial transition |
| Scope retention and storage hygiene | Authority issuer plus every signed public-scope reference survive into the durable row; raw nonce and raw idempotency key are absent from SQLite and its WAL; only an idempotency-key SHA-256 fingerprint persists |
| Schema compatibility | reference SQLite `user_version` is `2`; a legacy layout without issuer/principal/environment columns rejects rather than being altered or backfilled |
| Conservative recovery | an unfinished record can only become `indeterminate`; a production recovery path needs an external coordinator/fence to establish that an active worker cannot still dispatch |

The reference VM command is:

```bash
bash scripts/verify-action-instance-state-machine.sh
```

It reruns the v0.2 verifier before the state tests. Its successful result proves
only the isolated SQLite model and an at-most-once modeled callback per store
instance. It does not prove verifier-context provenance in an integration,
distributed recovery fencing, exactly-once execution, adapter execution, or an
external side effect.

## Isolated HTTP JSON write adapter reference

The adapter harness accepts a typed verifier-derived Authority context and the
committed synthetic disclosure only. It is a no-network reference for the one
`fireproof.http-json-write.v1` profile, not a generic HTTP client.

| Check | Required result |
| --- | --- |
| Strict disclosure bytes | reject BOM, invalid UTF-8, duplicate names, non-finite constants, trailing data, lone surrogates, excessive depth/size, and non-exact disclosure shapes before a row or callback exists |
| Commitment binding | exact profile, Authority issuer, action-instance ID, complete public scope, private resource/arguments, blinding nonce, and use nonce reproduce the signed commitments before consumption |
| Private-input boundary | synthetic resource is constrained `kind`/`id`; arguments are a JSON object; secret references must be empty; endpoint/URL/query/header/cookie/credential/request-method material rejects |
| Registered delivery shape | request has only opaque tenant/principal/environment/executor/target references, fixed `PATCH`, fixed `application/json`, and JCS bytes for `{resource, arguments}`; it has no URL, query, header map, credential, or raw nonce |
| Invocation order | normalize and bind, consume, commit `dispatch_started`, then invoke one wrapper-owned in-memory transport callback with the ephemeral idempotency key |
| Modeled outcome | explicit synthetic 2xx is `succeeded`; explicit synthetic 4xx is `failed`; exception, malformed response, 5xx, or any ambiguity is terminal `indeterminate` with no retry |
| Storage and reporting | SQLite/WAL/reportable records omit private disclosure/body/resource, blinding/use nonces, secret references, raw exception text, and raw idempotency key |
| Network boundary | source imports no network client and the harness supplies no URL, endpoint, DNS, credential, or customer data |

The reference VM command is:

```bash
bash scripts/verify-isolated-http-json-adapter.sh
```

It reruns the v0.2 verifier and durable state harness first. A passing result
proves only deterministic synthetic normalization and one in-memory modeled
dispatch path. It does not prove live HTTP behavior, context provenance in a
production integration, secret retrieval, current trust, distributed recovery
fencing, exactly-once external execution, or an external side effect.
