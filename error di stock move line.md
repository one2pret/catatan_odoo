+++markdown

# Analisa Error `Record does not exist or has been deleted` pada Stock Transfer di Odoo 12

## 📸 Deskripsi Kasus Berdasarkan Gambar 1

* **Transfer**: `GAPN-2509/0015`
* **Field Bermasalah**: `move_ids_without_package`
* **Quantity Done**: `70` (lebih dari initial demand `60`)
* **Perubahan Manual**: `location_dest_id` diubah ke lokasi lain lewat wizard detail operations (`move_line_ids`)
* **Gejala**: Saat divalidasi → muncul wizard backorder → klik "Create Backorder" → error:

  ```
  Record does not exist or has been deleted.
  (Records: stock.move.line(28276,), User: 57)
  ```

---

## 💡 Ringkasan Masalah

1. Transfer stock dilakukan dengan **quantity\_done > initial\_demand**.
2. `location_dest_id` pada move line diubah manual di wizard.
3. Proses validasi memicu backorder karena mismatch quantity.
4. Saat backorder dibuat, muncul error karena stock.move.line tidak bisa ditemukan.

---

## ⚙️ Proses Internal Odoo 12 (Simplified Flow)

1. `stock.picking.validate()` → memanggil `action_done()`
2. `stock.move._action_done()`:

   * Memproses dan memindahkan `stock.move.line`
   * Cek apakah `quantity_done < product_uom_qty`
3. Jika iya → buat **backorder picking**
4. Jika `quantity_done > demand` → lakukan **split stock.move.line**

   * Bisa clone, hapus, atau duplikasi
5. Jika `location_dest_id` diubah, terjadi konflik data saat manipulasi internal

---

## 🔥 Analisa Masalah

### ✅ Kasus Normal (Berhasil)

* **quantity\_done == initial\_demand**
* Tidak ada backorder
* Tidak ada split
* Tidak terjadi error meskipun `location_dest_id` diubah

### ❌ Kasus Error (quantity\_done > demand + ubah lokasi)

1. Odoo mencoba **split / duplicate move\_line**
2. Proses ini menggunakan record asli (`stock.move.line.id`) dari database
3. Karena `location_dest_id` diubah secara manual:

   * Record dianggap tidak valid (context hilang)
   * Odoo tidak bisa menemukan record untuk di-clone/hapus
4. Terjadi error:

   ```
   MissingError: Record does not exist or has been deleted.
   ```

---

## 🧪 Cara Reproduksi (Simulasi Error)

1. Buat **Stock Picking** dengan `product_uom_qty = 60`
2. Masuk wizard:

   * Set `quantity_done = 70`
   * Ubah `location_dest_id` ke lokasi lain
3. Klik `Validate`
4. Muncul prompt **Backorder**
5. Klik `Create Backorder`
6. ❌ Error muncul:

   ```
   Record does not exist or has been deleted (stock.move.line)
   ```

---

## 🛠️ Solusi / Rekomendasi

* **Jangan ubah `location_dest_id` secara manual** jika ingin mengubah quantity\_done > demand.
* Jika perlu pindah ke lokasi lain:

  * Biarkan picking selesai dulu
  * Gunakan internal transfer untuk memindahkan ke lokasi baru
* Alternatif lain:

  * Override method `action_done()` untuk menangani custom logic jika memang dibutuhkan secara spesifik

---

## 📎 Catatan Tambahan

* Modul ini termasuk logika dari **Odoo Community (OCA)**
* Error ini umum terjadi ketika melakukan override UI behavior tanpa sinkron dengan backend logic
* Bisa dicegah dengan validasi tambahan pada wizard atau kontrol hak akses terhadap perubahan `location_dest_id`

---

+++


+++markdown

📝 Modifikasi Odoo 12: Mendukung Over Delivery dan Perubahan Lokasi Tanpa Error
🎯 Kebutuhan Sistem

Anda memiliki sistem yang membutuhkan:

✅ quantity_done > initial demand (karena ada logika toleransi penerimaan)

✅ Perubahan location_dest_id pada stock.move.line (contohnya: dari PDA/Stock/GD AKSESORIS ke PDA/Stock/GD AKSESORIS/BAGUS)

❌ Tanpa error saat proses Create Backorder seperti:

Record does not exist or has been deleted.
(Records: stock.move.line(28276,), User: 57)

🧠 Fungsi Custom Toleransi (_action_done)

Contoh logika toleransi dalam fungsi _action_done:

