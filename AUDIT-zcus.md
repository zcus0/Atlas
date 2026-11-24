# Audit Fase 2 — Fork Atlas (zcus0/Atlas, branch zcus)

Tanggal: 2026-11-24. Objek: commit `be3a3d4c` (= upstream main = tag 0.6.1 + 13 commit fix).
Metode: clone lokal penuh, inventaris file, verifikasi SHA-256 semua binary terhadap
`package/Executables/AtlasModules/README.md`, grep pola download/network/obfuscation,
baca manual: Toolbox-Package.ps1, analytics.rs, settings.rs, docs/front-door.md.

## 1. Inventaris binary — LULUS 11/11

Total file ter-track: 1.550. Isi repo didominasi teks: 372 .ps1, 298 .psd1, 188 .cmd,
326 .rs (Atlas Manager, Rust). Binary di seluruh repo hanya:

- 4 .exe: multichoice.exe, SetTimerResolution.exe, MeasureSleep.exe, StoreFixer.exe
- 4 .cab: Z-Atlas-NoDefender (amd64/arm64), Z-Atlas-NoTelemetry (amd64/arm64)
- 3 .zip: ViVeTool v0.3.3 (x64/ARM64), TimerResolution-source.zip

**Semua 11 SHA-256 cocok persis dengan inventaris resmi repo.** Tidak ada .dll/.sys/.msi
asing. Setiap binary punya sumber + lisensi tertulis (GPL-3.0/CC0; ViVeTool dari
thebookisclosed/ViVe; StoreFixer byte-identical dari rilis TheyCreeper; CAB dibangun CI
dari config di repo tools/sxsc, builder Atlas-OS/sxsc di-pin commit penuh).

## 2. Download & network di script paket — BERSIH, semua terjelaskan

Pola unduhan (Invoke-WebRequest/curl) di script install/runtime hanya di:
- `Remove-Edge.ps1` — bicara ke API Edge Update Microsoft (edgeupdates/download.microsoft.com)
  untuk proses copot Edge. Bukan pihak ketiga.
- `tools/timer/Build-TimerTools.ps1` — build-time di mesin developer, bukan di PC user.

Domain unik di semua script: mayoritas microsoft.com (docs/download/Store/winget CDN),
github.com + api.github.com (rilis & browser), CDN browser pilihan user (Brave, Mozilla,
LibreWolf via Codeberg, Chrome), download.sysinternals.com, pool.ntp.org (sinkron waktu),
timestamp.digicert.com (penandatanganan saat build). **Tidak ada edit file hosts.**
45 file .url di AtlasDesktop = shortcut dokumentasi, bukan eksekusi.

## 3. Atlas Toolbox — titik unduhan ketemu, higiene lumayan, tetap dicopot dari fork

`Scripts/Operations/Toolbox-Package.ps1` (dipakai Install-Toolbox.ps1 dan langkah software
saat install bila user memilih):
- Download `AtlasToolbox-Setup.exe` dari rilis terbaru GitHub `Atlas-OS/atlas-toolbox`.
- Verifikasi: owner ID GitHub di-pin (78708182), repository ID di-pin (929016610),
  SHA-256 + ukuran aset dari API GitHub dicek. Jalur bundled (test build) dicek ke
  manifest SHA-256 + size.
- Kelemahan jujur dari komentar kodenya sendiri: **Toolbox unsigned** (tanpa Authenticode);
  kepercayaan bertumpu pada pin ID + digest GitHub.
- Keputusan fork: sesuai plan — matikan/pin. Toolbox bukan kebutuhan zcus.

## 4. Atlas Manager (app/, Rust) — ADA analitik, default ON → wajib dibalik di fork

- `app/src/services/analytics.rs`: event Plausible self-hosted ke
  `https://pla.atlasos.net/api/event`. Payload minimal terverifikasi dari kode + test-nya:
  hanya `{name: "pageview", domain: "manager.atlasos.net", url: <jalur layar>}` —
  tidak ada ID hardware/user di body. Antrean bounded, timeout 5 detik, non-blocking.
- **Default `usage_analytics: true`** (`app/src/services/settings.rs:137`, ditegaskan test).
  Ada saklar di Settings. → Perubahan fork #1: balik default jadi `false` (diff 1 baris).
- Update check: saat start, Manager tanya GitHub rilis terbaru (`check_updates`), nonaktif
  otomatis pada build embedded-package.
- `https://reports.atlasos.net` = fitur "Send a report" yang dipicu user dari Settings
  (backend-nya open source juga di `services/reports*` repo ini). Bukan kirim diam-diam.
- Pembuatan ISO memakai daftar file dari domain manager.atlasos.net (jalur analitik di atas)
  dan unduhan ISO tetap dari microsoft.com.

## 5. Obfuscation — tidak ada payload tersembunyi

Grep FromBase64String/-EncodedCommand/Invoke-Expression di package+services+tools:
- `Remove-AtlasLegacyElevationArtifacts.ps1`: string `iex(...)` adalah *catatan artefak
  Toolbox 0.1.x lama yang script ini HAPUS* dari registry — bukan dieksekusi.
- FromBase64String di Install/Tasks = decode opsi install dari Manager (sesuai front-door).
- `tools/lab/` = harness uji Hyper-V milik developer, tidak masuk jalur install user.

## 6. Perilaku sistem yang perlu diketahui (bukan pelanggaran, tapi tercatat)

- Jalur "Defender dimatikan" membuat scheduled task SYSTEM AtStartup (MdCore,
  Atlas.Security/Domain/Defender.ps1) untuk menegakkan pilihan itu. Wajar untuk fungsinya;
  di fork zcus Defender memang dicopot via CAB NoDefender dan AV = ESET.
- Toggle TimerResolution mendaftarkan scheduled task `Force Timer Resolution`.
- Front door terbuka & terdokumentasi (`docs/front-door.md`): `Entry/Install-Atlas.ps1
  -Option @(...) -Unattended`, bisa dijalankan manual di VM uji; build lokal bisa
  `-NoPassword` agar paket gampang dibuka. Ini mempermudah Fase 5 (tes VM) fork.

## Vonis Fase 2

**LULUS dengan 2 catatan tindakan untuk Fase 3:**
1. Balik default `usage_analytics` → false di Atlas Manager fork (settings.rs:137).
2. Matikan/pin jalur Atlas Toolbox (unsigned; tidak dibutuhkan) sebelum build.

Tidak ditemukan: binary tak terinventaris, unduhan pihak ketiga tersembunyi di jalur
install, edit hosts, payload ter-obfuscate, atau telemetri di luar analitik layar
Manager yang ter-toggle di atas.
