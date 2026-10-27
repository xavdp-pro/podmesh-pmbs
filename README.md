# PodMesh Backup Server (`podmesh-pmbs`) — v2.3.0

[**🤖 AI-First Install & Config**](AI_INSTALL_AND_CONFIG.md) · [Intent](INTENT.md) · [Law](LAW.md) · [Lineage](LINEAGE.md) · [Agents](AGENTS.md)

**PodMesh Backup Server (`pmbs`)** is an autonomous, high-performance deduplicated backup server and CLI written in Pure Rust under the **SHAPER PHOENIX** architecture. It provides 4 MiB SHA-256 Content-Addressable Storage (CAS), atomic Merkle-sealed snapshot manifests, referenced-chunk integrity auditing (`Op 10` verification), and staging-verified restoration — **100% AI-First and headless without any web UI dependency** (see **[`AI_INSTALL_AND_CONFIG.md`](AI_INSTALL_AND_CONFIG.md)**).

---

## Key Capabilities

- **4 MiB SHA-256 CAS Deduplication**: Splits regular files into canonical 4 MiB (`4,194,304` byte) chunks addressed by lowercase hex SHA-256 digests. Identical chunks across snapshots and workloads are stored once.
- **Atomic & Symlink-Safe Durability**: Every chunk and manifest write uses `O_NOFOLLOW`, owner-only permissions (`0700` directories, `0600` files), temporary staging files with `fsync` (`sync_all()`), and atomic POSIX `rename` / `create_new` (`O_CREAT | O_EXCL`).
- **Cryptographic Pre-Commit & Pre-Restore Verification**:
  - Server refuses snapshot publication unless all referenced chunks exist on disk with exact matching byte sizes.
  - Client restore (`pmbs restore`) verifies every chunk's SHA-256 digest and the full reconstructed file's SHA-256 digest in a staging directory before publishing to the target directory.
- **Dual Control Plane**:
  - **HTTP REST API** (`127.0.0.1:8007` by default) protected by constant-time `PMBS_API_TOKEN` Bearer authentication.
  - **Unix Domain Socket IPC** (`/var/lib/pmbs/pmbs.sock`, mode `0600`) for low-latency local node supervision (`PING`, `STATUS`, `LIST_DATASTORES`, `VERIFY`).

---

## Datastore Filesystem Layout

By default, `pmbs daemon` manages datastores under `/var/lib/pmbs` (configurable via `--data-dir`):

```text
/var/lib/pmbs/
├── pmbs.sock                                      # Local Unix Domain Socket (0600)
└── datastore-default/                             # Default Datastore (0700)
    ├── .chunks/                                   # Sharded 4 MiB SHA-256 CAS Pool
    │   ├── 00/
    │   │   └── 00a1b2...f9.chunk                  # Raw chunk bytes named by 64-hex SHA-256
    │   ├── ...
    │   └── ff/
    └── snapshots/                                 # Immutable Snapshot Manifests
        └── <backup_type>/                         # e.g. container, vm, host, meshfs
            └── <snapshot_id>/
                └── index.json                     # Atomic BackupManifest (0600)
```

---

## Building from Source

```bash
cargo build --release
./target/release/pmbs --version
# pmbs 2.3.0
```

---

## Authentication (`PMBS_API_TOKEN`)

All HTTP API operations require the `PMBS_API_TOKEN` environment variable to be set on both the daemon and CLI client. The token must be **32 to 256 printable non-whitespace ASCII characters**:

```bash
export PMBS_API_TOKEN="$(head -c 32 /dev/urandom | base64 | tr -d '=+/ \n' | head -c 48)"
```

---

## CLI Usage (`pmbs`)

### 1. Start the Daemon
```bash
pmbs daemon \
  --bind 127.0.0.1 \
  --port 8007 \
  --data-dir /var/lib/pmbs \
  --socket /var/lib/pmbs/pmbs.sock
```

### 2. Health & Status Check
```bash
# Via HTTP REST API
pmbs ping --remote http://127.0.0.1:8007
pmbs status --remote http://127.0.0.1:8007

# Via local Unix Domain Socket
pmbs status --socket /var/lib/pmbs/pmbs.sock
```

### 3. Manage Datastores
```bash
pmbs datastore list --remote http://127.0.0.1:8007
pmbs datastore create sovereign-store --remote http://127.0.0.1:8007
```

### 4. Capture a Quiesced Backup Snapshot
```bash
pmbs backup /var/lib/podmesh/quiesced-volume \
  --quiesced-input \
  --datastore datastore-default \
  --backup-type container \
  --snapshot snap-20261027-001 \
  --max-bytes 10737418240 \
  --max-files 10000 \
  --remote http://127.0.0.1:8007
```

### 5. Audit Referenced Chunks (`Op 10` Storage Verification)
```bash
pmbs verify \
  --datastore datastore-default \
  --snapshot snap-20261027-001 \
  --remote http://127.0.0.1:8007
```

### 6. Restore and Cryptographically Verify a Snapshot
```bash
pmbs restore /var/lib/podmesh/restored-volume \
  --datastore datastore-default \
  --backup-type container \
  --snapshot snap-20261027-001 \
  --max-bytes 10737418240 \
  --remote http://127.0.0.1:8007
```

---

## HTTP REST API Endpoints

All endpoints require header `Authorization: Bearer <PMBS_API_TOKEN>`:

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/`, `/status`, `/api/v1/status` | Server version (`2.3.0`), active datastores, and engine status |
| `GET` | `/api/v1/datastores` | List all initialized datastores |
| `POST` | `/api/v1/datastores` | Create a new datastore (`{"name": "<datastore>"}`) |
| `POST` | `/api/v1/datastores/:datastore/chunks` | Upload a raw chunk (`<= 4 MiB`); returns `{"ok": true, "chunkId": "<sha256>"}` |
| `GET` | `/api/v1/datastores/:datastore/chunks/:chunk_id` | Download raw bytes of a 64-hex SHA-256 chunk |
| `GET` | `/api/v1/datastores/:datastore/snapshots` | List all `BackupManifest` records in the datastore |
| `POST` | `/api/v1/datastores/:datastore/snapshots` | Publish a `BackupManifest` (supports `If-None-Match: *` create-only header) |
| `POST` | `/api/v1/datastores/:datastore/verify` | Verify all chunks referenced by `{"snapshotId": "..."}` (or all snapshots if omitted) |

---

## Companion Repository & Governance

- **Canonical Governance**: See [`INTENT.md`](INTENT.md), [`LAW.md`](LAW.md), [`LINEAGE.md`](LINEAGE.md), and [`AGENTS.md`](AGENTS.md).
- **Private Qualification Lab**: `xavdp-pro/podmesh-pmbs-lab`
- **License**: Apache-2.0 (see [`LICENSE`](LICENSE) and [`DEDICATION.md`](DEDICATION.md)).
