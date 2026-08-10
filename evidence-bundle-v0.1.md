# FireProof Evidence Bundle v0.1

- Status: internal candidate implemented by R1; not a public release
- Bundle format: `fireproof.evidence-bundle/v0.1`
- Receipt profile: `fireproof.action-receipt/v0.2`

## Purpose

An Evidence Bundle is the portable unit an outside party can take away from an
agent runtime and verify locally. It carries signed claims and the byte-level
inventory needed to inspect those claims. It is not a database export, a
credential container, a dashboard link, or a self-authenticating trust root.

R1 deliberately admits only a directory bundle. ZIP, TAR, URLs, network key
lookup, encrypted payload transport, and embedded credentials are out of
scope. A later format may add one of them only as a new version with attack
tests for extraction, path, and canonicalization behavior.

## Directory shape

The bundle directory contains exactly `bundle.json` and the files declared by
its inventory. It may not contain a symlink, an unlisted file, a special file,
or a path that escapes the directory. R1 permits at most five files, eight
directories, and four directory levels. Empty or unlisted directories are
rejected too.

The hostile-directory implementation is verified only on Linux with
`O_NOFOLLOW`. It opens and reads the listed files through no-follow directory
descriptors, rejects hard-linked files, and rejects name-replacement races it
can observe. On another platform the R1 CLI returns `unsupported`; it does not
silently claim equal filesystem protections. A verifier operator must still
use an immutable or access-controlled copy when a hostile local process can
modify the same underlying file while it is being read.

For an `action` bundle, these signed entries are required:

```text
bundle.json
manifest.dsse.json
authority.dsse.json
fulfillment.dsse.json
```

A `decision` bundle contains the manifest and Authority but no Fulfillment.
An optional `disclosure` entry is accepted only for the public synthetic test
profile. It never establishes production validity and is never required for a
receipt to verify.

`bundle.json` has this exact shape:

```json
{
  "format": "fireproof.evidence-bundle/v0.1",
  "bundle_kind": "action",
  "profile": "fireproof.action-receipt/v0.2",
  "entries": [
    {
      "role": "manifest",
      "path": "manifest.dsse.json",
      "media_type": "application/vnd.fireproof.dsse-envelope.v1+json",
      "sha256": "lowercase-hex-sha256-of-the-exact-file-bytes"
    }
  ]
}
```

The index is strict JSON with no duplicate names or floating-point values. The
verifier permits only the roles `manifest`, `authority`, `fulfillment`, and
`disclosure`; each role and path appears at most once. Every listed artifact is
between 1 byte and 2 MiB; the combined artifact limit is 6 MiB.

`schemas/bundles/v0.1/evidence-bundle.schema.json` is the normative structural
schema and is carried inside the installed verifier. The verifier applies that
schema and then the stricter role, file, media-type, path, and directory rules
in this document.

The inventory is integrity metadata, not a signature or source of trust. Its
deterministic `bundle_digest` is:

```text
SHA-256("FIREPROOF-EVIDENCE-BUNDLE-V1\\0" || JCS(bundle.json object))
```

It identifies the normalized inventory whose artifact hashes were checked. It
does not establish identity or substitute for the signed receipts.

## Trust input

The verifier caller must provide `--trust-root PATH`. The Trust Root is a local
JSON file with this exact shape:

```json
{
  "format": "fireproof.trust-root-set/v0.1",
  "keys": [
    {
      "public_jwk": {"kty": "OKP", "crv": "Ed25519", "x": "..."},
      "jwk_thumbprint_sha256": "..."
    }
  ]
}
```

The root set is explicitly chosen by the verifier operator. It is not carried
inside the bundle, fetched from FireProof, inferred from DSSE `keyid`, or
accepted from a network URL. `keyid` remains only an unauthenticated lookup
hint; R1 does not use it to establish identity.

The root verifies the Trust Manifest. The signed manifest then controls which
role-bound keys may sign Authority and Fulfillment receipts for which scope and
claimed time interval. A Trust Root key may not reappear as a receipt key.

## Cryptographic profile

R1 freezes the existing v0.2 profile:

- DSSE v1 JSON envelope with the standard textual decimal-length PAE;
- RFC 8785 JCS UTF-8 payload bytes;
- Ed25519 only; every signature is exactly 64 raw bytes;
- one DSSE signature per R1 artifact;
- strict standard or URL-safe canonical base64, with no mixed alphabets;
- public Ed25519 JWK thumbprints following RFC 7638 construction.

DSSE v1.0.2 permits standard and URL-safe Base64. R1 accepts either padded or
unpadded canonical RFC 4648 representation, rejects whitespace, mixed
alphabets, non-canonical pad bits, and illegal characters, and validates its
standard DSSE PAE against the published `hello world` reference vector.

The manifest payload type is
`application/vnd.fireproof.trust-manifest.v0+json`. Authority and Fulfillment
payloads use `application/vnd.fireproof.action-receipt.v0.2+json`. The verifier
authenticates the raw decoded DSSE payload bytes first, then parses those exact
same bytes once, requires JCS byte equality, and finally performs schema and
semantic validation.

COSE is not part of R1. It is a valid future transport candidate for compact
CBOR-native environments, but adding it now would create two signing truths
before the portable evidence boundary itself is proven.

## Verification result

The installed command is:

```text
fireproof-verify verify BUNDLE_DIR --trust-root ROOT.json --at 2026-07-15T12:01:00Z
```

The default output is one JSON report. It includes the contract version,
bundle digest, profile, explicit verification time, ordered checks, claims,
and limits. It never mixes a JSON result with progress logs.

| Outcome | Exit code | Meaning |
| --- | ---: | --- |
| `valid` | 0 | Required artifacts, signatures, roles, scope, and receipt linkage are valid under the supplied root. |
| `invalid` | 10 | The bundle is malformed, altered, contradictory, unauthorized by its signed manifest, or temporally invalid at its claimed action time. |
| `unverifiable` | 11 | The supplied root cannot establish a unique trusted manifest signer. |
| `unsupported` | 12 | The bundle, Trust Root, payload type, or cryptographic profile is outside R1. |
| `operational_error` | 70 | The verifier cannot safely read a required local resource. |

`valid` means `signer_authorized_under_supplied_manifest`. It explicitly does
not mean that a real-world external effect, key custody, policy correctness,
legal non-repudiation, or objective execution time has been proven.

The manifest's current status is reported at `--at` (or recorded current UTC
when it is omitted), but a later manifest expiry does not erase a historically
well-formed claimed chain. R1 has no trusted timestamp or transparency proof;
the receipt timestamps remain issuer-attested claims.

## R1 exclusions

- no production disclosure of raw input, result bytes, secrets, or credentials;
- no policy re-evaluation from a referenced policy hash alone;
- no revocation-at-signing claim without reliable time and published revocation
  evidence;
- no transparency log, timestamp authority, blockchain, or automatic network
  fetch;
- no adapter execution, SDK, or real external write.

Those boundaries are intentional. R2 will create one constrained action
runtime; R4 may admit timestamping, transparency, another transport, or a
second verifier implementation only through the technology-admission process.
