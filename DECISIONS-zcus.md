# DECISIONS — Fork Atlas untuk zcus-pc (branch `zcus`)

Basis: audit Fase 2 (AUDIT-zcus.md) + inventaris live PC 2026-11-24.
PC: Ryzen 5 5600, RX 580 2048SP, RAM 16 GB, Win11 Pro 26200 (stock saat audit).
Aturan fork: semua perubahan di sini, upstream murni di branch `upstream-baseline`.

**PRINSIP UTAMA (user, 2026-11-24 15:56): semua fitur harus bisa di-enable/
disable lagi lewat Atlas tools (Toolbox/AtlasDesktop) — tidak dipakai = disable,
dipakai lagi = enable. TIDAK ADA penghapusan permanen yang tidak bisa dibalik.**
Konsekuensi: ember "MATI/COPOT" hanya boleh berisi (a) toggle/setting yang bisa
dibalik, atau (b) app yang bisa diinstall ulang dari sumber resmi (Store/Microsoft:
Edge, OneDrive, Xbox app, Copilot). Defender TIDAK dicopot (tetap toggle biasa).
Service tidak dihapus — cukup Manual/Disabled.

## Perubahan fork yang sudah diterapkan (commit ini)

1. **Analitik Atlas Manager mati secara default**
   `app/src/services/settings.rs` — `usage_analytics` default `true` → `false`;
   test `analytics_defaults_off_and_respects_a_saved_opt_in` menggantikan test lama.
   Alasan: analitik Plausible (pla.atlasos.net) payload-nya minimal, tapi default ON
   tidak sesuai standar fork ini. Saklar di Settings tetap ada; opt-in eksplisit
   yang tersimpan tetap dihormati.
2. **Atlas Toolbox DIPIN, bukan dimatikan (diubah 2026-11-24 sore, user butuh)**
   Rencana awal "dimatikan dua lapis" DIBATALKAN atas keputusan user — edit di
   Invoke-SoftwarePhase.ps1 & Install-Toolbox.ps1 sudah di-revert bersih.
   Penggantinya: Toolbox tidak download "latest" saat install, tapi **dibundel
   versi tetap** lewat opsi build resmi `--toolbox` (jalur Bundled\Toolbox, yang
   oleh Atlas diverifikasi ke manifest SHA-256 + size sebelum install).
   - Versi pin: **v0.2.0**, `AtlasToolbox-Setup.exe`, 34.973.530 byte.
   - SHA-256 terverifikasi cocok dengan digest resmi GitHub:
     `90fa49981192e4427ce29b500bc9e2bcd0d1e47453fb7b76fb36d46ea7fa30a0`
   - File pin: `~/workspace/atlas-toolbox-pinned/AtlasToolbox-Setup-0.2.0.exe`
     (di luar repo; dimasukkan saat build, bukan di-commit).
   Catatan: Toolbox tetap unsigned (fakta tidak berubah) — pin + hash adalah
   mitigasinya: yang terinstall persis byte yang sudah diverifikasi ini, bukan
   unduhan "terbaru" yang berubah-ubah. Upgrade Toolbox = ganti pin secara sadar.
3. **eClean tidak dipilih** — opsi `install-eclean` di Manager jangan dicentang
   (aplikasi pihak ketiga opsional; tidak ada di PC ini).

## Matriks keputusan tweak (untuk build .atlaspkg)

### LINDUNGI (jangan disentuh tweak apa pun)
- **Defender Antivirus: TETAP ON (diubah 2026-11-24 sore).** Rencana awal "Defender
  dicopot, ESET yang jaga" GUGUR: lisensi ESET user sudah tidak ada — sekali install
  ulang, ESET tidak bisa diaktivasi lagi. Defender gratis, tanpa lisensi, jadi AV
  utama setelah inul (opsi `defender-enable` di Atlas; CAB NoDefender TIDAK dipakai).
  Windows Firewall tetap ON; SmartScreen ON. ESET boleh balik kapan saja kalau
  lisensinya ketemu lagi (ESET HOME / form lost-license), keputusan ini bisa ditinjau
  ulang sebelum build — tapi default fork sekarang: Defender hidup.
  **FINAL (2026-11-24 sore):** recovery ditutup. Key ESET user ternyata key shared
  dari seller — dipakai beramai-ramai sampai statusnya *overused* (melebihi batas
  aktivasi). Key seperti itu tidak bisa diselamatkan: walau ketemu pun aktivasi
  ditolak server ESET, dan support tidak membantu key di luar reseller resmi.
  Defender = AV final untuk build ini, bukan lagi cadangan sementara.
