+++
title = 'Override ir.model.access.csv'
date = 2024-07-29T18:41:16+07:00
draft = false
categories = ['Web Dev']
tags = ['odoo']
+++

### Latar Belakang Masalah
Biasanya, ketika kita meng-install module odoo dari third party, sudah otomatis ada pengaturan `ir.model.access.csv` bawaan modul tersebut. Ada kalanya, 
kita ingin menyesuaikan hak akses tersebut dengan kebutuhan di modul kustomisasi kita.

### Solusi
Maka dari itu, kita perlu melakukan override hak akses yg ada di `ir.model.access.csv`.

### Override ir.model.access.csv

Dibawah ini adalah contoh hak akses di `ir.model.access.csv` yang ingin kita override.

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_hr_holidays_status_manager,hr.holidays.status manager,model_hr_leave_type,hr_holidays.group_hr_holidays_manager,1,1,1,1
```

Untuk meng-override nya, silahkan buka file `ir.model.access.csv` di modul kustomisasi kalian dan ubah menjadi kira-kira seperti ini,
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
hr_holidays.access_hr_holidays_status_manager,hr.holidays.status manager,model_hr_leave_type,my_hr_tms.module_sub_category_permission_code_manager,1,1,1,1
```

- pada bagian `id` diubah menjadi `hr_holidays.access_hr_holidays_status_manager`. `hr_holidays` adalah nama modul yang kita override.

- lalu di bagian `group_id:id` di ubah menjadi `my_hr_tms.module_sub_category_permission_code_manager`.

  - `my_hr_tms` adalah nama module kustomisasi kalian yang meng-override `ir.model.access.csv` di module third party `hr_holidays`.
  - `module_sub_category_permission_code_manager` adalah nama group baru buatan kalian, yang nantinya akan digunakan di modul kustomisasi.

### Contoh Penggunaan

Sebagai contoh penggunaan, kita akan coba override `groups` pada `menuitem`.

- sebelum di override
```xml
<menuitem        
    id="hr_holidays_status_menu_configuration_ext"        
    action="hr_holidays.open_view_holiday_status"        
    name="Permission Code"        
    parent="my_hr_tms.hr_menu_tms_setting"        
    groups="hr_holidays.group_hr_holidays_manager"        
    sequence="40"
/>
```

- setelah di override
```xml
<menuitem        
    id="hr_holidays_status_menu_configuration_ext"        
    action="hr_holidays.open_view_holiday_status"        
    name="Permission Code"        
    parent="my_hr_tms.hr_menu_tms_setting"        
    groups="my_hr_tms.module_sub_category_permission_code_manager"        
    sequence="40"
/>

```

Setelah melakukan perubahan, jangan lupa untuk upgrade modul kalian.

### Penutup
Untuk mengubah hak akses di `ir.model.access.csv` yang sudah ada. Cukup melakukan override di file `ir.model.access.csv` pada modul kustomisasi kalian.
Contoh penggunaannya bisa di banyak tempat. Salah satunya pada `menuitem`.