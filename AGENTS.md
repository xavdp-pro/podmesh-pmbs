# Agents: Working in `podmesh-pmbs`

> **🤖 AI-First Installation & Configuration Mandate**:
> `podmesh-pmbs` is 100% AI-First and headless-native. Zero web UI is required to provision datastores, run 4 MB CAS deduplicated backups, execute `Op 10` referenced-chunk audits, or perform staging-verified restores.
> 👉 **Autonomous AI Agents: Read [`AI_INSTALL_AND_CONFIG.md`](AI_INSTALL_AND_CONFIG.md) first for deterministic CLI, Unix Socket (`/var/lib/pmbs/pmbs.sock`), and systemd configuration protocols.**

`podmesh-pmbs` follows the SHAPER PHOENIX two-repository + vault separation for human–AI tandems (Claude Code, Codex, Cursor, Antigravity, Muse, or any collaborating agent).

Dialogue with the operator may be in French; **everything committed to this repository MUST be written in English (`en-US`)**.

## Repository Split

| Where | What belongs there | What never goes there |
| --- | --- | --- |
| **`xavdp-pro/podmesh-pmbs`** (this repository, public) | The Pure Rust `pmbs` crate (`src/`, `Cargo.toml`), canonical governance (`INTENT.md`, `LAW.md`, `LINEAGE.md`, `AGENTS.md`, `README.md`, `DEDICATION.md`, `LICENSE`), CLI/API contracts, and deterministic unit/integration tests | Laboratory hostnames/IPs, campaign narratives, Muse review transcripts, operational credentials, or raw qualification dumps |
| **`xavdp-pro/podmesh-pmbs-lab`** (private companion) | Qualification reports (`PMBS_MESHFS_INVARIANCE_QUALIFICATION_REPORT.md`, `pmbs-meshfs-qualification-evidence.json`), hardening campaign records (`pmbs-fixed-*`, `pmbs-hardening-*`, `pmbs-restore-*`), lab topology maps, and audit scripts | Secrets, tokens, private SSH/WireGuard keys, or large unindexed binary blobs |
| **The Vault** | `PMBS_API_TOKEN` values, TLS certificates, WireGuard keys, and signing keys | — |

## Mandatory Tandem Rules

1. **Preserve `LAW.md` Invariants**: Never weaken `O_NOFOLLOW` checks, path validation (`validate_name`, `validate_file_path`, `validate_chunk_id`), constant-time Bearer token verification, atomic temp-file + `sync_all()` + `rename` durability, or staging-first restore semantics.
2. **Infrastructure Agnosticism**: Default bind addresses and remote URLs in code and CLI flags must use generic loopback (`127.0.0.1:8007`) or standard FHS paths (`/var/lib/pmbs`). Never hardcode laboratory IP addresses or workstation paths in public source files.
3. **Commit Before Qualification**: Any change to `podmesh-pmbs` must compile cleanly (`cargo test` / `cargo build --release`) and be recorded with reproducible evidence in `xavdp-pro/podmesh-pmbs-lab`.