- OS user saat ini ternyata **Ghost Spectre** (bukan stok) → Atlas tidak bisa ditimpa;
  wajib install bersih dari ISO resmi Microsoft dulu, baru Atlas fork. Semua app/data
  di C: hilang → checklist backup (kunci SSH fleet = paling kritis) sebelum inul.
- Riot Vanguard (Valorant) + BattlEye (PUBG) → Secure Boot ON, TPM ON, VBS/Core
  Isolation dibuat gampang di-toggle balik (default OFF untuk FPS, sesuai pilihan
  user; anti-cheat jadi syarat lulus tes VM).
- Steam, Epic, Discord, Brave (browser utama), TikTok LIVE Studio, Voicemeeter,
  AMD Noise Suppression, Tailscale, sshd, Git/Go/Node (NVM), FlyEnv, WSL,
  Microsoft Store + Gaming Services + App Installer (winget/Game Pass).
- OSLink (ldremote + LDRemoteSvc + UltraViewService) — DIPAKAI aktif, bukan sisa lama.
- Clipboard history + notifikasi/Action Center (multitasking).
- WebView2 Runtime (Discord & Orca adalah app Electron).
- Search Indexer: hidup, dibatasi ke folder kerja (bukan dimatikan total).

### DIET (service → Manual, keluar dari startup, app tetap terinstall)
- VMware stack (VMAuthd, VMnetDHCP, VMUSBArb, NAT Service) + tray VMware.
- VirtualBox. WSLService → Manual (hidup sendiri saat `wsl` dipanggil).
- MuMuPlayer (startup off; service manual). Google Play Games services → Manual.
- Startup diet: Hermes WebUI lokal, Air Live Drive, Proton Drive, Gemini app,
  Free Download Manager, Edge Update, Google Updater, RobloxPlayerBeta,
  Epic Games Launcher (jalan saat dibutuhkan, bukan saat boot).

### MATI / COPOT
- Copilot, Recall, Click to Do,
  Widgets/News, iklan & saran Start, telemetri/DiagTrack.
- OneDrive, Teams consumer, Office Hub, Xbox app (kecuali Gaming Services).
- Game Bar & Game DVR (streaming lewat OBS/TikTok LIVE Studio; DVR sudah
  terverifikasi OFF di PC: GameDVR_Enabled=0).
- Hibernation & Fast Startup (desktop; Fast Startup sumber bug driver klasik).
  (Keputusan user 2026-11-24 15:49 — hibernation MATI, dikunci.)
- **Microsoft Edge: COPOT TOTAL** (keputusan user 2026-11-24 15:49). Pakai operasi
  Remove-Edge bawaan Atlas (sudah diaudit Fase 2: bicara ke API Edge Update
  Microsoft saja). WebView2 Runtime TIDAK ikut dicopot (Discord/Orca butuh).
  Edge Update di startup ikut hilang dengan sendirinya.
- **Browser pasca-install: Brave** (keputusan user 2026-11-24 15:49) — dipilih
  lewat opsi browser bawaan Atlas di fase Software, jadi Brave terinstall dari
  CDN resmi Brave saat install Atlas, bukan install manual sesudahnya. Profile
  Brave lama di-restore dari backup checklist.
- Background apps: semua OFF kecuali Steam, Discord, OBS/TikTok LIVE Studio,
  Tailscale, Orca.

### DARI WINUTIL — keputusan user 2026-11-24 15:54
- **BitLocker: DISABLE** (pilihan user; desktop rumahan, bukan laptop).
- **svchost Splitting: DISABLE** (pilihan user; kategori CAUTION di WinUtil —
  dicatat sebagai keputusan sadar, bukan rekomendasi).
- **Brave Debloat: DITOLAK user.** Alasan user: dia mau bisa debloat/restore
  SENDIRI — prinsipnya tweak harus bisa dibalikkan (reversible), bukan dibakar
  permanen ke build. Konsekuensi: tweak opsional macam ini dijalankan dari
  WinUtil/Toolbox pasca-install oleh user, bukan dibake ke paket Atlas.
  Prinsip umum fork ditambah: **utamakan toggle yang reversible**.

