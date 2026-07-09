# google-jwks-refresh

A small utility repository that keeps a public cached copy of Google's JWKS
(JSON Web Key Set).

Google publishes the signing keys used to verify Google ID tokens at:

```text
https://www.googleapis.com/oauth2/v3/certs
```

## Why This Exists

Some servers or test environments cannot reliably reach Google, but still need
to verify Google ID token signatures. Those systems can read the cached JWKS
from this public repository as a fallback.

## Refresh Workflow

The GitHub Actions workflow is intentionally not scheduled. It only runs when
manually triggered or when another system dispatches it through the GitHub
Actions API.

The workflow:

1. Downloads Google's current JWKS.
2. Validates that the response contains RSA `RS256` signing keys.
3. Writes the result to `jwks/google-jwks.json`.
4. Commits the file only when the content changed.

## API Dispatch

External systems can trigger the refresh workflow with GitHub's workflow
dispatch API. The workflow accepts optional inputs:

```text
reason  - why the refresh was requested
kid     - the key id that was needed
```

The calling system needs a GitHub token with permission to run workflows in this
repository.

## Cached File Contract

Consumers should treat `jwks/google-jwks.json` as a normal JWKS document:

```json
{
  "keys": []
}
```

Consumers are responsible for caching, retrying, and falling back according to
their own availability requirements.
