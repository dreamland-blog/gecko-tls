# gecko-tls

`gecko-tls` is a Rust-powered Python networking library focused on browser-compatible TLS and HTTP behavior. The core implementation is written in Rust and exposed to Python through PyO3.

The intended shape of the project is:

```text
Python (gecko_tls)
  -> Rust (PyO3)
      -> HttpClient (TCP + TLS -> HTTP/1.1 and HTTP/2)
          -> BoringSSL backend for Chrome, OkHttp, and Cronet-style profiles
          -> NSS backend for Firefox-style profiles
      -> H3Client (QUIC/UDP -> HTTP/3)
          -> quinn + rustls + h3
```

## Goals

- Match browser TLS ClientHello behavior, including cipher suites, extension ordering, ALPN, GREASE, and certificate compression where applicable.
- Match HTTP/2 behavior, including SETTINGS frames, pseudo-header ordering, frame ordering, and request header ordering.
- Support HTTP/3 profiles through QUIC transport parameters and browser-style request construction.
- Expose a Python API that keeps high-level usage simple while keeping fingerprint parameters defined in Rust profiles.
- Release the Python GIL during blocking Rust network operations so Python callers can run concurrent requests.

## Planned Profiles

| Profile | TLS backend | HTTP/2 pseudo-header order | GREASE | HTTP/3 |
| --- | --- | --- | --- | --- |
| Firefox 151 | NSS or BoringSSL | `m,p,a,s` | No | Yes |
| Chrome Latest | BoringSSL | `m,a,s,p` | Yes | Yes |
| OkHttp 4 | BoringSSL | `m,p,a,s` | No | No |
| Cronet | BoringSSL | `m,a,s,p` | Yes | Yes |

## Planned Source Layout

```text
src/
  lib.rs          PyO3 module entry point
  client.rs       HTTP client, redirects, decompression, streaming responses
  error.rs        Rust error to Python exception mapping
  profiles/       Browser profile definitions
  tls/            TLS backend selection and configuration
  nss/            NSS and NSPR FFI bridge
  quic/           HTTP/3 client
  h2/             HTTP/2 fingerprint formatting
python/
  gecko_tls/      Python package wrapper
examples/         Usage examples
```

## Build Direction

The project is expected to build as a Python extension module with `maturin`:

```bash
maturin develop
```

Optional feature flags are expected for backend-specific support:

```bash
maturin develop --features nss
maturin develop --features http3
```

## Status

This repository currently contains the initial project description and structure plan. The Rust and Python implementation still needs to be added.
