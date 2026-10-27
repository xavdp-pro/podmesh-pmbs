# AI-First Installation, Datastore Provisioning & Zero-Transit Backup Protocol — `podmesh-pmbs` (v2.3.0)

> **Core Directive for Autonomous AI Agents**:
> `podmesh-pmbs` (PodMesh Backup Server) is a **100% AI-First, headless deduplicated backup server in Pure Rust** (~8.9 MB static binary, < 15 MB RSS under load).
> **Zero web interface is required or used** to configure datastores, execute backups, run `Op 10` referenced-chunk audits, perform staging-verified restores, or synchronize across sites.
> Every operation emits deterministic JSON (`stdout` / `stderr`) and is controlled via the `pmbs` CLI, Unix Domain Socket (`/var/lib/pmbs/pmbs.sock`), or authenticated loopback REST API (`127.0.0.1:8007`).

---

## 1. Mandatory Invariants for AI Agents (`LAW.md`)

1. **Mandatory Authentication (`PMBS_API_TOKEN`)**: The daemon refuses to start (fail-closed) if `PMBS_API_TOKEN` is unset or shorter than 32 characters.
2. **Symlink & Path Hardening (`O_NOFOLLOW`)**: `pmbs backup` and `pmbs restore` reject symlinks in any ancestor or entry path and enforce `0700` directory / `0600` file permissions.
3. **Create-Only Snapshot Immutability**: Snapshots are published atomically via `create_new` + `sync_all()` + `rename` after all 4 MB SHA-256 CAS chunks are verified on disk. Existing snapshot IDs cannot be overwritten.
4. **Staging-First Restore**: `pmbs restore` requires a non-existent destination directory, verifies every chunk's SHA-256 before writing, and leaves no partial output on error.
5. **Rule 38 Bounded Retry ($k \le 2$)**: If a backup, verify, or restore operation returns `"ok": false`, the AI agent may attempt at most $k \le 2$ deterministic fixes before halting fail-closed.

---

## 2. Step-by-Step AI Installation

### 2.1 Build or Install the `pmbs` Binary

From the `podmesh-pmbs` repository root:

```bash
cargo build --release
install -m 0755 target/release/pmbs /usr/local/bin/pmbs
pmbs --version
```

### 2.2 Provision Directories & Secret Token

Run on the target node as `root`:

```bash
install -d -m 0700 /var/lib/pmbs
install -d -m 0700 /var/lib/pmbs/datastores
install -d -m 0750 /etc/pmbs

# Generate a cryptographically strong 64-hex-character (32-byte) API token if not present
if [ ! -f /etc/pmbs/pmbs.env ]; then
  TOKEN="$(openssl rand -hex 32)"
  cat <<EOF > /etc/pmbs/pmbs.env
PMBS_API_TOKEN=${TOKEN}
PMBS_BIND=127.0.0.1
PMBS_PORT=8007
PMBS_DATA_DIR=/var/lib/pmbs
PMBS_SOCKET=/var/lib/pmbs/pmbs.sock
RUST_LOG=info
EOF
  chmod 0600 /etc/pmbs/pmbs.env
fi
```

### 2.3 Systemd Service (`/etc/systemd/system/pmbs.service`)

```ini
[Unit]
Description=PodMesh Backup Server (PMBS) Pure Rust Daemon v2.3.0
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
EnvironmentFile=/etc/pmbs/pmbs.env
UMask=0077
ExecStart=/usr/local/bin/pmbs daemon \
  --bind ${PMBS_BIND} \
  --port ${PMBS_PORT} \
  --data-dir ${PMBS_DATA_DIR} \
  --socket ${PMBS_SOCKET}
Restart=on-failure
RestartSec=2

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
systemctl daemon-reload
systemctl enable --now pmbs.service
```

---

## 3. AI-Driven Operations & JSON Contracts

Before running authenticated CLI commands (`datastore`, `backup`, `verify`, `restore`, `sync`), load the environment token into the agent shell:

```bash
set -a && source /etc/pmbs/pmbs.env && set +a
```

### 3.1 Liveness & Status Probes

```bash
# Fast IPC ping via Unix Domain Socket (/var/lib/pmbs/pmbs.sock)
pmbs ping

# Query server status and active datastores
pmbs status
```

### 3.2 Datastore Provisioning

```bash
# Create a dedicated datastore (e.g. for Proxmox VE or MeshFS workloads)
pmbs datastore create pve-meshfs-store --remote http://127.0.0.1:8007

# List all datastores and deduplication metrics
pmbs datastore list --remote http://127.0.0.1:8007
```

### 3.3 Creating an Immutable 4 MB CAS Backup (`pmbs backup`)

> **Important**: `<SOURCE>` must be a real directory (not a symlink or single file), and `--quiesced-input` confirms writers to that directory are paused/quiesced.

```bash
pmbs backup /mnt/pve/shared-meshfs/vm-9501-quiesced \
  --quiesced-input \
  --datastore pve-meshfs-store \
  --backup-type vm \
  --snapshot vm-9501-20261028T000000Z \
  --remote http://127.0.0.1:8007 \
  --max-bytes 107374182400 \
  --max-files 10000
```

Expected JSON output on `stdout` (`exit 0`):

```json
{
  "ok": true,
  "datastore": "pve-meshfs-store",
  "backupType": "vm",
  "snapshot": "vm-9501-20261028T000000Z",
  "fileCount": 2,
  "bytes": 16777815,
  "chunks": 5,
  "manifestSha256": "..."
}
```

### 3.4 Referenced-Chunk Cryptographic Audit — `Op 10` (`pmbs verify`)

Audits every referenced 4 MB chunk in the snapshot manifest against its SHA-256 digest on disk in $\mathcal{O}(1)$ manifest traversal:

```bash
pmbs verify \
  --datastore pve-meshfs-store \
  --snapshot vm-9501-20261028T000000Z
```

Expected JSON output:

```json
{
  "status": "VERIFIED",
  "datastore": "pve-meshfs-store",
  "snapshotsAudited": 1,
  "chunksAudited": 5,
  "corruptedChunks": 0,
  "missingChunks": 0,
  "integrityVerified": true
}
```

### 3.5 Staging-Verified Restore (`pmbs restore`)

> **Important**: `<DESTINATION>` must NOT exist prior to calling `pmbs restore`. PMBS creates it atomically with mode `0700` and verifies every chunk hash before writing.

```bash
pmbs restore /var/tmp/restore-vm-9501-staging \
  --datastore pve-meshfs-store \
  --backup-type vm \
  --snapshot vm-9501-20261028T000000Z \
  --remote http://127.0.0.1:8007
```

Expected JSON output:

```json
{
  "ok": true,
  "destination": "/var/tmp/restore-vm-9501-staging",
  "restoredFiles": 2,
  "restoredBytes": 16777815,
  "verifiedChunks": 5
}
```

### 3.6 Inter-Site Differential CAS Replication (`pmbs sync`)

Synchronize missing 4 MB CAS chunks and snapshot manifests from a remote PMBS node over WireGuard or HTTPS:

```bash
pmbs sync http://10.10.10.210:8007 \
  --remote-datastore pve-meshfs-store \
  --target-datastore pve-meshfs-store \
  --remote http://127.0.0.1:8007
```
