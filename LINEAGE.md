# Lineage — `podmesh-pmbs`

## Origin and Extraction

- **Repository**: `xavdp-pro/podmesh-pmbs`
- **Extracted Version**: `v2.3.0`
- **Extraction Date**: `2026-10-27`
- **Upstream Source**: Extracted from [`xavdp-pro/podmesh`](https://github.com/xavdp-pro/podmesh) (`crates/pmbs`) following the **SHAPER PHOENIX** modular separation doctrine.
- **Companion Laboratory Repository**: `xavdp-pro/podmesh-pmbs-lab` (private qualification evidence, Muse security/hardening audits, and Op 10 invariance records).

## Evolution Milestones

1. **Initial Pure Rust CAS Implementation (`xavdp-pro/podmesh/crates/pmbs`)**
   - Built as a sovereign Pure Rust replacement for external backup appliances, implementing 4 MiB SHA-256 content-addressable chunking, Axum REST API (`:8007`), and Unix Domain Socket control plane (`/var/lib/pmbs/pmbs.sock`).
2. **Security & Invariance Hardening Campaigns (2026-10-21 — 2026-10-24)**
   - Enforced mandatory constant-time `PMBS_API_TOKEN` authentication on all HTTP routes.
   - Enforced `O_NOFOLLOW` symlink rejection across datastore, backup source, and restore destination paths.
   - Added atomic create-only snapshot publication (`If-None-Match: *`) and pre-commit chunk existence/length validation.
   - Added staging-verified restoration (`pmbs restore`) and bounded quiesced capture (`pmbs backup`).
   - Qualified with MeshFS CAS invariance and Op 10 verification suites.
3. **SHAPER PHOENIX Modular Separation (`v2.3.0`, 2026-10-27)**
   - Promoted from an internal workspace crate of `podmesh` into the standalone canonical repository `xavdp-pro/podmesh-pmbs` at version `2.3.0`, paired with its dedicated private qualification laboratory `xavdp-pro/podmesh-pmbs-lab`.
