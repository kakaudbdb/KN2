# PRD — Kain Nusantara (ERP Tekstil)

## Problem Statement Asli
Lanjutkan development dari repo https://github.com/wakasajanamasa/KN (ERP tekstil multi-entitas:
React + FastAPI + MongoDB). Development sebelumnya berhenti setelah iteration_273 (semua tes PASS,
tersisa action items minor).

## Arsitektur
- Backend: FastAPI modular (`backend/routers/*`, `backend/services/*`), MongoDB via Motor.
- Frontend: React CRA, **dilayani dari bundle statis `frontend/build` oleh `static_server.js`
  (TIDAK ada hot reload — setelah edit `frontend/src` WAJIB `bash scripts/rebuild_frontend.sh`)**.
- Multi-entitas: PT Kain Suka Cita (ent_ksc / KSC) + CV Kanda Suka (KANDA); isolasi via
  `entity_scope.py` (entity_ctx / resolve_scope_ids / assert_entity_access), header `X-Entity-Id`.
- Restore lingkungan setelah clone: `bash /app/.restore_env.sh` (pip + yarn + mongo + seed_realistic.py + build).
- Kredensial demo: lihat `/app/memory/test_credentials.md` (admin@kainnusantara.id / demo12345).

## Persona
- Admin (Budi) — akses penuh; Manager (Dewi) — persetujuan; Admin Sales, Finance, Sales, Gudang, Desainer.

## Yang Sudah Diimplementasikan
- s/d iteration_273: Aturan Persetujuan dikolapskan ke skema mesin tunggal
  {doc_type, entity_id, min_amount, max_amount, required_role, sort, active, is_percent};
  UI + mesin (config_service.evaluate_approval) membaca koleksi yang sama. 3 fix kosmetik
  (blanket modal unit 130px, label mini PO create, thead COA).
- 2026-06 (sesi ini — action items iteration_273):
  1. CSS `.form-row-3col` ditambahkan (`styles/components.css`) — modal aturan kini 3 kolom.
  2. Cakupan entitas pada Aturan Persetujuan: dropdown `rule-entity-id` di form
     (Semua entitas / KSC / KANDA), payload POST & PATCH kirim `entity_id`, kolom Cakupan
     menampilkan nama entitas. Backend memvalidasi `entity_id` ∈ allowed_entity_ids
     (403 bila tidak berwenang) pada POST & PATCH (`_assert_rule_entity_allowed`).
  3. GET /approval-rules/{id}: cek 404 dipindah SEBELUM assert_entity_access.
  4. Lebar select Satuan & Grade pada baris Tambah Item PO create: 104px → 130px (kedua grid).
  5. Escape menutup FormModal — sudah tertangani `useEscapeClose` (INV-UI-10), diverifikasi ulang.

- 2026-06 (verifikasi FASE I): FASE I (Inspeksi & QC sebagai dokumen) TERNYATA SUDAH SELESAI
  di repo (plan.md §STATUS I, 2026-08-24, POC 93/93). Diverifikasi ulang di lingkungan ini:
  layar SPK Inspeksi & QC (Gudang → Operasi Gudang → tab SPK Inspeksi & QC) render 2 SPK KSC
  + banner kebijakan; kebijakan default qc.color_mismatch_action=tahan /
  qc.handfeel_mismatch_action=peringatkan; release-hold oleh warehouse → 403
  (HOLD_RELEASE_ROLES = admin, manager); seed 3 SPK + 7 complaint_reasons ada.

- 2026-06 (Dasbor Tahanan QC): beranda manajer kini memuat kartu KPI "Barang ditahan QC"
  (klik → layar SPK Inspeksi, testid manager-home-qc-hold-kpi) + papan antrean
  `inspection_hold` (baris per SPK ber-tahanan). Backend: `MANAGER_BOARD_KEYS =
  HOME_BOARD_KEYS + ("inspection_hold",)` di home_service.manager_home — definisi antrean
  tetap satu di approval_backlog_service (INV-HOME-01), Control Tower admin tidak diubah.

- 2026-06 (FASE N + M ditutup): DRIFT terakhir FASE N dibereskan — notifikasi
  `contra_bon_cycle` kini beralamat `create_addressed(roles=("finance","manager"))`
  (contra_bon_reminder.py), 0 dokumen `recipient_role="all"`. FASE M dibukukan dengan POC
  baru `backend/test_core_makloon_lini_poc.py` (M1–M6, 35/35 PASS, nol residu).
  `audit_md_erp_readiness`: SELESAI=96 · BELUM=0 · DRIFT=0. Rencana MD-ERP habis.

