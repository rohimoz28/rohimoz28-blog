# Membangun Backup Orchestrator Multi-Node dengan GitLab CI: Pendekatan Hybrid SSH dan Shell Executor

## TL;DR
GitLab CI jadi control plane tunggal untuk backup multi-node dengan workflow terstandar, observable, dan aman.

## WHY

### Problem yang Ingin Diselesaikan
Saat jumlah server bertambah, backup sering berubah jadi proses manual yang rapuh:
- Schedule tersebar di banyak host dan sulit diaudit.
- Proses backup antar server tidak konsisten (nama file, retention, logging).
- Kredensial rawan tercecer karena tersimpan lokal di script host.
- Troubleshooting lambat karena log tidak terpusat.

Pada architecture ini, ada 3 node target:
- **Node A**: server `103` (`ITCORP`) dengan eksekusi via SSH dari pipeline.
- **Node B**: server `133` (`SANBE PROD`) dengan shell executor lokal host.
- **Node C**: server `132` (`BINASANPRIMA`) dengan shell executor lokal host.

### Kenapa Solusi Ini Relevan
Pendekatan hybrid dipilih karena realitas infrastruktur tidak selalu homogen. Sebagian host paling aman diakses remote (SSH), sebagian lain lebih efisien dieksekusi langsung dari runner lokal. GitLab CI diposisikan sebagai **single orchestrator** agar kontrol, audit trail, dan observability tetap satu pintu.

## WHAT

### Gambaran Arsitektur
Komponen utama architecture:
1. **GitLab CI Pipeline** sebagai control plane (trigger schedule/manual, rules, artifact).
2. **Backup Script per Node/Project** sebagai execution unit (`database` dan/atau `filestore`).
3. **Runner per Node** untuk menyesuaikan mode eksekusi (remote SSH atau local shell).
4. **CI/CD Variables** sebagai source of truth untuk konfigurasi dan credential.
5. **Log Aggregation ke Artifact** untuk audit dan troubleshooting.

### Pola Eksekusi per Node
- **Node A (103)**: pipeline mengirim script via `ssh ... "bash -s"` lalu menjalankan backup di host target.
- **Node B (133)**: job langsung `bash scripts/133/...` karena runner berada di host target.
- **Node C (132)**: job langsung `bash scripts/132/...`, dengan fokus backup PostgreSQL host-level dan kompresi paralel.

### Standardisasi yang Diterapkan
- Validasi variabel wajib sebelum backup dimulai.
- Penamaan file backup berbasis timestamp/date agar traceable.
- Logging ganda: human-readable (`.log`) dan machine-readable (`.json`).
- Retention cleanup otomatis untuk menjaga kapasitas storage.
- Artifact `pipeline_logs/` untuk visibilitas hasil eksekusi.

## HOW

### Workflow End-to-End
1. Pipeline dipicu oleh **schedule** atau **manual run**.
2. Rules menentukan job mana yang aktif berdasarkan flag environment (`BACKUP_* == true`).
3. Job menjalankan script backup sesuai node:
   - Node A melalui SSH.
   - Node B/C secara lokal via shell executor.
4. Script melakukan validasi pre-check:
   - env var wajib,
   - container/path/database check,
   - optional anti-duplikasi backup harian (node tertentu).
5. Backup dieksekusi (pg_dump/tar/docker cp), lalu file dipindah ke backup host directory.
6. Log `.log` dan `.json` ditulis ke log directory node.
7. `after_script` mengumpulkan log ke `pipeline_logs/` (scp/cp).
8. GitLab menyimpan artifact untuk inspeksi pasca-eksekusi.
9. Cleanup retention menghapus backup lama sesuai policy.

### UML Sequence Diagram (PlantUML)

```plantuml
@startuml
title Backup Orchestrator Multi-Node Workflow
autonumber

actor "Scheduler/Manual Trigger" as Scheduler
participant "GitLab CI Pipeline" as CI
participant "Runner Node A (SSH Mode)" as RA
participant "Node A (103 / ITCORP)" as A
participant "Runner Node B (Local Shell)" as RB
participant "Node B (133 / SANBE)" as B
participant "Runner Node C (Local Shell)" as RC
participant "Node C (132 / BINASANPRIMA)" as C
participant "GitLab Artifact Storage" as Art

Scheduler -> CI: Trigger pipeline (schedule/web)
CI -> CI: Evaluate rules + BACKUP flags

alt Job untuk Node A aktif
    CI -> RA: Start job backup A
    RA -> A: SSH + inject env + run backup script
    A -> A: Validate env/container/db/path
    A -> A: Generate backup + write .log/.json + cleanup retention
    RA -> A: SCP log files
    RA -> Art: Upload pipeline_logs/
end

alt Job untuk Node B aktif
    CI -> RB: Start job backup B
    RB -> B: Execute local backup script
    B -> B: Validate env/container/db/path
    B -> B: Generate backup + write .log/.json + cleanup retention
    RB -> Art: Upload pipeline_logs/
end

alt Job untuk Node C aktif
    CI -> RC: Start job backup C
    RC -> C: Execute local postgres backup script
    C -> C: pg_dump parallel + compress + cleanup retention
    C -> C: Write .log/.json
    RC -> Art: Upload pipeline_logs/
end
@enduml
```

### Kenapa Opsi Ini Bagus Dijalankan
1. **Pragmatis untuk infrastruktur heterogen**: tidak memaksa semua node pakai satu model eksekusi.
2. **Operational consistency**: walau mode eksekusi beda, kontrol lifecycle backup tetap seragam dari pipeline.
3. **Security posture lebih baik**: credential disimpan di GitLab Variables, bukan hardcoded di repository.
4. **Observability bawaan**: log terstruktur + artifact mempermudah audit, post-mortem, dan alert integration.
5. **Mudah scale-out**: tambah node baru cukup mengikuti pola folder script + template job + variables.

## Catatan Implementasi untuk Jekyll
- Diagram menggunakan `PlantUML`.
- Untuk render di Jekyll, gunakan plugin/integrasi PlantUML (contoh lewat Kroki atau converter saat build).
- Setelah integrasi aktif, block ` ```plantuml ` di atas bisa dirender otomatis.

## Penutup
Architecture ini menempatkan GitLab CI sebagai pusat orkestrasi backup lintas node dengan design yang seimbang: fleksibel di layer eksekusi, tetapi ketat di layer governance. Hasilnya adalah backup process yang lebih konsisten, lebih mudah diaudit, dan lebih siap untuk scale.
