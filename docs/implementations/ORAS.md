# ORAS

This document describes how `ORAS` authenticates with a remote registry. Error handling is omitted for simplicity.

## Authenticate

- Request any registry API endpoint `/v2/<api>`
- Try to load token for the registry from cache
- If cache hit:
  - If the scheme of the cached token is `"Basic"`:
    - Get `<cached-token>`
    - Set the `Authorization` header to `"Basic <cached-token>"`
    - Request `/v2/<api>` with the `Authorization` header
  - If the scheme of the cached token is `"Bearer"`:
    - Get `<cached-token>` for the built-in known scopes (e.g., the built-in known scopes for `PUT "/v2/<repo>/manifests/<reference>"` is `"repository:<repo>:pull,push"`)
    - Set the `Authorization` header to `"Bearer <cached-token>"`
    - Request `/v2/<api>` with the `Authorization` header
- If receive `401 Unauthorized`, parse the `Www-Authenticate` header in the response to get `<scheme>`, `<realm>`, `<service>` and `<scope>`
- If `<scheme>` is `"Basic"`:
  - [Use Basic Auth](#use-basic-auth)
- If `<scheme>` is `"Bearer"`:
  - [Use Bearer Auth](#use-bearer-auth)

### Use Basic Auth

- Read `<username>` and `<password>` from user input or configured credentials stores
- Store base64-encoded `<credentials>` into cache
  - `<credentials> = base64Encode(<username>:<password>)`
- Set the `Authorization` header to `"Basic <credentials>"`
- Request `/v2/<api>` with the `Authorization` header

### Use Bearer Auth

- Merge the challenged `<scope>` with the built-in known scopes to get `<result-scopes>`
  - For example, the challenged is `"repository:<repo>:pull,push"` and the built-in known scopes is `"repository:<repo>:delete,pull"`, the merged scopes will be `"repository:<repo>:delete,pull,push"`
- If `<result-scopes>` is different than the built-in known scopes:
  - Try to load token again for `<result-scopes>` from cache
  - If cache hit:
    - Set the `Authorization` header to `"Bearer <cached-token>"`
    - Request `/v2/<api>` with the `Authorization` header
- If `<result-scopes>` is the same as the built-in known scopes, or cache missed:
  - Read `<realm>` `<username>`, `<password>`, `<identity-token>` and `<registry-token>` from user input or configured credentials stores
  - If `<registry-token>` is not empty:
    - Store `<registry-token>` for `<result-scopes>` into cache
    - Set the `Authorization` header to `"Bearer <registry-token>"`
    - Request `/v2/<api>` with the `Authorization` header
  - If `<registry-token>`, `<username>`, `<password>` and `<identity-token>` are all empty:
    - [Fetch Distribution Token](#fetch-distribution-token)
  - If `<identity-token>` is empty and user-configured `ForceAttemptOAuth2` is `false`:
    - [Fetch Distribution Token](#fetch-distribution-token)
  - Otherwise:
    - [Fetch OAuth2 Token](#fetch-oauth2-token)

#### Fetch Distribution Token

- If one of `<username>` and `<password>` is not empty:
  - Set the `Authorization` header to `"Basic base64Encode(<username>:<password>)"`
- If `<username>` and `<password>` are both empty:
  - Don't set the `Authorization` header and it means to fetch an anonymous token
- Issue a `GET` request to the challenged `<realm>` URL with the following query parameters:
  - `service`: the challenged `<service>`
  - `scope`: the `<result-scopes>` from merging the built-in known scopes and the challenged `<scope>`
- Parse the response body to get `<access_token>` and `<token>`
- If `<access_token>` is not empty:
  - Store `<access_token>` for `<result-scopes>` into cache
  - Set the `Authorization` header to `Bearer <access_token>`
  - Request `/v2/<api>` with the `Authorization` header
- If `<access_token>` is empty but `<token>` is not empty:
  - Store `<token>` for `<result-scopes>` into cache
  - Set the `Authorization` header to `"Bearer <token>"`
  - Request `/v2/<api>` with the `Authorization` header

#### Fetch OAuth2 Token

- If `<identity-token>` is not empty, set the following form values:
  - `grant_type`: the `"refresh_token"` literal
  - `refresh_token`: the `<identity-token>`
- If `<identity-token>` is empty while both `<username>` and `<password>` are not empty, set the following form values:
  - `grant_type`: the `"password"` literal
  - `username` the `<username>`
  - `password`: the `<password>`
- Continue to set the following form values:
  - `service`: the challenged `<service>`
  - `client_id`: `"oras-go"`
  - `scope`: the `<result-scopes>` from merging the built-in known scopes and the challenged `<scope>`
- Set the `Content-Type` header to `"application/x-www-form-urlencoded"`
- Issue a `POST` request to the challenged `<realm>` URL with the URL-encoded form in the body
- Parse the response body to get `<access_token>`
- Store `<access_token>` for `<result-scopes>` into cache  
- Set the `Authorization` header to `"Bearer <access_token>"`
- Request `/v2/<api>` with the `Authorization` header

## "Log in"

- Read `<username>` from user input
- If `<username>` is not empty:
  - Read `<password>`
- If `<username>` is empty:
  - Read `<identity-token>`
- Issue a `GET` request to the `/v2/` endpoint using `<username>` and `<password>` or `<identity-token>` through the [authentication workflow](#authenticate)
- If receive `200 OK`:
  - Save `<username>` and `<password>`, or `<identity-token>` into configured credentials stores for the registry
- Otherwise:
  - Don't save `<username>` and `<password>`, or `<identity-token>`

## "Log out"

- Remove the credentials of the registry from configured credentials stores

## Code References

- Authenticate: [`auth.Client.Do`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L156-L266)
- Use basic auth: [`auth.Client.Do`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L204-L215) and [`auth.Client.fetchBasicAuth`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L269C1-L283)
- Use bearer auth: [`auth.Client.Do`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L216-L258) and [`auth.Client.fetchBearerToken`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L285C1-L298)
- Fetch distribution token: [`auth.Client.fetchDistributionToken`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L300-L350)
- Fetch OAuth2 token: [`auth.Client.fetchOAuth2Token`](https://github.com/oras-project/oras-go/blob/v2.3.1/registry/remote/auth/client.go#L352-L403)
- "Log in": [`credentials.Login`](https://github.com/oras-project/oras-go/blob/062ed0e058f87a9661dfb5db41e8e72f74dfffe2/registry/remote/credentials/registry.go#L31-L60)
- "Log out": [`credentials.Logout`](https://github.com/oras-project/oras-go/blob/062ed0e058f87a9661dfb5db41e8e72f74dfffe2/registry/remote/credentials/registry.go#L62-L69)
