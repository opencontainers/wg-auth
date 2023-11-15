# Containerd

Containerd is a container runtime.  This document describes how it authenticates to remote registries.

## Authentication

- Authentication information is stored on a per host basis.
  - `service` and `realm` are assumed to be constant per host.
  - Individual `scope` values are added without any sort of merging (i.e. stored as both `repository:<repo>:pull` and `repository:<repo>:pull,push` instead of just `repository:<repo>:pull,push`).
  - This also applies to hosts that are the target of redirection.
- Requests are retried up to 5 times. Only status codes 401 (Unauthorized), 405 (Method Not Allowed), 408 (Request Timeout), and 429 (Too Many Requests) are retried.
- Multiple scopes are included in push or repository remounts
  - Sample push scopes
    - `repository:<repo>:pull`
    - `repository:<repo>:pull,push`
  - Sample remount scopes
    - `repository:<source_repo>:pull`
    - `repository:<target_repo>:pull`
    - `repository:<target_repo>:pull,push`

### Request

1. Build an url to the resource
2. Check per host cache for credentials and attach credentials if present
3. Make the request
4. If 401, use data from the challenge to try to get a token and add to cache on success
5. Got step 2
6. Retry up to 5 times

### Use Basic Authentication

- If user and secret are available, set header to `Basic base64(user:secret)`.
- Otherwise return an error.

### User Bearer Authentication

- If no secret is available, attempt anonymous access using `GET`
- Otherwise attempt `POST`
  - Fallback to `GET` on 405 (Method Not Allowed), 404 (Not Found), 401 (Unauthorized), and 400 (Bad Reqeust)

#### With GET

- Uses `realm` from the challenge as destination.
- Any 2xx is considered a success.
- Set `service` value from the challenge in the request query string.
- Set each `scope` as an individual parameters in the request query string.
- If a secret is specified, set Authorization header to `Basic base64(user:secret)`.
- Decode json response for `token`, `access_token`, `expires_in`, `issued_at` and `refresh_token`. The value of `access_token` is used over the value of `token`.

#### With POST

- Use a `POST` request to obtain a token. Uses `realm` from the challenge as destination.
- Any 2xx is considered a success.
- Content-Type is `application/x-www-form-url-encoded; charset=utf-8`.
- Set form fields:
  - Each `scope` as a space separated value.
  - `service` from the challenge
  - `client_id`
  - If no user is specified:
    - `grant_type`: `refresh_token`
    - `refresh_token`: the token value
  - Otherwise:
    - `grant_type`: `password`
    - `username`: username value
    - `password`: secret value
  - If refresh token is requested:
    - `access_type`: `offline`
- Decoded json response and verify an `access_token` has been specified

## References

- client.Pull  
<https://github.com/containerd/containerd/blob/main/client/pull.go#L43>
- Attempt remote call and start authentication if needed with retries <https://github.com/containerd/containerd/blob/main/remotes/docker/resolver.go#L579>
- Parse challenge and store details for host <https://github.com/containerd/containerd/blob/main/remotes/docker/authorizer.go#L143>
- Determine type of authentication <https://github.com/containerd/containerd/blob/main/remotes/docker/authorizer.go#L111>
- Build and execute token server requests <https://github.com/containerd/containerd/blob/main/remotes/docker/auth/fetch.go>
