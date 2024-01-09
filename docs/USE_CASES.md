# Use Cases

In designing spec changes for authentication and authorization, the working group will initially focus on documenting the existing workflows used by clients and servers today.
Once complete, future use cases will be added if feasible.
The specification should favor designs that do not prevent future use cases and external considerations.

## Existing

- Anonymous access
- Various auth methods: token, basic
- Different types of credentials: passwords, tokens
- Different types of access: pull, push, delete
- Cross repository blob mount
- Attempting to use a token that has expired, received for a different repository, or a different access type
  - Performing a head request (to verify the absence) before performing a blob post
- Unscoped requests to `_catalog`
- HTTP redirects and CDN integration

## Future

- Authentication for extensions in the registry (actions other than push and pull)
- Authentication for global APIs (not scoped to a repository)
- Long running clients (registry mirroring, caches)
  - Timeout of stale credentials
  - Accessing multiple repositories
- Machine based authentication (spiffe or similar workload based attestation)
- Certificate authority
- Mutual TLS
- Fine grained authentication (individual tags within a repository)

## External Considerations

- UI authentication vs client tooling/CLI workflow
- Multi-tenant clients
- Artifact scanning and signing
- Encryption/decryption images/artifacts
