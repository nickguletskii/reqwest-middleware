# reqwest-middleware

A crate implementing a wrapper around [reqwest](https://crates.io/crates/reqwest)
to allow for client middleware chains.

[![Crates.io](https://img.shields.io/crates/v/reqwest-middleware.svg)](https://crates.io/crates/reqwest-middleware)
[![Docs.rs](https://docs.rs/reqwest-middleware/badge.svg)](https://docs.rs/reqwest-middleware)
[![CI](https://github.com/TrueLayer/reqwest-middleware/workflows/CI/badge.svg)](https://github.com/TrueLayer/reqwest-middleware/actions)
[![Coverage Status](https://coveralls.io/repos/github/TrueLayer/reqwest-middleware/badge.svg?branch=main&t=YKhONc)](https://coveralls.io/github/TrueLayer/reqwest-middleware?branch=main)

This crate provides functionality for building and running middleware but no middleware
implementations. This repository also contains a couple of useful concrete middleware crates:

* [`reqwest-retry`](https://crates.io/crates/reqwest-retry): retry failed requests.
* [`reqwest-tracing`](https://crates.io/crates/reqwest-tracing):
  [`tracing`](https://crates.io/crates/tracing) integration, optional opentelemetry support.

## Overview

The `reqwest-middleware` client exposes the same interface as a plain `reqwest` client, but
`ClientBuilder` exposes functionality to attach middleware:

```toml
# Cargo.toml
# ...
[dependencies]
reqwest = "0.11"
reqwest-middleware = "0.1.1"
reqwest-retry = "0.1.1"
reqwest-tracing = "0.1.2"
tokio = { version = "1.12.0", features = ["macros", "rt-multi-thread"] }
```

```rust
use reqwest_middleware::{ClientBuilder, ClientWithMiddleware};
use reqwest_retry::{RetryTransientMiddleware, policies::ExponentialBackoff};
use reqwest_tracing::TracingMiddleware;

#[tokio::main]
async fn main() {
    // Retry up to 3 times with increasing intervals between attempts.
    let retry_policy = ExponentialBackoff::builder().build_with_max_retries(3);
    let client = ClientBuilder::new(reqwest::Client::new())
        // Trace HTTP requests. See the tracing crate to make use of these traces.
        .with(TracingMiddleware)
        // Retry failed requests.
        .with(RetryTransientMiddleware::new_with_policy(retry_policy))
        .build();
    run(client).await;
}

async fn run(client: ClientWithMiddleware) {
    client
        .get("https://truelayer.com")
        .header("foo", "bar")
        .send()
        .await
        .unwrap();
}
```

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
</sub>
