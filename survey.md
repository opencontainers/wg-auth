# Landscape Survey

This is a survey to document state-of-art in authentication mechanisms supported by image registry servers and clients.

## Registries

|Registry| Supported Authentication | References |
|---|---|---|
| dockerhub | | https://docs.docker. com/docker-hub/access-tokens/ |
| acr | | https://learn.microsoft.com/en-us/azure/container-registry/container-registry-repository-scoped-permissions |
| gcr | | https://cloud.google.com/docs/authentication/token-types |
| ecr | | https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html |
| ghcr | | https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry#authenticating-to-the-container-registry |
| zot | | https://zotregistry.io/latest/articles/authn-authz/ |

## Clients

|Client | Supported Authentication | Supported Registry | References |
|---|---|---|---|
| docker cli | | | https://docs.docker.com/engine/reference/commandline/login/ |
| oras | | | https://oras.land/docs/commands/oras_login/ |
| regclient | | | https://github.com/regclient/regclient/blob/main/docs/README.md |
| skopeo | | | https://github.com/containers/skopeo/blob/main/docs/skopeo-copy.1.md |
