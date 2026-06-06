# amp (lodestar-team fork)

A self-built fork of [Amp](https://thegraph.com/) — the blockchain-native
database — maintained at [`lodestar-team/amp`](https://github.com/lodestar-team/amp)
so that [engine.camp](https://engine.camp) builds and runs its indexing engine
from source rather than from a prebuilt binary.

> **What this is.** This repository is an independent fork of Edge & Node
> Ventures' Amp, taken at commit `a1937bf` (2025-12-12). It is **not affiliated
> with, sponsored by, or endorsed by Edge & Node Ventures or The Graph.** "Amp"
> is their name; the executables here keep the names `ampd`/`ampctl`/`ampsync`
> for drop-in compatibility, but this is the *lodestar-team fork*, versioned
> independently (see [Versioning](#versioning)). The Licensed Work remains under
> the Business Source License 1.1 — see [License](#license).

## Why this fork exists

camp serves a free, no-key public REST/SQL API over an Arbitrum One Amp indexer.
Running a prebuilt `ampd` we don't compile means we can't see, pin, or patch what
we're actually running. This fork gives us:

- **Provenance** — the binary self-reports its exact commit (`ampd --version`),
  built by us from source we can read.
- **Control** — the option to patch the engine for camp's needs (e.g. a future
  HyperSync extractor) in clearly-isolated crates.
- **A clean legal footing** — BUSL-1.1's Additional Use Grant permits camp's
  free, non-competitive production use (see [License](#license)).

This is a **frozen Dec-2025 snapshot** (a depth-1 clone, no upstream history).
The upstream source repository is no longer public at its former URL, so this
fork does not currently track an upstream branch.

## Build from source

Verified on Linux (Arch/Manjaro), June 2026.

### Prerequisites

| Tool | Why | Arch/Manjaro | Debian/Ubuntu |
|------|-----|--------------|---------------|
| Rust **1.91.0** | pinned by `rust-toolchain.toml` (auto-installed by rustup) | `rustup` | `rustup` |
| `protoc` | tonic/prost gRPC codegen | `pacman -S protobuf` | `apt install protobuf-compiler` |
| `cmake` | builds `snmalloc-sys` (default allocator) | `pacman -S cmake` | `apt install cmake` |
| `pkg-config`, C/C++ toolchain | native deps | `pacman -S pkgconf base-devel` | `apt install pkg-config build-essential` |
| `git` | build stamps the commit via `vergen` | (system) | (system) |

### Build

```sh
# rustup picks up the pinned 1.91.0 toolchain automatically
cargo build --release -p ampd -p ampctl

# binaries land in target/release/
./target/release/ampd --version      # e.g. "ampd v0.1.0"
```

To build without the bundled snmalloc allocator (skips the `cmake` requirement,
uses the system allocator — functionally identical):

```sh
cargo build --release -p ampd -p ampctl --no-default-features
```

A Nix flake is also provided (`flake.nix`): `nix build .#ampd`.

## Running

`ampd` is configured by a single TOML file (passed via `--config` or the
`AMP_CONFIG` env var) that points at a PostgreSQL metadata DB, a data directory
for Parquet output, a providers directory, and a manifests directory. A local
Postgres for development can be started with `docker-compose up -d`.

### Commands

| Command | Purpose |
|---------|---------|
| `ampd dev` | All-in-one: Flight + JSON Lines + Admin servers in one process (the single-box mode). Flags: `--flight-server`, `--jsonl-server`, `--admin-server` (all on if none given). |
| `ampd server` | Query servers only (`--flight-server`, `--jsonl-server`). |
| `ampd controller` | Controller + Admin API. |
| `ampd worker --node-id <id>` | Distributed extraction worker. |
| `ampd migrate` | Run metadata-DB migrations. |

Default ports (overridable in config): Arrow Flight `1602`, JSON Lines HTTP
`1603`, Admin API `1610`.

The JSON Lines server takes a raw SQL body and streams one JSON row per line:

```sh
curl -X POST http://localhost:1603 --data 'select * from "my_namespace/eth_rpc".logs limit 10'
```

`ampctl` manages the engine: `ampctl {manifest,provider,job,dataset,worker}`.
See `ampctl --help`.

## Versioning

This fork is versioned **independently of upstream Amp.** `v0.1.0` is the first
release cut from the cargopete fork (the Dec-2025 `a1937bf` baseline). The
running binary's version string comes from `git describe`, so a build at a
tagged commit reports that tag (e.g. `ampd v0.1.0`); an untagged build reports
the short commit hash.

## License

This work is licensed under the **Business Source License 1.1**. See
[LICENSE](LICENSE).

- **Licensor:** Edge & Node Ventures, Inc.
- **Change Date:** three years from publication of each version → **Apache 2.0**.
- **Additional Use Grant:** production use is permitted *provided it does not
  compete* with Edge & Node Ventures' offerings of the Licensed Work. Per the
  grant, a product offered to third parties **not on a paid basis**, or that does
  not significantly overlap with their offering, **is not competitive.** camp is
  a free, no-key public service → its production use is permitted under this
  grant, for as long as it stays free.

BUSL grants no trademark rights. This fork is not affiliated with Edge & Node
Ventures or The Graph.
