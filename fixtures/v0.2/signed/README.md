# Synthetic v0.2 signing vectors

These vectors contain public Ed25519 JWKs and detached DSSE signatures only.
No private key was persisted or tracked. The companion intent disclosure is
deliberately public, synthetic test data; it is not an example of production
receipt storage.

The verifier must treat `keyid` as a lookup hint, not a source of trust. It
pins verification to the supplied Trust Root and the role-constrained keys in
the signed trust manifest.
