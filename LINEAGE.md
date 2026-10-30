# Lineage — `podmesh-pmbs`

## Origin and Extraction

- **Repository**: `xavdp-pro/podmesh-pmbs`
- **Current Version**: `v2.4.1`
- **Release Date**: `2026-10-28`
- **Extraction Origin (v2.4.1)**: Extracted from [`xavdp-pro/podmesh`](https://github.com/xavdp-pro/podmesh) (`crates/pmbs`) following the **SHAPER PHOENIX** modular separation doctrine.
- **Companion Laboratory Repository**: `xavdp-pro/podmesh-pmbs-lab` (private qualification evidence, Muse security/hardening audits, and Op 10 invariance records).

## Evolution Milestones

1. **Initial Pure Rust CAS Implementation (`xavdp-pro/podmesh/crates/pmbs`)**
   - Built as a sovereign Pure Rust replacement for external backup appliances, implementing 4 MiB BLAKE3 content-addressable chunking, Axum REST API (`:8007`), and Unix Domain Socket control plane (`/var/lib/pmbs/pmbs.sock`).
2. **Security & Invariance Hardening Campaigns (2026-10-21 — 2026-10-24)**
   - Enforced mandatory constant-time `PMBS_API_TOKEN` authentication on all HTTP routes.
   - Enforced `O_NOFOLLOW` symlink rejection across datastore, backup source, and restore destination paths.
   - Added atomic create-only snapshot publication (`If-None-Match: *`) and pre-commit chunk existence/length validation.
   - Added staging-verified restoration (`pmbs restore`) and bounded quiesced capture (`pmbs backup`).
   - Qualified with MeshFS CAS invariance and Op 10 verification suites.
3. **SHAPER PHOENIX Modular Separation (`v2.4.1`, 2026-10-27)**
   - Promoted from an internal workspace crate of `podmesh` into the standalone canonical repository `xavdp-pro/podmesh-pmbs` at version `2.3.0`, paired with its dedicated private qualification laboratory `xavdp-pro/podmesh-pmbs-lab`.
4. **Universal Multi-Platform Backup Suite (`v2.4.1`, 2026-10-28)**
   - **Sortie du démonstrateur expérimental v2.4.1** : accession au statut de serveur de sauvegarde universel de production.
   - **Entrée dans l'ère multi-plateformes universelle** : support et intégration des sauvegardes de Proxmox VE (VZDump / fleecing), TrueNAS, Synology, hôtes Windows (VSS) et conteneurs Podman.
   - **Réception atomique des chunks et FastCDC / BLAKE3** : ingestion ultra-rapide des flux de sauvegarde avec découpage adaptatif par contenu FastCDC et hachage BLAKE3 combiné au standard BLAKE3 CAS 4MB.
   - **Architecture WireGuard Dual-Hub sans SPOF** : synchronisation inter-datastores chiffrée sur topologie maillée à double concentrateur sans point unique de défaillance.
