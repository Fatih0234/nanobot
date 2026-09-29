# How to Configure the OpenAI Codex Model Catalog

The WebUI model picker for the OpenAI Codex provider is populated from your
ChatGPT account's model endpoint. The catalog request is account-authenticated
and carries a `client_version` query parameter that the server uses when
deciding which models to expose — account-specific visibility, hidden flags,
and access rules still apply.

## How it works

- Discovery calls `GET https://chatgpt.com/backend-api/codex/models` with your
  OAuth token and `client_version`.
- nanobot sends `99.99.99` so the response is not restricted to the catalog of
  a pinned Codex CLI release. This is tested behavior for an internal endpoint
  that may change; the same convention has been used in an official Codex
  release workflow
  ([`rust-release-prepare.yml`](https://github.com/openai/codex/blob/d807d44a/.github/workflows/rust-release-prepare.yml)).
- The version affects model discovery only. It does not claim inference
  compatibility, and it is not part of the provider's public API contract.
- Results are cached for five minutes per account. When the endpoint cannot be
  reached, nanobot serves a cached list for up to 24 hours, then falls back to
  the built-in model list.

## When to override

If your account should only see the models a specific Codex client version
exposes — for example to pin discovery to a released client or to test older
catalogs — set `catalogClientVersion` on the `openaiCodex` provider:

```json
{
  "providers": {
    "openaiCodex": {
      "catalogClientVersion": "0.159.0"
    }
  }
}
```

The value must be a strict three-component version (`MAJOR.MINOR.PATCH`,
digits only). Omitting the field restores the sentinel. It is treated as a
literal string — `${VAR}` environment references are rejected during
configuration loading. This override is only available in the config file;
there is no WebUI setting for it.

## Caveats

- A model appearing in the ChatGPT catalog does not guarantee it is exposed to
  Codex or supported for inference; a listing is not proof a request will
  succeed.
- Cached results for a different version are never shared: changing
  `catalogClientVersion` fetches a fresh catalog for that version.
