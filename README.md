# Music: FLAC + Lirik ke SD Card

Alur: **Downloads → songs → (lirik) → library → SD card**

## Struktur folder

| Folder | Isi |
|---|---|
| `songs/` | FLAC mentah (`Artis - Judul.flac`) + `.lrc` hasil getlyrics |
| `library/` | hasil rapihan: `Artis/Album/NN - Judul.flac` + `.lrc` |
| `done/` | arsip batch lama yang sudah diproses |
| `no-lyrics/` | buat lagu yang liriknya nggak ketemu (opsional) |
| `covers/` | gambar cover manual |
| `tools/` | script |

Butuh: `python3`, `mutagen`, `requests` (`pip install mutagen requests`).

## Step by step

### 1. Pindahin FLAC baru ke `songs/`
```fish
mv /workspace/Downloads/*.flac /workspace/Music/songs/
```

### 2. Cek tag metadata (artist, title, album, tracknumber)
Script baca **tag**, bukan nama file. Lagu tanpa artist/title bakal `NO META`.
```fish
for f in /workspace/Music/songs/*.flac; metaflac --show-tag=ARTIST --show-tag=TITLE --show-tag=ALBUM $f; end
```
Kalau ada yang kosong, isi pakai MusicBrainz Picard atau:
```fish
metaflac --set-tag="ARTIST=Nama Artis" --set-tag="TITLE=Judul" "file.flac"
```

### 3. Ambil lirik synced
```fish
/workspace/Music/tools/getlyrics
```
- Nyimpen `.lrc` di sebelah FLAC-nya, yang sudah punya `.lrc` di-SKIP.
- `Lyrics tidak ditemukan` → cek ejaan artist/title di tag, atau cari manual di lrclib.net.

### 4. Rapihin ke `library/`
```fish
/workspace/Music/tools/organize_music
```
- **Copy** (bukan move) ke `library/Artis/Album/NN - Judul.flac` + `.lrc`.
- File yang sudah ada di library di-SKIP, jadi aman dijalanin ulang.

### 5. Salin ke SD card
Colok SD card, cek nama mount-nya:
```fish
lsblk -o NAME,SIZE,MOUNTPOINTS
```
Biasanya di `/run/media/Shuu/<NAMA_SD>`. Lalu:
```fish
rsync -rtv --size-only --modify-window=2 /workspace/Music/library/ /run/media/Shuu/<NAMA_SD>/Music/
```
- Garis miring di akhir `library/` penting (yang disalin isinya, bukan foldernya).
- `-rt` tanpa `-a`: SD card biasanya FAT/exFAT, nggak bisa simpan owner/permission.
- Coba dulu pakai `-n` (dry run) kalau ragu.

### 6. Lepas SD card dengan aman
```fish
sync; udisksctl unmount -b /dev/<sdX1>
```

### 7. (Opsional) Beresin `songs/`
Kalau sudah aman di library + SD:
```fish
mv /workspace/Music/songs/* /workspace/Music/done/
```

## Catatan
- Karakter `<>:"/\|?*` di nama diganti `_` (biar aman di exFAT).
- Track number kosong jadi `00`.
- Album kosong masuk folder `Unknown Album`.
