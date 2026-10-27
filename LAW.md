# PodMesh Backup Server (`podmesh-pmbs`) — Invariant Law

Status: Binding Architectural Invariants, v2.3.0 (2026-10-27).
Scope: All code, CLI commands, HTTP REST endpoints, and local IPC handlers in `podmesh-pmbs`.

## Core Invariants

### LAW-1: Bit-for-Bit Content and POSIX Metadata Preservation
1. Every captured file's byte stream MUST be preserved and reconstructed bit-for-bit identically to the source at capture time.
2. Logical relative paths, exact byte sizes, chunk offsets, and recorded POSIX metadata attributes MUST be preserved deterministically without locale or platform mangling.
3. Symlinks, device nodes, FIFOs, sockets, and path traversal components (`..`, `.`, control characters, backslashes, colons) in source or manifest paths MUST be explicitly rejected rather than silently followed or skipped. Every file open on storage and source paths uses `O_NOFOLLOW`.

### LAW-2: Cryptographic Seal Verification Before Restore
1. No chunk may enter the datastore unless its computed SHA-256 digest matches its 64-character lowercase hex identifier.
2. During restoration (`pmbs restore`), every chunk fetched from the server MUST be verified against its expected `ChunkRef` SHA-256 digest and exact byte length before assembly.
3. The assembled file MUST be verified against the manifest's full-file SHA-256 `digest` and `size` inside a temporary staging directory (`.pmbs-restore-staging-*` with mode `0700`) before moving into the final target location. If any chunk or file digest mismatches, restoration MUST abort and clean up staging artifacts without leaving partial output.

### LAW-3: Immutable Snapshot Retention (Create-Only Publication)
1. Published snapshots are immutable records. Creating a snapshot via CLI (`pmbs backup`) or REST (`POST /api/v1/datastores/:datastore/snapshots` with `If-None-Match: *`) MUST use atomic create-only publication (`create_new(true)` / `O_CREAT | O_EXCL` after `sync_all()`).
2. Attempting to overwrite an existing `(backup_type, snapshot_id)` tuple under create-only semantics MUST fail with `412 Precondition Failed` (or `AlreadyExists`), never silently mutating historical backup manifests.
3. Every chunk referenced by a manifest MUST already exist on disk and match its expected byte length before the manifest is committed.

### LAW-4: Zero Silent Truncation
1. Both backup capture and restore enforce explicit, bounded resource limits (`--max-bytes`, `--max-files`, `--max-file-bytes`, `--max-seconds`, and HTTP `CHUNK_SIZE` body limits).
2. If a file grows or shrinks during capture, if a reader returns fewer or more bytes than declared in file metadata, or if any quota or deadline is exceeded, `podmesh-pmbs` MUST fail the operation immediately with a structured error.
3. Under no circumstance may `podmesh-pmbs` emit a partial snapshot or truncated file while reporting success (`ok: true` or exit code `0`).

### LAW-5: Constant-Time Authentication & Symlink-Free Storage
1. Every HTTP request to `podmesh-pmbs` MUST present `Authorization: Bearer <PMBS_API_TOKEN>`, validated in constant time against a token of 32 to 256 printable non-whitespace ASCII bytes. Missing or invalid server token configuration returns `503 Service Unavailable`.
2. All datastore directories, chunk shards, and snapshot directories MUST be verified free of symlinks and created with owner-only permissions (`0700` for directories, `0600` for files).
