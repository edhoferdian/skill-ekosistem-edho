# Skill Ekosistem Edho

Agent Skills milik Edho Ferdian, dibuat mengikuti [Agent Skills open standard](https://agentskills.io) (`SKILL.md`) sehingga bisa dipakai lintas coding agent — Claude Code, Codex CLI, Gemini CLI, opencode, Cursor, dan lainnya — tanpa modifikasi.

## Skills yang tersedia

| Skill | Deskripsi singkat |
|---|---|
| [`code-review-edho-ferdian`](skills/code-review-edho-ferdian/SKILL.md) | Review kode ala senior engineer (Code Quality, Security, Performance, Blueprint/Spec Consistency) dengan laporan temuan, severity berbasis bukti, dan fix adaptif. |
| [`dev-kickoff-edho-ferdian`](skills/dev-kickoff-edho-ferdian/SKILL.md) | Kickoff & eksekusi proyek development dari dokumen spesifikasi apa pun (PRD, SRS, SDD, WBS, dll) — validasi silang, Execution Context Pack, dan eksekusi task-by-task dengan Reflection gate. |

## Install

Cara termudah pakai [`npx skills`](https://github.com/vercel-labs/skills) — otomatis mendeteksi agent yang terpasang di mesin kamu (Claude Code, Codex, Gemini CLI, opencode, dll) dan symlink skill ke semuanya sekaligus.

Install semua skill:

```bash
npx skills add edhoferdian/skill-ekosistem-edho --all
```

Install skill tertentu saja:

```bash
npx skills add edhoferdian/skill-ekosistem-edho --skill code-review-edho-ferdian
npx skills add edhoferdian/skill-ekosistem-edho --skill dev-kickoff-edho-ferdian
```

Lihat daftar skill yang tersedia di repo ini tanpa install:

```bash
npx skills add edhoferdian/skill-ekosistem-edho --list
```

### Update

`npx skills` menyimpan satu salinan kanonis (symlink ke tiap agent), jadi update cukup sekali dan langsung berlaku di semua agent yang terpasang:

```bash
npx skills update --yes
```

Atau cek dulu apakah ada update sebelum apply:

```bash
npx skills check --yes
```

### Install manual (Claude.ai / Claude Desktop / Claude Code)

Kalau cuma pakai Claude dan mau upload langsung tanpa `npx`, ambil file `.skill` dari folder [`dist/`](dist/) dan upload lewat pengaturan Skills di Claude — tapi cara ini **tidak auto-update**, harus download ulang manual tiap ada perubahan.

## Struktur repo

```
skills/
  <skill-name>/
    SKILL.md          # frontmatter (name, description) + instruksi
    references/        # dokumen pendukung yang di-load sesuai kebutuhan
dist/
  <skill-name>.skill   # arsip zip siap upload manual (opsional, dibuat dari skills/)
```

Untuk menambah skill baru: buat folder baru di `skills/<nama-skill>/` dengan `SKILL.md` (frontmatter minimal `name` + `description`), commit, push — otomatis ter-discover oleh `npx skills`.