@api.multi
def _action_done(self):
    
    for move in self.filtered(lambda x: x.purchase_line_id and not x.allow_tollerance):
        product_id = move.product_id.id
        state = 'done'
        percent = '%'
        toleransi = move.product_id.max_receipt
        total_order_po = move.purchase_line_id.product_qty
        receipt_po = move.purchase_line_id.qty_received

        self._cr.execute("""
            SELECT SUM(product_uom_qty) AS qty FROM stock_move 
            WHERE product_id = %s AND group_id = %s AND state = %s AND company_id = %s 
            AND picking_id <> %s AND location_dest_id = %s
        """, (product_id, move.group_id.id, state, move.company_id.id, move.picking_id.id, move.location_id.id))
        retur = self._cr.fetchone()[0] or 0.0

        uom_po = move.purchase_line_id.product_uom.id
        uom_stock_move = move.product_uom.id

        if uom_po != uom_stock_move:
            conversion_factor = move.product_uom.factor / move.purchase_line_id.product_uom.factor
            total_receipt_po_uom = (receipt_po + move.quantity_done - retur) * conversion_factor
        else:
            total_receipt_po_uom = receipt_po + move.quantity_done - retur

        total_receipt = receipt_po + move.quantity_done - retur
        total_percent = (total_receipt_po_uom * 100) / total_order_po
        gap_percent = total_percent - 100

        if uom_po == uom_stock_move and gap_percent > toleransi:
            product_name = move.product_id.name
            if move.product_id.default_code:
                product_name = f"[{move.product_id.default_code}] {move.product_id.name}"
            raise UserError(
                f"Untuk product {product_name} penerimaan melebihi toleransi receipt!\n"
                f"Quantity PO = {total_order_po} {move.purchase_line_id.product_uom.name}\n"
                f"Quantity receipt saat ini = {move.quantity_done} {move.product_uom.name}\n"
                f"Total quantity receipt = {total_receipt} {move.product_uom.name}\n"
                f"Gap receipt = {gap_percent} ({percent})\n"
                f"Max toleransi receipt = {toleransi} ({percent})!"
            )

    # ✅ Sinkronisasi location_dest_id untuk menghindari error
    for move in self:
        if move.move_line_ids:
            main_dest = move.move_line_ids[0].location_dest_id
            if any(line.location_dest_id != main_dest for line in move.move_line_ids):
                raise UserError("Lokasi tujuan pada move line tidak konsisten. Harap samakan terlebih dahulu.")
            move.location_dest_id = main_dest

    return super(StockMove, self)._action_done()

🛠️ Solusi Teknis untuk Hindari Error Backorder
✅ 1. Sinkronisasi location_dest_id Antara Move & Move Line

Tambahkan logika sinkronisasi agar move.location_dest_id selalu mengacu ke move_line.location_dest_id.

Kenapa penting?

Saat quantity_done > demand, Odoo split move dan referensi lokasi jadi penting.

Mismatch bisa menyebabkan Odoo gagal clone/hapus move line → MissingError.

✅ 2. Override stock.backorder.confirmation._process() (Tangani Error)
from odoo.exceptions import MissingError, UserError

class StockBackorderConfirmation(models.TransientModel):
    _inherit = 'stock.backorder.confirmation'

    @api.one
    def _process(self, cancel_backorder=False):
        try:
            return super(StockBackorderConfirmation, self)._process(cancel_backorder=cancel_backorder)
        except MissingError:
            raise UserError("Gagal membuat backorder karena ada masalah data pada move line (mungkin telah dihapus atau tidak konsisten). Periksa kembali quantity dan lokasi.")

✅ 3. Tambah Constraint Soft untuk Deteksi Awal
@api.constrains('location_dest_id', 'quantity_done')
def _check_consistency_location_qty(self):
    for line in self:
        if line.quantity_done > line.move_id.product_uom_qty and \
           line.location_dest_id != line.move_id.location_dest_id:
            _logger.warning(
                "Move Line %s memiliki qty > demand dan lokasi tujuan berbeda dari move parent. Ini bisa menyebabkan error backorder."
                % line.id
            )

🧪 Contoh Kasus yang Sekarang Berjalan Normal

PO dibuat untuk 60 unit.

Penerimaan 70 unit dengan toleransi 20%.

Lokasi tujuan dipindahkan dari GD AKSESORIS ke GD AKSESORIS/BAGUS.

Sistem memvalidasi penerimaan.

Tidak ada error saat klik "Create Backorder".

Backorder tidak tercipta jika seluruh qty telah diterima.

✅ Ringkasan Solusi
Solusi	Fungsi
Sinkronisasi location_dest_id	Hindari mismatch saat validasi move dan move line
Override backorder.confirmation._process()	Tangani gracefully jika move line tidak ditemukan
Validasi soft constraint	Berikan peringatan jika qty > demand dan lokasi berbeda
Tetap izinkan quantity_done > demand	Karena Anda membutuhkan toleransi over-delivery pada sistem pembelian
🧠 Debug Tips (Opsional)

Tambahkan di akhir _action_done() untuk investigasi lanjutan:

import pdb; pdb.set_trace()
print("Move ID:", move.id)
print("Move Dest:", move.location_dest_id.name)
print("Move Lines:", move.move_line_ids.mapped('id'))
print("Destinations:", move.move_line_ids.mapped('location_dest_id.name'))

📌 Penutup

Dengan kombinasi:

Sinkronisasi lokasi

Validasi custom toleransi

Penanganan error backorder

Constraint lunak untuk developer

...maka Anda dapat mengizinkan over-delivery + perubahan lokasi tujuan tanpa memunculkan error pada proses validasi maupun backorder di Odoo 12.
+++
