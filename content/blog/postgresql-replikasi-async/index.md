+++
title = 'PostgreSQL Replication'
date = '2025-11-14T14:09:18+07:00'
draft = false
description = 'Dummy project untuk replikasi PostgreSQL'
categories= ['PostgreSQL']
tags = ['postgresql']
+++

## PostgreSQL Replication
Kalo kamu membaca artikel ini, artinya kamu sudah menganggap replikasi database adalah sesuatu yang krusial.
Beruntung, PostgreSQL secara default mendukung replikasi dengan streaming data Asynchronous dan Synchronous.

## Asynchronous & Synchronous
PostgreSQL mendukung 2 jenis streaming data pada proses replikasi. Async dan Sync.

### Asynchronous Replication
Pada replikasi Asinkron, commit transaksi di server Primary dikonfirmasi berhasil ke klien segera setelah
data Write-Ahead Log (WAL) ditulis ke disk Primary.

Server Primary TIDAK menunggu konfirmasi apa pun dari server Standby bahwa mereka telah menerima 
atau memproses WAL tersebut.

### Synchronous Replication
Pada replikasi Sinkron, commit transaksi di server Primary dikonfirmasi berhasil ke klien hanya 
setelah Primary menerima konfirmasi dari server Standby bahwa mereka telah menerima WAL.

Tujuan utama adalah jaminan data dan memastikan RPO nol (tidak ada kehilangan data) dalam skenario failover.

|      | Async                                                                                | Sync                                                                                    |
|------|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| Pros | - Kinerja Tinggi<br/> - Paling Umum (Diterapkan pada banyak case)                    | - Integritas Data Maksimal<br/> - Konsistensi Tinggi                                    |
| Cons | - Potensi Kehilangan Data <br/> - Tidak menjamin RPO (Recovery Point Objective) nol  | - Kinerja Lebih Rendah <br/> - IRL, Membutuhkan setidaknya satu Standby yang berfungsi  |

Memilih tipe streaming data pada proses replikasi database akan sangat bergantung dengan kebutuhan bisnis yang dihadapi.

## Dummy Project
Untuk dummy project bisa dicek di repo [Github](https://github.com/rohimoz28/postgresql-async-replication).
Pada project tersebut, membuat 1 aplikasi API menggunakan Flask dan 2 database (Master & Replica) yang semuanya
menggunakan docker.

## Monitoring
Monitoring terhadap proses replikasi database yang berjalan amatlah penting demi menjaga integritas data.
Pada dummy project, sudah di sertakan query monitoring minimal yang diperlukan dan dijalankan di database master.

Terimakasih!