### MERGE WINUTIL — semua 69 tweak, dedup vs Atlas (instruksi user 2026-11-24 15:57)
Metode: daftar TAMBAH dijalankan dari WinUtil PASCA-install (bukan dibake ke
fork — sesuai prinsip reversible + tidak duplikasi kode). Klasifikasi hasil
cross-check ke pohon tweak Atlas (package/.../Scripts/Tweaks):

DUPLIKAT — Atlas sudah punya, JANGAN dijalankan dari WinUtil (±35):
Activity History, Hibernation, Widgets, Consumer Features, Telemetry,
Delivery Optimization, End Task Right Click, WPBT, Reserved Storage,
Storage Sense, Windows AI (Copilot/Recall/Click to Do), Right-Click Menu lama,
Background Apps, OneDrive, Edge Remove, Services Manual, Location Tracking,
Mouse Acceleration (qol/disable-mouse-accel), Long Paths
(qol/explorer/enable-long-paths), Explorer Auto-Discovery
(performance/disable-auto-folder-discovery), Visual Effects, Start Menu
Recommendations, Bing Search, Dark Mode, Show Extensions/Hidden Files,
Verbose Logon, Detailed BSoD, Home & Gallery Explorer, svchost Splitting
(performance/system/disable-service-host-split — pilihan user kemarin ternyata
sudah toggle Atlas), Mitigations, Core Isolation, Restore Point.

TAMBAH — WinUtil saja, tidak ada di Atlas (±22, jalankan pasca-install):
- Game Mode toggle (→ OFF, kunci user; WinUtil WPFToggleGameMode).
- BitLocker Disable (keputusan user 15:54; Atlas tidak punya).
- Store Recommended Search Results Disable.
- Prevent Device Metadata/Companion Apps dari network.
- RDP Unsigned File Warnings Disable.
- Razer Software Auto-Install Block + Logitech Download Assistant Block.
- Adobe URL Block List.
- IPv4 Preferred (bukan disable IPv6 — versi aman).
- Disk Cleanup + Delete Temp Files (utilitas sekali jalan pasca-install).
- Grup selera/QoL: NumLock on Startup, Taskbar Alignment/Search/TaskView,
  Scrollbars Always Visible, Sticky Keys OFF, Lock Screen Disable,
  Login Blur OFF, Hide Settings Home Page, New Outlook OFF,
  Start Menu Previous Layout (opsional).

DITAHAN — konflik dengan keputusan user, TIDAK dijalankan walau "semua":
- Disable Search Indexing total (user: indexer hidup, dibatasi folder kerja).
- Disable Notifications (user: notifikasi/Action Center dipertahankan).
- Ultimate Performance Plan (user punya GameTurbo + power plan Atlas).
- IPv6 Disable & Teredo Disable (risiko Tailscale; IPv4-Preferred cukup).
- Brave Debloat (user tolak 15:54 — kendali sendiri).
- Edge Debloat (moot — Edge dicopot), O&O ShutUp10++ (alat eksternal, tumpang
  tindih toggle privasi Atlas), UTC Time (tidak dual-boot), Battery Percentage
  & S0/S3 sleep toggles (desktop — tidak relevan).

### BARU — tweak yang harus ditambahkan (belum ada di Atlas)
- **Game Mode OFF eksplisit**: Atlas tidak punya handling Game Mode
  (tidak ada `AutoGameModeEnabled` di package). Tambah tweak registry:
  `HKCU\Software\Microsoft\GameBar` → `AutoGameModeEnabled = 0`.
  (Registry PC saat ini kosong/tidak eksplisit — kunci di fork supaya pasti.)
- **Power plan**: Ultimate Performance (pengganti "GameTurbo" custom saat ini);
  HAGS tidak diset (RX 580 dukungannya terbatas).

### WinUtil
- Hanya untuk install app (atau diganti daftar winget di playbook). DILARANG:
  preset Standard/Advanced, MicroWin, dan tweak Defender/Update di atas playbook.

## Target terukur (diukur ulang di hardware ini setelah install)
- Idle: ~1,3–1,7 GB RAM / 45–76 proses (dari 181 proses kondisi pakai saat audit).
- Store & update komponen berfungsi; Vanguard/BattlEye lolos; FPS dalam batas noise.
- Rollback = reinstall dari ISO Microsoft (Atlas tidak bisa di-uninstall);
  image disk penuh wajib sebelum deploy ke PC utama.
