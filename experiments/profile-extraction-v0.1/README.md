# Experimental profile extraction v0.1

This directory is non-normative and exists only to test whether the current
receiver-acceptance architecture can separate a stable acceptance kernel from
operation-specific wire fields.

It does **not** change the existing Supplier Master profile, schemas, vectors,
or fixture bytes.

The descriptor names only the profile-level values that are currently
hardcoded into the R2 verifier/reviewer:

- profile identifier;
- operation identifier;
- permitted field;
- target commitment field name; and
- mutation commitment field name.

Two descriptors are provided:

- `supplier-master.profile.json` maps the existing R2 Supplier Master wire
  contract without renaming any field.
- `customer-credit.profile.json` describes a second transactional domain using
  `customer_ref_commitment` and `credit_limit_commitment`.

The experiment passes only if a verifier/runtime refactor can consume a profile
descriptor while preserving the committed Supplier Master bytes and semantics.
The descriptor is intentionally small: cryptographic formats, trust roles,
action identity, replay behavior, terminal lifecycle, and evidence bundle
semantics remain kernel concerns rather than profile configuration.