- 2026-06 (sesi lanjutan ke-7 — penutupan & backlog): restore diverifikasi (audit 96/0/0,
  POC N 35/35, POC M 35/35, demo persis, residu 0); commit WIP di-amend jadi pesan penutup
  N+M. Backlog kecil selesai: P2 merged_max (POC approval 41/41), Bug #7 badge tab
  (center_delta 0.00px), 3 pintu RFID verify → DOOR_EXEMPT (penjaga 233 cek HIJAU).
  MASTER_ROADMAP EPIC 0 & 1 diverifikasi TERNYATA SUDAH DIBANGUN sesi lampau — flag
  ui.show_coming_soon terbukti mengendalikan sidebar (dibalik via API lalu dikembalikan),
  role-home per peran hidup, akses sales bersih dari HPP/vendor bill. Regresi
  iteration_274: backend 100% · frontend 100% · nol residu. Header MASTER_ROADMAP dibetulkan.

- 2026-06 (audit vs panduan training MD/Admin Sales): 2 gelombang testing agent
  (iteration_275 alur A/B/C/E/F, iteration_276 alur D/G/H/I/J/K/L + data demo bab 33).
  Mayoritas SESUAI-DOKUMEN; deviasi DITAMPUNG (belum diperbaiki, sesuai permintaan) di
  `TEMUAN_AUDIT_TRAINING.md`: 3 KRITIS (T1 gerbang confirm SO mati — bisa lompati
  verifikasi & ACC manajer; T2 antrean PIN jalan buntu utk sales_admin; T3 isolasi
  entitas bocor di detail/PDF inspeksi), 2 TINGGI (T4 putuskan-ulang pemenuhan 400,
  T5 revisi baris PO yg sudah diterima), 4 SEDANG, 2 MINOR, 7 temuan data-demo/dokumen.
  Seed demo di-reset via scripts/seed_reset.sh setelah audit.

- 2026-06 (sesi lanjutan ke-8 — repo dipindah ke github.com/pandeyoga/KN020926, restore
  via `.restore_env.sh`, demo persis 5·3·12·20 rolls). **3 temuan KRITIS audit training
  DIPERBAIKI:** T1 gerbang confirm SO (default verifikasi ON + 409 "belum disetujui manajer"
  bila status ≠ approved & approval_required), T2 PIN untuk sales_admin (DECIDER/CROSS roles
  di `routers/internal_requests.py`), T3 isolasi detail/PDF inspeksi
  (`entity_scope.assert_active_entity_access`). Bukti: POC E-8 desk 97/97, gate --full HIJAU,
  iter275 A4 + Alur F PASS, iter276 DD2 404.

## Backlog / Prioritas
- **P1 berikutnya (TEMUAN_AUDIT_TRAINING.md):** ~~T4~~ (iter278) · ~~T5~~ (iter279) · ~~T7–T8~~ (iter280,
  2026-09-02: SPK makloon hasil kurang → status Sebagian + `completion_hold` sampai klaim diputus;
  stage `makloon_claim` di /approvals/my-queue + kartu "Klaim Selisih Makloon"; IssueModal makloon
  menampilkan stok tersedia per gudang + 409 menuntun bila bahan kurang); lalu T9 (saran reorder
  R&D/warehouse_id), T10–T11 minor; paket seed demo D1–D5. T6 sudah array-aware di `PoBoardView.jsx`.
- ~~P2: refactor readability `merged_max` di PATCH approval_rules~~ — SELESAI 2026-06
  (ternary bersarang → dua baris jelas; POC approval coverage tetap 41/41, PATCH menolak
  min>max dengan benar). Bug #7 badge tab "Menunggu" juga SELESAI (screenshot before/after).
  Bonus: 3 pintu RFID verify (scan fisik label, bukan keputusan manusia) didaftarkan ke
  `DOOR_EXEMPT` penjaga INV-APPR-01 — penjaga kembali HIJAU 233 cek.
- Rencana fase MD-ERP selesai semua; backlog berikutnya dari BUG_BACKLOG.md / permintaan pemilik.

## Catatan Verifikasi Sesi Ini
- curl: POST rule entity ent_ksc → evaluate-approval mengembalikan rule_id tsb (mesin membaca);
  POST/PATCH entity bogus → 403; PATCH entity sah → OK.
- UI (screenshot): 9 aturan seed tampil, modal 3 kolom, dropdown entitas berisi 3 opsi,
  buat aturan ber-entitas KSC via UI berhasil (Cakupan = "PT Kain Suka Cita (KSC)"),
  Esc menutup modal, select Satuan/Grade 130px terbaca.
