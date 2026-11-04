# Credential — demo kredensial di Ethereum Sepolia

Credential adalah registry EVM dengan frontend React untuk menerbitkan, melihat, memeriksa status dan metadata, serta mencabut kredensial. Ini adalah demo testnet, bukan layanan sertifikasi atau sistem identitas yang telah diaudit.

Demo publik: https://credential-sepolia.loftyfrog256.chatgpt.site

Source rilis: branch `codex/credential-v1-release`; [draft PR #1](https://github.com/jsolution-protocol/ProfFound/pull/1). PR menunggu uji penerimaan Sepolia sebelum merge.

## Deployment

- Jaringan: Ethereum Sepolia, chain ID `11155111`.
- Kontrak: `0x8C2BA413dAe20d53f6F1A32E0068DAE91D94FE03`.
- [Source terverifikasi](https://eth-sepolia.blockscout.com/address/0x8C2BA413dAe20d53f6F1A32E0068DAE91D94FE03?tab=contract).
- [Transaksi deployment](https://sepolia.etherscan.io/tx/0x7c0920034e20eeea5d1f63a6392f640d625667e05cabbccae99cc61414273e74), blok `11933240`.
- Solidity `0.8.20+commit.a1b79de6`, optimizer 200 runs, EVM Shanghai. Creation dan runtime bytecode telah dicocokkan dengan artifact lokal.
- Bukti deployment: [deployment.json](deployments/sepolia/deployment.json).

## Menjalankan frontend

Prasyarat: Node.js yang kompatibel dengan Vite 8, npm, dan ekstensi MetaMask untuk transaksi. Melihat credential tidak membutuhkan wallet.

```powershell
cd frontend
npm.cmd ci
npm.cmd run dev
```

Buka URL Local di terminal. Pilih Ethereum Sepolia dan hubungkan wallet pengujian untuk issue/revoke. Gunakan Sepolia ETH dari faucet; saldo Anvil tidak berlaku di Sepolia. Frontend tidak memerlukan private key atau file `.env`.

1. Siapkan dua akun berbeda: issuer dan recipient.
2. Isi alamat recipient dan judul pada form issue.
3. Expiry opsional memakai zona waktu perangkat; kosong berarti tidak kedaluwarsa. Pilih waktu cukup jauh untuk konfirmasi transaksi.
4. Pilih file metadata asli, misalnya `metadata/sample-credential.json`; hash terisi otomatis. Contoh metadata adalah fixture, bukan sertifikat nyata untuk recipient Anda.
5. Konfirmasi transaksi di MetaMask, lalu lihat credential berdasarkan ID atau alamat recipient.
6. Pilih file metadata yang sama untuk membandingkan proofHash.
7. Revoke menggunakan issuer asli, lalu query ulang statusnya.

Metadata tidak otomatis diunggah. Bagikan file aslinya kepada verifier secara terpisah. Jangan masukkan informasi pribadi sensitif ke transaksi publik.

## Apa yang diverifikasi

Status valid berarti credential ada, belum dicabut, dan waktu blok belum melewati expiry. Pada tepat waktu expiry credential masih valid; setelahnya tidak valid. Expiry tidak mengubah status tersimpan menjadi Revoked. Query ulang untuk memperbarui tampilan. Anvil default membutuhkan blok baru agar waktu blockchain bergerak.

`proofHash = keccak256(byte file asli)`. Tidak ada normalisasi JSON, trim, atau fallback SHA-256. Perubahan spasi, urutan key, BOM, encoding, atau LF/CRLF dapat mengubah hash. File diproses lokal di browser. Hash cocok membuktikan kecocokan byte terhadap digest issuer, bukan kebenaran isi atau kredibilitas issuer. File yang ditautkan di dalam metadata tidak otomatis diverifikasi. Credential lama dengan aturan hash lain memerlukan aturan aslinya.

CLI menggunakan dependensi ethers frontend:

```powershell
node scripts/generate-proof-hash.js metadata/sample-credential.json
# Argumen ketiga opsional: hash bytes32 dari kontrak, untuk perbandingan.
```

## Arsitektur dan cakupan

- `src/CredentialRegistry.sol`: sumber kebenaran status on-chain, indexing recipient, expiry, kontrol revoke issuer.
- `src/interfaces/`: struktur credential, event, custom error, dan ERC-165.
- `frontend/src/`: React + ethers, RPC baca Sepolia, koneksi MetaMask dan pemeriksaan chain sebelum pengiriman transaksi.
- `metadata/`: schema dan contoh metadata off-chain. Registry tidak menyimpan file atau URI metadata.
- `scripts/`: hashing dan test frontend/helper.
- `deployments/sepolia/`: source input compiler, ABI, bytecode dan bukti deployment publik.

Siapa pun dapat menjadi issuer kepada wallet lain; self-issuance ditolak. Tidak ada daftar issuer terpercaya. Kontrak juga menyediakan batch issue/revoke, tetapi UI v1 berfokus pada operasi satu credential. Tidak ada klaim kepatuhan penuh terhadap standar W3C VC atau verifikasi dokumen bertanda tangan terpisah.

## Pengujian

```powershell
forge test
node --test scripts/*.test.mjs
cd frontend
npm.cmd run lint
npm.cmd run build
```

Baseline terakhir: 21 test Foundry lulus, termasuk fuzz 256 runs; 15 test JavaScript lulus. Lint dan build berhasil. Lihat [checklist dan bukti uji](docs/06-release-checklist.md) untuk status terbaru. Test lulus bukan audit keamanan.

Uji dua akun di Anvil mencakup issue, revoke, penolakan non-issuer, pembatalan MetaMask, jaringan salah, dan expiry. Uji transaksi end-to-end Sepolia masih perlu diselesaikan. Issuance melalui Smart Account pernah gagal; penonaktifan Smart Account issuer berhasil, tetapi itu belum membuktikan penyebab pasti atau kompatibilitas seluruh wallet.

## Publikasi frontend dan biaya

```powershell
cd frontend
npm.cmd ci
npm.cmd run build
```

Host hanya folder `frontend/dist`. Jangan unggah root repository, `.env`, private key, deployment page development, atau `node_modules` ke document root. Frontend adalah website statis; tidak membutuhkan VPS/database/backend untuk fitur v1. Gunakan HTTPS dan URL bawaan hosting dahulu; domain berbayar opsional. Pembelian dan biaya perpanjangan domain harus diperiksa sebelum checkout.

Halaman `deploy-sepolia.html` khusus server development lokal dan bukan bagian build produksi. Jangan deploy kontrak ulang untuk menerbitkan frontend. Konfigurasi kontrak ada di `frontend/src/contractConfig.js`; chain yang diizinkan ada di `frontend/src/walletNetwork.js`. Keduanya harus sesuai deployment.

## Pengembangan kontrak lokal

```powershell
anvil
# Pada terminal lain:
forge create src/CredentialRegistry.sol:CredentialRegistry --rpc-url http://127.0.0.1:8545 --unlocked --from 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 --broadcast
```

Akun broadcaster bawaan tersebut hanya untuk Anvil. Jangan menggunakan private key bawaan Anvil di jaringan publik. Untuk menggunakan deployment lokal di UI, sesuaikan alamat, RPC dan expected chain bersama-sama; default repo adalah Sepolia.

## Keterbatasan rilis

- Demo Sepolia; bukan deployment mainnet dan belum diaudit.
- Ketersediaan RPC publik dan metadata eksternal tidak dijamin.
- Pencarian recipient mengembalikan seluruh ID; belum ada pagination untuk skala besar.
- Metadata bukan penyimpanan permanen otomatis, schema validation, atau penilaian issuer.
- Kecocokan hash dan status kontrak ditampilkan terpisah.
- Refresh status diperlukan; frontend belum memberikan monitoring expiry real-time.
- End-to-end testnet, review/merge draft PR rilis, dan video demo mengikuti checklist rilis. Frontend dan source branch sudah dipublikasikan setelah scan heuristik secret.

File lisensi belum tersedia di repository; tentukan lisensi sebelum distribusi source sebagai proyek berlisensi terbuka.
