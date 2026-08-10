# v0.2 Cross-Runtime Compatibility Vectors

`v0.2-conformance.json` is the public, synthetic oracle shared by the Python,
TypeScript, and Rust conformance runners. It is intentionally narrower than the
detached Python verifier: it establishes agreement on the listed bytes and
negative cases, not a general verifier or production interoperability claim.

## What is pinned

- exact JCS UTF-8 bytes and SHA-256 values for the signed manifest, Authority,
  and Fulfillment fixture payloads;
- Unicode/key-order and number-normalization JCS probes;
- strict JSON rejection for duplicate names (including escaped duplicates),
  `NaN`, BOMs, invalid UTF-8, unpaired surrogates, invalid escapes, control
  bytes, trailing bytes, and malformed input;
- domain-separated intent, nonce, and public-scope commitments;
- closed Ed25519 JWK parsing and RFC 7638 thumbprints;
- DSSE PAE byte construction, canonical standard/URL-safe base64 admission,
  and the public synthetic Ed25519 signatures;
- signed-payload canonicality and Authority/Fulfillment linkage negatives.

The file stores binary values as unpadded base64url and uses RFC 6901 JSON
Pointers. Its source paths point only to tracked `fixtures/v0.2` files. The
golden payload bytes are deliberately embedded instead of regenerated during a
test: a change to those bytes requires a visible vector diff and review across
all three runtimes.

## Rules for changes

Do not add a production key, raw customer nonce, credential, endpoint, or
customer payload. Do not change a golden result to make one runtime pass. A
contract change needs its own versioned decision, new vectors, and independent
VM evidence from Python, TypeScript, and Rust.

Vector agreement never proves current trust, replay prevention, adapter
execution, exactly-once execution, or an external side effect.
