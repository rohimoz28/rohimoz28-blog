+++
title = 'Shell Script Untuk Backup Otomatis'
date = '2025-03-28T17:50:54+07:00'
draft = false
description = 'Backup aplikasi adalah sesuatu yang wajib dilakukan. Jika rutin dilakukan, ada baiknya kita buat menjadi otomatis.'
categories= ['Database']
tags = ['postgresql', 'odoo']
+++

## Latar Belakang Masalah
Sebagai seorang programmer Odoo, kita wajib untuk melakukan backup rutin aplikasi.
Aplikasi Odoo bisa di backup melalui frontend. Namun, ketika harus dilakukan rutin, adakalanya saya lupa
untuk melakukan backup. Dikarenakan load pekerjaan yang sedang banyak, atau memang lupa saja :)

Maka dari itu, diperlukan suatu mekanisme backup otomatis. Dan ketika proses backup selesai, entah itu berhasil atau gagal,
maka akan mengirimkan notifikasi melalui webhook discord.

Namun, kali ini, saya hanya akan melakukan backup terhadap database-nya saja. Dikarenakan
untuk kebutuhannya hanya diperlukan backup transaksi selama 7 hari terakhir.

## Tech Yang Digunakan
1. Docker
2. Crontab
3. Shell Script
4. Webhook
5. Linux

## Shell Script

Biasanya saya membuat sebuah folder khusus untuk semua shell script. Misalkan saya akan buat path:
`mkdir -p /home/rohimoz28/script/auto_backup.sh`

```shell
#!/bin/bash

# Define variables
BACKUP_DIR="path_to_backup_dir"
DB_NAME="your_database_name"
DB_USER="your_user_database"
DB_PASSWORD="your_pass_database"
DB_HOST="your_host_name"
DISCORD_WEBHOOK_URL="your_discord_webhook_url)"
TODAY=$(date +"%Y-%m-%d")
TIME_NOW=$(date +"%Y-%m-%d %H:%M:%S")
BACKUP_FILE_TODAY="${BACKUP_DIR}/auto_backup_sanbe_hrms.dump"
BACKUP_GZ_TODAY="${BACKUP_FILE_TODAY}.gz"

# Define a function to send a message to Discord
send_discord_notification() {
    local message=$1

    # Construct payload
    local payload=$(
        cat <<EOF
    {
              "content": "$message"
      }
EOF
    )

    # Send POST request to Discord Webhook
    curl -H "Content-Type: application/json" -X POST -d "$payload" $DISCORD_WEBHOOK_URL
}

#1 Check if backup directory exists, if not create it
if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
    if [ $? -ne 0 ]; then
        send_discord_notification "STATUS: Failed\nDB: $DB_NAME\nTIME: $TIME_NOW\nError: Could not create backup directory: $BACKUP_DIR. Check your permissions."
        echo "Error: Could not create backup directory: $BACKUP_DIR. Check your permissions."
        exit 1
    fi
    echo "Backup directory created: $BACKUP_DIR"
fi

#2 Export the PostgreSQL password
export PGPASSWORD="$DB_PASSWORD"

#3 Create today's backup as .dump format
if ! pg_dump -U "$DB_USER" -h "$DB_HOST" -F c -b -v -f "$BACKUP_FILE_TODAY" "$DB_NAME"; then
    send_discord_notification "STATUS: Failed\nDB: $DB_NAME\nTIME: $TIME_NOW\nError: Failed to create dump backup for $DB_NAME!"
    echo "Failed to create dump backup!"
    exit 1
else
    echo "Created today's dump backup: $BACKUP_FILE_TODAY"
fi

#4 Check if the dump backup file was created before compressing
if [ ! -f "$BACKUP_FILE_TODAY" ]; then
    send_discord_notification "STATUS: Failed\nDB: $DB_NAME\nTIME: $TIME_NOW\nError: Dump backup file not found for compression: $BACKUP_FILE_TODAY"
    echo "Dump backup file not found for compression: $BACKUP_FILE_TODAY"
    exit 1
else
    # Compress the dump file into a gzipped file
    if ! gzip "$BACKUP_FILE_TODAY"; then
        send_discord_notification "STATUS: Failed\nDB: $DB_NAME\nTIME: $TIME_NOW\nError: Failed to compress the backup file!"
        echo "Failed to compress the backup file!"
        exit 1
    else
        echo "Compressed dump backup into GZ: $BACKUP_GZ_TODAY"
        # send_discord_notification "STATUS: Success\nDB: $DB_NAME\nTIME: $TIME_NOW\nSuccess: create auto backup $DB_NAME is success!"
    fi
fi
```

## Crontab
Untuk lebih mudah dalam mengatur waktu pada crontab sesuai yang kita inginkan, bisa mengunjungi
website [ini](https://crontab.guru/).

Misalkan saya ingin menjalankan proses backup otomatis menggunakan script `auto_backup.sh` pada
jam 5 pagi setiap harinya.

- buka crontab

    `crontab -e`
- ketikkan waktu yang diinginkan

    `0 5 * * * /home/rohimoz28/script/auto_backup.sh`

## Kesimpulan
Menggunakan gabungan shell script dan crontab merupakan hal yang lumrah pada linux
jika kita ingin membuat sebuah task yang akan berjalan rutin dan otomatis.

Terimakasih