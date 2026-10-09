# Konvensi Git ↔ ClickUp (RnD Sandbox)

Repo uji coba: `it8eji/Testing-Clickup`
ClickUp: Space **RnD Sandbox** → List **Development** (ID `1100470000005127`)

Tujuannya: status task di ClickUp bergeser sendiri mengikuti kerja di Git, tanpa developer harus memindahkan kartu manual.

## 1. ID task

Setiap task ClickUp punya ID, terlihat di URL task (`app.clickup.com/t/<workspace>/z8vwhq12ed`) atau lewat tombol salin ID di task.
Di Git selalu ditulis dengan awalan `CU-`:

```
CU-z8vwhq12ed
```

Satu ID cukup muncul di **salah satu**: nama branch, judul/deskripsi PR, atau pesan commit.

## 2. Branch

| Branch | Fungsi |
|---|---|
| `main` | Production (LIVE) |
| `staging` | UAT / staging |
| `dev` | Integrasi, sumber QA |
| `feat/CU-<id>-<ringkas>` | Fitur baru |
| `fix/CU-<id>-<ringkas>` | Perbaikan bug |
| `chore/CU-<id>-<ringkas>` | Infra, config, dokumentasi |

Contoh: `feat/CU-z8vwhq12ed-approve-sto-level`

Aturan: huruf kecil, kata dipisah `-`, tanpa spasi. Branch fitur selalu dibuat dari `dev`.

## 3. Commit

Format: `<jenis>: <ringkasan> CU-<id>`

```
feat: approver resolver per level CU-z8vwhq12ed
fix: batas periode approve salah hitung CU-z8vwhq12ed
```

Jenis: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`.

> ClickUp juga punya sintaks bawaan `#<id>[status]` di commit untuk mengubah status. Jangan dipakai dalam alur normal, karena bisa bentrok dengan workflow otomatis. Simpan untuk kasus darurat saja.

## 4. Pull Request

- Arah PR: `feat/...` → `dev` → `staging` → `main`.
- Judul PR memuat ID: `Approve STO dengan level spesifik CU-z8vwhq12ed`.
- PR **Draft** tidak menggeser status. Tandai "Ready for review" saat siap direview.
- PR ke `dev` **tanpa ID**: workflow membuat task baru di List Development (status CODE REVIEW), menempelkan `[CU-<id>]` ke judul PR, dan memberi komentar berisi link task.

## 5. Status yang bergeser otomatis

| Kejadian di GitHub | Status ClickUp | Catatan |
|---|---|---|
| Branch dibuat / commit pertama dengan ID | IN DEVELOPMENT | Hanya jika task masih BACKLOG, ANALYSIS, atau READY FOR DEV |
| PR ke `dev` dibuka / ready for review | CODE REVIEW | |
| Reviewer pilih **Request changes** | REVISI | Hanya dari CODE REVIEW |
| Push perbaikan ke PR yang sedang REVISI | CODE REVIEW | |
| PR merge ke `dev` | QA/QC | |
| PR merge ke `staging` | STAGING | ID diambil dari semua commit di PR promosi |
| PR merge ke `main` | LIVE/PRODUCTION | Task tertutup |

Status tidak pernah dimundurkan oleh merge. Contoh: task yang sudah STAGING tidak kembali ke QA/QC.

**Tetap manual:**
- BACKLOG → ANALYSIS → READY FOR DEV (PM/analyst).
- BLOCKED.
- QA gagal di QA/QC → pindahkan ke REVISI secara manual, lalu buat PR perbaikan.

## 6. Pengaman

- Workflow hanya mengubah task di List dengan ID `CLICKUP_LIST_ID`. Task Space live tidak akan tersentuh walaupun ID-nya disebut.
- Token disimpan di GitHub Secrets, tidak pernah di kode. Token ClickUp pribadi punya akses penuh seperti akun pemiliknya, jadi jangan dibagikan.
- Repo ini **public**: log Actions dan komentar PR bisa dilihat siapa saja. Jangan menaruh data internal.

## 7. Setup sekali

1. **Isi awal repo.** Commit `README.md`, `KONVENSI-CLICKUP.md`, dan `.github/workflows/clickup-sync.yml` ke `main`. Lalu buat branch `dev` dan `staging` dari `main`.
2. **Token ClickUp.** Di ClickUp: avatar → Settings → **Apps** (ClickUp API) → **Generate** API token (`pk_...`).
3. **Secret GitHub.** Repo → Settings → Secrets and variables → Actions → *New repository secret*: `CLICKUP_TOKEN` = token tadi.
4. **Variable GitHub.** Tab *Variables* → *New repository variable*: `CLICKUP_LIST_ID` = `1100470000005127`.
5. Repo → Settings → Actions → General: pastikan Actions diizinkan.

## 8. Skenario uji

Pakai task `[DUMMY] APPROVAL` (`CU-z8vwhq12ed`, status awal BACKLOG). Cek status di ClickUp setelah setiap langkah, dan log di tab **Actions**.

| # | Langkah | Hasil yang diharapkan |
|---|---|---|
| 1 | Buat branch `feat/CU-z8vwhq12ed-approve-sto-level` dari `dev` | IN DEVELOPMENT |
| 2 | Commit + push 1 file | Tetap IN DEVELOPMENT, commit tertaut di task |
| 3 | Buka PR ke `dev` | CODE REVIEW |
| 4 | Review → Request changes | REVISI |
| 5 | Push perbaikan | CODE REVIEW |
| 6 | Merge PR ke `dev` | QA/QC |
| 7 | PR `dev` → `staging`, merge | STAGING |
| 8 | PR `staging` → `main`, merge | LIVE/PRODUCTION |
| 9 | Branch `feat/tanpa-id`, PR ke `dev` | Task baru dibuat, judul PR dapat `[CU-...]`, ada komentar link |
| 10 | Sebut ID task Space live di judul PR | Warning di log, task live tidak berubah |

Catatan langkah 4: GitHub tidak mengizinkan pembuat PR me-*request changes* PR-nya sendiri. Langkah ini butuh akun kedua sebagai reviewer, atau dilewati dulu.