- Data uji dibersihkan; DB kembali 9 aturan seed, semua active.


## 2026-09-02 — Klon ulang repo pandeyoga/KNKN ke environment baru
- Sumber: https://github.com/pandeyoga/KNKN (commit 55bb32a). `.env` backend/frontend & `.emergent` environment dipertahankan.
- `.restore_env.sh` dijalankan: pip/yarn install, MongoDB hidup, bootstrap fondasi (expense_categories 8 · gl_accounts 75 · uoms 8), `seed_realistic.py` OK, frontend build OK.
- Verifikasi: login admin@/manager@/salesadmin@ (demo12345) 200, GET /api/sales-admin/desk 200, UI Meja Admin Sales tampil.
- Pengujian terakhir sebelum jeda: iteration_278 (T4 audit training / B guard UI / C pagar gudang E4.1) — belum dijalankan ulang di sesi ini.

## 2026-09-02 — Sesi #072: Restore repo skkajshs/sipro + penutup gelombang 1+2 catatan demo
- Restore via `.restore_env.sh` (`.env` dipertahankan), semua service RUNNING, FE bundle statis di-build.
- Titik henti iter281 DITUTUP: KNSelect Radix menghormati `opt.render` (terverifikasi UI), seed `artwork_parang.png` 480×360 (menggantikan 1×1 px, juga pada master lama), kontras `pabrik:` + 5 warna seed ber-`factory_name`.
- Regresi MD-05 diperbaiki: `scripts/seed_rnd_kpi_demo.py` & `backend/test_core_sampling_poc.py` — proofing tanpa hasil ukur (POC S 66/66, seed KPI desainer 4 permintaan).
- Verifikasi: testing agent iteration_282 BE 100% / FE 100%, pytest iter279–281 hijau.
- Status catatan demo: AS-01, MD-03, MD-04, MD-05, MD-07, PB-02 = SELESAI. **Berikutnya (gelombang 3, keputusan owner sudah ada):** AS-02, AS-03, MD-06, PB-01; lalu T9–T11 & seed D1–D5.

## 2026-09-02 — Sesi #073: Gelombang 3 catatan demo (AS-02 · AS-03 · MD-06) + lencana artwork
- AS-02: `PATCH /api/purchase-requisitions/{id}/lines/{line_no}` — qty beli PR (termasuk PR dari SO) boleh dinaikkan; `order_qty`/`extra_qty`/`qty_history`; UI "ubah qty" di detail PR.
- AS-03: `POST /api/sales-orders/{id}/items/{pid}/release-rolls` — Admin Sales melepas roll sebagian per baris; status SO tetap; jejak `reservation_releases`; UI "Lepas Roll" + log.
- MD-06: `GET /api/rnd/labdip-history` + modal Riwayat Labdip (Pustaka Warna & detail sample), tanggal butuh per putaran, deep-link + highlight putaran.
- Galeri Desain: lencana "Belum ada artwork" pada kartu tanpa berkas.
- Verifikasi: `iter283_as02_as03_scenario.py` 21/21, testing agent iteration_283 BE 100% / FE 100%.
- **Berikutnya:** PB-01 (blanket PO/kontrak → PO), MD-01/02/08, FB-01/02; audit training T9–T11; seed D1–D5.

## 2026-09-02 — Sesi #074: PB-01 · MD-01/02/08 · lencana labdip telat
- PB-01: kontrak blanket menyimpan termin (`payment_term`), PPN (`tax_mode`), harga incl/excl PPN → turun otomatis ke call-off/PO (`payment_due_date` = ETA + net_days); UI blok termin di modal buat & detail kontrak, baris termin di detail PO.
- MD-02/01: master benang (bahan, ply, puntiran, status celup + nomor/sistem) di produk stage Benang & spesifikasi R&D stage Benang; gramasi/lebar disembunyikan.
- MD-08: kode/nama versi supplier ikut ke katalog (`supplier_codes`); pencarian Master Produk & pemilih produk PR/PO/amandemen cocok di kedua sisi.
- Lencana merah "Labdip telat N" di kartu Pustaka Warna (putaran terbuka lewat tanggal butuh) → buka riwayat labdip.
- Verifikasi: testing agent iteration_284 BE 100% / FE 100% (pytest `test_iter284_pb01_md02_md08.py` 13/13).
- **Berikutnya:** FB-01/FB-02 (modul baru), audit training T9–T11, seed D1–D5.
