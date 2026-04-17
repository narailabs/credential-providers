# Changelog
All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- `CredentialResolver` class — own a registry per instance, enabling isolation between plugins/libraries that share the process.
- `defaultResolver` export — the module-level singleton backing the free functions.
- `ParseCredentialRefOptions` — `{ strict?, resolver? }` on `parseCredentialRef`. In `strict: true` mode, unknown provider prefixes throw instead of returning null.
- `CloudSecretsProvider.forTesting({ client, ...config })` — public, supported alternative to the old `_client` injection hook.
- `cacheTtlMs` option on `CloudSecretsProvider` — opt-in in-memory TTL cache for getSecret (default 0 = off). Errors are never cached. `clearCache()` method for manual invalidation.
- `allowLoosePermissions` option on `FileProvider` — explicit opt-out of the POSIX mode check, decoupled from the warning-suppression flag.
- `SECURITY.md` — vulnerability reporting contact, supported-versions policy.

### Changed
- **BREAKING**: `CredentialProvider.getSecretSync?` removed from the interface; `EnvVarProvider.getSecretSync`, `FileProvider.getSecretSync`, `KeychainProvider.getSecretSync` removed. Use `await p.getSecret(name)`. The sync surface was never used internally and was blocking-unsafe on keychain backends.
- **BREAKING**: `resolveSecret` now throws `AggregateError` (with `.errors` preserving every thrown error) when every provider in the chain throws, instead of re-throwing only the last error. Consumers that `try/catch` the rejection get richer diagnostics.
- **BREAKING**: `FileProvider`'s `suppressWarning` option no longer bypasses the POSIX 0o077 mode check. Consumers who need loose permissions must now set `allowLoosePermissions: true`.
- **BREAKING**: `CloudSecretsConfig._client` field removed from the public interface. Use `CloudSecretsProvider.forTesting(...)` instead.
- **BREAKING**: `KNOWN_PROVIDERS` no longer exported. Reference-string recognition is now driven by the resolver's registry plus a small built-in fallback allowlist (`env_var`, `keychain`, `file`, `cloud_secrets`). Custom providers registered via `registerProvider` are automatically recognized by `parseCredentialRef`.
- Node `engines` widened from `>=20.0.0 <21.0.0` to `>=20.0.0` — Node 22 LTS is now supported.
- `package.json` `repository`, `bugs`, `homepage` point at the actual `narailabs/credential-providers` repo.
- Internal-consumer references (wiki_db, Phase H, doc-wiki, wiki.config.yaml) scrubbed from source and test comments.

## [0.1.0] - 2026-04-16
### Added
- Initial release: pluggable credential providers for Node.
- EnvVarProvider, FileProvider, KeychainProvider (macOS/Linux), CloudSecretsProvider (AWS/GCP/Azure).
- `parseCredentialRef` for `provider:key` reference strings.
- `resolveSecret(name, {provider, fallback})` chain helper.
