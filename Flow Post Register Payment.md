# Validasi Proses untuk Register Pembayaran di account_payment.py
===========================================================

Validasi proses untuk register pembayaran di account_payment.py dihandle di metode post() dari model account.payment. Berikut adalah langkah-langkahnya:

### Validasi Cek

* Pastikan pembayaran berada di status 'draft'.
* Pastikan semua invoice yang terkait berada di status 'open'.

### Generate Nama Pembayaran

* Jika tidak ada nama, generate nama menggunakan sequence yang sesuai berdasarkan jenis pembayaran (misalnya 'account.payment.customer.invoice' untuk pembayaran masuk pelanggan).

### Buat Journal Entry

* Panggil _create_payment_entry() untuk membuat journal entry utama.
* Untuk transfer, buat journal entry kedua di jurnal tujuan dan rekonsiliasi akun transfer.

### Update Status Pembayaran

* Set status pembayaran menjadi 'posted'.
* Simpan nama move(s) untuk referensi.

### Rekonsiliasi Invoice

* Jika invoice terkait, rekonsiliasi dengan pembayaran.

**Task Selesai**

Proses ini memastikan bahwa pembayaran direkam dengan benar di sistem akuntansi dan terkait dengan invoice yang relevan.

**Ringkasan Proses Validasi**

Proses validasi untuk register pembayaran di account_payment.py mengikuti langkah-langkah berikut:

1. **Validasi Cek**
	* Verifikasi pembayaran berada di status 'draft'.
	* Pastikan semua invoice terkait berada di status 'open'.
2. **Generate Nama Pembayaran**
	* Jika tidak ada nama, generate nama menggunakan sequence yang sesuai berdasarkan jenis pembayaran.
3. **Buat Journal Entry**
	* Panggil _create_payment_entry() untuk membuat journal entry utama.
	* Untuk transfer, buat journal entry kedua di jurnal tujuan dan rekonsiliasi akun transfer.
4. **Update Status Pembayaran**
	* Set status pembayaran menjadi 'posted'.
	* Simpan nama move(s) untuk referensi.
5. **Rekonsiliasi Invoice**
	* Jika invoice terkait, rekonsiliasi dengan pembayaran.
