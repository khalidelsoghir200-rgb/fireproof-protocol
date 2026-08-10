# Action Receipt v0 Validation Matrix

| Check | Authority | Fulfillment | Envelope | Required result |
| --- | --- | --- | --- | --- |
| JSON and duplicate-key parsing | yes | yes | yes | fail closed on malformed/ambiguous JSON |
| JSON Schema validation | yes | yes | yes | positive fixture passes; each negative fixture fails |
| Timestamp semantics | issued before expiry | authority issued at <= fulfillment issued at <= authority expiry; equality is allowed at second precision but does not prove causal ordering | n/a | fail closed |
| Action linkage | source of digest | exact digest match | n/a | fail closed |
| Authority linkage | n/a | ID and raw-payload digest match | payload bytes retained | fail closed |
| DSSE payload type | n/a | n/a | exact allowlist | fail closed |
| Base64 decoding | n/a | n/a | strict standard/url-safe acceptance | fail closed on malformed input |
| Signature | signed input | signed input | Ed25519 PAE over raw payload bytes | unknown/untrusted/bad signature fails |
| Policy semantics | policy hash only | policy hash reference | n/a | `policy_evaluated` only with disclosed policy/input |

No test can promote a result to a stronger verification level than the evidence
it consumed. For example, structural fixture success is not a signature pass,
and a valid signature is not proof of an external side effect.
