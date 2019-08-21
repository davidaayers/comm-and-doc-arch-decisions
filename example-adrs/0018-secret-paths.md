# 18. Define Paths to Secrets

Date: 06/13/2017

## Context

As we increase our adoption of [Vault](https://www.vaultproject.io/), we need to establish a convention of our paths to secrets in the [generic backend](https://www.vaultproject.io/docs/secrets/generic/).

* The convention should support a variety of secrets - passwords, TLS certificates, SSH keys, etc.
* The convention should allow for a variety of granularity. It should allow for allowing a client read access to a single secret (e.g. app user's database password), as well as allowing a client read/write access to a entire group of secrets (e.g. r/w access to all database password for a DBA).

## Decision

Secrets added to Vault's generic backend should use the following path convention:

`/usage/environment/scope/key`

Where:

* `usage` - What is the usage of this secret
* `environment` - In what environment will this secret be used
* `scope` - The scope of the secret
* `key` - The key used to look up the secret

Below are some examples:

### Database passwords

The password for the `example` user in the FOO databases would be located at:

* `/database/development/foo/example`
* `/database/test/foo/example`
* `/database/production/foo/example`

Developers would have read only access to `/database/development/*`.

DBAs would have read/write access to `/database/*`

### TLS certificates

The wild card TLS certificates would be located at:

* `/certificate/development/www/cert`
* `/certificate/test/www/key`
* `/certificate/production/www/key`

The platforms team would have read/write access to `/certificate/*`.

## Consequences

All new secrets added to Vault's generic backend should follow this convention. Existing secrets can remain in their current path, but should be migrated when possible. If a new secret needs to be added and its usage is not addressed here, this ADR should be amended.

## Related Categories

* Platforms
* Security
