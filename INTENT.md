# PodMesh Backup Server (`podmesh-pmbs`) — Intent

Status: Canonical Design Intent, v2.3.0 (2026-10-27).
Owner: Xavier de Poorter. Prepared under the SHAPER PHOENIX canonical structure.

## Purpose

`podmesh-pmbs` exists as an autonomous, high-performance deduplicated backup server written in Pure Rust. It provides sovereign, content-addressable storage (CAS) backup and restoration services for PodMesh universes, functional units, and Linux hosts without relying on external hypervisor lock-in or opaque binary blobs.

By decomposing backup workloads into fixed-bound 4 MiB SHA-256 content-addressable chunks and sealing snapshot manifests with deterministic cryptographic digests (Merkle-style manifest and per-file + per-chunk verification), `podmesh-pmbs` guarantees that backup artifacts can be stored once across generations, verified independently of live runtimes, and restored with strict cryptographic proof.

## Architectural Pillars

1. **4 MiB SHA-256 Content-Addressable Chunks (CAS)**
   Every regular file in a quiesced capture is sliced into up to 4 MiB (`4,194,304` bytes) segments. Each segment is addressed and stored by its lowercase hexadecimal SHA-256 digest in a two-level sharded directory tree (`.chunks/<first-2-hex>/<64-hex>.chunk`), eliminating duplicate storage across snapshots, containers, and hosts.

2. **Merkle Sealing & Cryptographic Manifest Integrity**
   Every snapshot manifest (`index.json`) binds:
   - Each logical relative path to its total byte length, full-file SHA-256 digest, ordered list of chunk references (`id`, `size`, `offset`), and POSIX metadata where applicable.
   - Only after every referenced chunk is durably synced (`fsync`) to disk is the snapshot manifest atomically published via `O_CREAT | O_EXCL` (`If-None-Match: *` create-only semantics).

3. **Op 10 Survival & Verification Guarantees**
   Under the PodMesh / SHAPER PHOENIX operational doctrine, a backup is not considered valid merely because an upload succeeded. `podmesh-pmbs` provides explicit **Op 10 verification guarantees**:
   - **Referenced-Chunk Cryptographic Audit (`pmbs verify`)**: Re-reads every chunk referenced by a snapshot from cold disk, recomputes its SHA-256 digest, verifies chunk byte lengths and contiguous file offsets, and reports any missing or corrupted chunk before disaster strikes.
   - **Authenticated Pre-Restore Verification (`pmbs restore`)**: Verifies all chunk digests and the complete reconstructed file SHA-256 digest in a staging directory before atomically publishing restored files into an empty, non-symlink destination.

## Boundaries and Explicit Non-Claims

- `podmesh-pmbs` captures explicitly quiesced directory trees (`--quiesced-input`) and verifies chunk/file integrity on disk.
- Referenced-chunk verification (`pmbs verify`) proves storage integrity of the backup repository; full Op 10 disaster recovery certification additionally requires restoring the snapshot into an isolated environment and booting/validating the workload (tracked in `xavdp-pro/podmesh-pmbs-lab`).
- Network exposure requires an authenticated Bearer token (`PMBS_API_TOKEN`, 32–256 printable ASCII bytes, constant-time comparison) over loopback or an authenticated TLS/WireGuard boundary.
