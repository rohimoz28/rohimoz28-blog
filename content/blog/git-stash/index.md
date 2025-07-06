+++
title = 'Git Stash'
date = '2025-07-06T18:32:24+07:00'
draft = true
description = ''
categories= ['Web Dev', 'Git']
tags = ['git']
+++
## Latar Belakang
Ketika seorang developer sedang mengerjakan sebuah fitur lalu ditengah-tengah pekerjaan yang belum selesai tersebut, 
developer terpaksa atau dipaksa untuk meninggalkan pengerjaan fitur yang belum selesai untuk pindah ke branch lain.

Biasanya dikarenakan ada pekerjaan lain yang urgent untuk diselesaikan yang menuntut developer untuk checkout atau pindah dari branch yang sedang di develop.

Pada waktu tersebut, biasanya developer malas atau enggan untuk membuat commit dari pekerjaan yang belum selesai.

## Git Stash
Definisi dari official Git.
> Stashing takes the dirty state of your working directory — that is, your modified tracked files and staged changes — and saves it on a stack of unfinished changes that you can reapply at any time (even on a different branch).

Intinya, dengan fitur **Git Stash**, kita akan menyimpan pekerjaan yang belum selesai ke sebuah tempat penyimpanan khusus untuk list pekerjaan yang belum selesai.

## Contoh Command

### Membuat Stash
Untuk membuat Stash, gunakan command `git stash`.
![Membuat Stash](./images/membuat-stash-baru.png)

### Menampilkan Stash
Ketika ingin menampilkan list dari stash yang telah kita buat, gunakan command `git stash list`.
![List Stash](./images/git-stash-list.png)

### Mengeluarkan Stash Dari Index
Untuk mengeluarkan stash dari index, gunakan command `git stash pop`. Namun **perlu diingat**, menggunakan command ini akan otomatis hanya mengeluarkan stash terakhir yang kita buat.
![Stash Pop](./images/git-stash-pop.png)

### Membuat Branch Dari Stash
Kita juga bisa membuat branch baru dari stash menggunakan command `git stash branch <nama_branch>`. Namun, menggunakan command ini juga otomatis membuat branch baru dari stash terakhir yang kita buat.
![Stash Branch](./images/git-stash-branch.png)

## Kesimpulan
Waktu Penggunaan: ketika sedang ditengah pekerjaan yang belum selesai lalu ingin pindah ke pekerjaan atau branch lain.

Pro Tips : selalu gunakan comment/commit ketika membuat stash supaya tidak lupa.

Sekian. Terimakasih!

