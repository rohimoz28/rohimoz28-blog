+++
title = 'Git Stash - Part 2'
date = '2025-07-06T20:04:51+07:00'
draft = false
description = 'Sekarang waktunya menggunakan stash seperti seorang pro.'
categories= ['Web Dev', 'Git']
tags = ['git']
+++
## Git Stash Pro Tips
Artikel ini adalah lanjutan dari artikel [Git Stash - Part 1]({{< ref "blog/git-stash-1" >}}).

Pada artikel ini, kita akan coba menggunakan Stash seperti seorang pro.

### Menyimpan Stash Dengan Pesan
`git stash push -m "pesan_commit_untuk_stash"` yang nantinya bisa dijadikan referensi task yang belum selesai.
![Git Stash Push](./images/git-stash-push.png)

Lalu ketika kita cek list index stash maka akan muncul sebagai berikut,
![Git Stash Push List Index](./images/git-stash-push-list.png)

### Menampilkan Konten Stash
Jikalau kita ingin melihat konten dari stash, kita bisa gunakan command `git stash show -p <index_stash>`.
![Git Stash Show](./images/git-stash-show.png)

Atau, jika hanya ingin melihat resume dari file-file yang berubah, bisa gunakan command `git stash show <index_stash>`
### Apply Stash Berdasarkan Index
Misalkan kita punya beberapa index pada stash, lalu ingin mengeluarkan stash dengan index 1, kita bisa gunnakan
command `git stash apply <index_stash>`.
![Git Stash List Index All](./images/git-stash-list-index-all.png)

**Note: ** apply stash tidak otomatis menghapusnya dari index stash.

Kita akan keluarkan stash di index kedua.
![Git Stash Apply](./images/git-stash-index-apply.png)

### Hapus Stash Berdasarkan Index Stash
Jika kita ingin menghapus stash berdasarkan index, kita bisa gunakan command `git stash drop <index_stash>`. 

Sebagai contoh, kita akan menghapus index stash kedua (index stash : stash@{1}).
![Git Stash Drop](./images/git-stash-drop-command.png)

Hasilnya akan sebagai berikut,
![Git Stash Drop](./images/git-stash-drop-after.png)

### Hapus Semua Stash
Terakhir, jika kita ingin menghapus semua index stash, kita bisa gunakan command `git stash clear`.
![Git Stash Clear](./images/git-stash-clear.png)

## Kesimpulan
Gunakan git stash dengan bijak. Menggunakan tools kolaborasi seperti git, yang paling penting adalah disiplin dan kominkasi yang jelas ke tim.

---
## References:
- [Official Page - Git Stash](https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning)
- [Medium - Git Beyond the Basics: Commands That Make You More Effective](https://medium.com/@charu.sharma517/git-beyond-the-basics-commands-that-make-you-more-effective-e2e245a889db)