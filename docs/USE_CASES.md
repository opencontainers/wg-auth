# Use Cases

In designing spec changes for authentication and authorization, the working group will initially focus on documenting the existing workflows used by clients and servers today.
Once complete, future use cases will be added if feasible.
The specification should favor designs that do not prevent future use cases and external considerations.

## Existing

- Anonymous access
- Various auth methods (token, basic)
- Different types of credentials (passwords, tokens)
- Pull/push an image
- Cross repository blob mount
- Unscoped requests to `_catalog`
- CDN integration

## Future

- Authentication for extensions in the registry (actions other than push and pull)
- Authentication for global APIs (not scoped to a repository)
- Long running clients (registry mirroring, caches)
  - Timeout of stale credentials
  - Accessing multiple repositories
- HTTP redirection (when should a client allow/reject)
- Certificate authority
- Mutual TLS
- Fine grained authentication (individual tags within a repository)

## External Considerations

- UI authentication vs client tooling/CLI workflow
- Multi-tenant clients
- Artifact scanning and signing
- Encryption/decryption images/artifacts
