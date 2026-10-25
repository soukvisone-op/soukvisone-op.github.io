# ไฟล์แบบนี้ → ใช้โปรแกรมอะไร (ตารางรวมทุกชนิด)

> เปิดหน้านี้ก่อน ถ้าไม่แน่ใจว่าไฟล์ที่ได้มาต้องทำอะไรต่อ
> คอลัมน์สุดท้าย = คำสั่งแรกที่ควรพิมพ์ (คัดลอกได้เลย)

---

## 0) กฎ 3 ข้อก่อนเริ่ม

1. **อย่าเชื่อนามสกุล** — `file` ก่อนเสมอ
2. **ทุกไฟล์ลอง `strings`** — เร็วและฟรี
3. **ไม่เจอ → `binwalk`** — ดูว่ามีอะไรซ่อนข้างใน

```bash
file target            # 1
strings -n 6 target    # 2
binwalk target         # 3
```

---

## 1) รูปภาพ

| นามสกุล / ชนิด | magic (hex) | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|---|
| `.png` | `89 50 4E 47` | chunk แปลก, ข้อมูลต่อท้าย, LSB | `pngcheck`, `zsteg`, `exiftool` | `pngcheck -v img.png` |
| `.jpg` `.jpeg` | `FF D8 FF` | metadata, ไฟล์ต่อท้าย, steghide | `exiftool`, `steghide`, `binwalk` | `exiftool -a -u -g1 img.jpg` |
| `.gif` | `47 49 46 38` | frame ซ่อน, comment | `exiftool`, `strings` | `strings -n 4 img.gif` |
| `.bmp` | `42 4D` | LSB, palette | `zsteg`, `StegSolve` | `zsteg -a img.bmp` |
| `.tif` `.tiff` | `49 49 2A 00` | หลาย layer, metadata | `exiftool`, `binwalk` | `binwalk img.tif` |
| `.webp` | `52 49 46 46` | chunk ซ่อน | `binwalk`, `strings` | `binwalk img.webp` |
| `.svg` | `<svg` (text) | comment, script | `cat` | `grep -i -e csg -e flag img.svg` |
| `.ico` | `00 00 01 00` | หลายขนาดซ้อน | `binwalk`, `exiftool` | `binwalk favicon.ico` |
| `.heic` | `ftypheic` | metadata | `exiftool` | `exiftool img.heic` |
| QR / บาร์โค้ด | — | ข้อความในโค้ด | `zbarimg` | `zbarimg -q --raw qr.png` |

---

## 2) เสียง / วิดีโอ

| นามสกุล | magic | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|---|
| `.wav` | `RIFF….WAVE` | spectrogram, LSB, DTMF | `sox`, `multimon-ng` | `sox s.wav -n spectrogram -o spec.png` |
| `.mp3` | `49 44 33` (ID3) | ข้อความใน tag, เสียงซ่อน | `exiftool`, `ffmpeg` | `exiftool s.mp3` |
| `.flac` | `66 4C 61 43` | comment block | `metaflac`, `exiftool` | `metaflac --list s.flac` |
| `.ogg` `.opus` | `4F 67 67 53` | comment, เสียง | `ffmpeg`, `sox` | `ffmpeg -i s.ogg out.wav` |
| `.mp4` `.mkv` | `ftyp` / `1A 45 DF A3` | subtitle, metadata | `ffprobe`, `mediainfo` | `ffprobe -hide_banner s.mp4` |
| `.sstv` / สัญญาณวิทยุ | — | ภาพจากเสียง | `qsstv`, `sstv` | `sstv -d s.wav -o out.png` |

**สูตรเสียงที่ใช้บ่อย**

```bash
sox s.wav -n spectrogram -o spec.png -x 2000 -y 1025   # ดูข้อความในคลื่น
multimon-ng -t wav -a DTMF s.wav                        # เสียงกดปุ่ม -> ตัวเลข
sox s.wav rev.wav reverse                               # เล่นย้อนกลับ
sox s.wav -n stat                                       # ข้อมูลไฟล์
```

---

## 3) ไฟล์บีบอัด / ห่อ

| นามสกุล | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|
| `.zip` | รายชื่อไฟล์, รหัสผ่าน | `unzip`, `zipinfo`, `7z` | `unzip -l f.zip` |
| `.zip` (ติดรหัส) | hash → crack | `zip2john` + `john` | `zip2john f.zip > h && john --wordlist=rockyou.txt h` |
| `.rar` | รหัสผ่าน | `unrar`, `7z` | `7z l f.rar` |
| `.7z` | รหัสผ่าน | `7z` | `7z l f.7z` |
| `.tar` | โครงสร้าง | `tar` | `tar -tvf f.tar` |
| `.gz` `.tgz` | ชั้นใน | `gunzip`, `tar` | `file f.gz && gunzip -c f.gz > out` |
| `.bz2` `.xz` `.zst` `.lz4` `.lz` | ชั้นใน | `bunzip2`, `xz`, `zstd` | `xz -dc f.xz > out` |
| `.cab` | ไฟล์ Windows | `cabextract` | `cabextract f.cab` |
| `.docx` `.xlsx` `.pptx` | จริง ๆ เป็น zip | `unzip` | `unzip -o f.docx -d out/` |
| `.pyz` `.whl` | zip ของ python | `unzip` | `unzip -l f.pyz` |

---

## 4) เอกสาร

| นามสกุล | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|
| `.pdf` | metadata, JS, ไฟล์ฝัง, ข้อความซ่อน | `pdfid`, `pdf-parser`, `mutool`, `qpdf` | `pdfid f.pdf` |
| `.doc` `.xls` `.ppt` (เก่า) | macro, OLE stream | `olevba`, `oledump` | `olevba f.doc` |
| `.docm` `.xlsm` | macro VBA | `olevba -c` | `olevba f.docm` |
| `.rtf` | object ฝัง | `rtfobj` | `rtfobj f.rtf` |
| `.eml` | header, base64 attachment | `cat`, `python3` | `head -40 f.eml` |
| `.txt` `.log` | ข้อความ, base64 | `grep`, `decode.py` | `grep -a -o 'CSG_FLAG{[^}]*}' f.txt` |
| `.md` `.json` `.xml` | โครงสร้าง | `jq`, `xmllint` | `jq . f.json` |

**ชุดคำสั่ง PDF**

```bash
pdfid f.pdf                              # มี /JS /OpenAction ไหม
pdftotext -layout f.pdf out.txt          # ดึงข้อความ
qpdf --qdf --object-streams=disable f.pdf u.pdf   # คลายบีบอัด
mutool extract f.pdf                     # ดึงรูป/ฟอนต์
```

---

## 5) โปรแกรม / ไบนารี

| นามสกุล | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|
| ไม่มีนามสกุล (ELF) | symbol, การป้องกัน | `file`, `checksec`, `readelf` | `checksec --file=f` |
| `.exe` `.dll` (PE) | imports, packer | `objdump`, `die`, `capa` | `die f.exe` |
| `.exe` ที่แพ็ก | UPX | `upx -d` | `upx -l f.exe` |
| `.exe` (.NET) | decompile | `dnSpy` | เปิด dnSpy → Method |
| `.pyc` | bytecode | `pycdc`, `uncompyle6` | `pycdc f.pyc` |
| PyInstaller `.exe` | แตก pyc | `pyinstxtractor-ng` | `pyinstxtractor-ng f.exe` |
| `.apk` | smali/java | `jadx`, `apktool` | `jadx -d out f.apk` |
| `.dex` | java | `jadx`, `dex2jar` | `jadx -d out f.dex` |
| `.jar` | class | `Bytecode Viewer` | เปิด GUI |
| `.so` `.elf` | symbol | `readelf -s`, `r2` | `readelf -s f.so` |
| `.php` | ชั้น eval | `matryoshka` | ถอด `eval(gzinflate(base64))` |
| `.js` (obfuscated) | deobfuscate | `js-beautify` | `js-beautify f.js` |
| `.ps1` `.vbs` | `-enc` base64 | `script_deobfuscate.py` | `python3 scripts/…` |

---

## 6) ดิสก์ / memory / log

| ชนิด | ต้องดูอะไร | tool | คำสั่งแรก |
|---|---|---|---|
| `.img` `.dd` `.raw` (ดิสก์) | partition, ไฟล์ที่ลบ | `mmls`, `fls`, `icat` | `mmls disk.img` |
| `.E01` (EnCase) | ต้องแปลงก่อน | `ewfmount`, `libewf` | `ewfmount f.E01 /mnt/ewf` |
| `.vmdk` `.qcow2` | แปลงเป็น raw | `qemu-img` | `qemu-img convert -O raw f.vmdk f.raw` |
| `.raw` `.mem` `.dmp` (RAM) | process, command line | `volatility3` | `vol -f mem.raw windows.pslist` |
| `.evtx` | event log | `chainsaw`, `hayabusa` | `chainsaw dump f.evtx` |
| `.hive` (registry) | key ที่แก้ | `regripper` | `rip.pl -r NTUSER.DAT -f ntuser` |
| `.db` `.sqlite` | ตาราง/ข้อมูล | `sqlite3` | `sqlite3 f.db ".tables"` |
| `.pcap` `.pcapng` | protocol, ไฟล์ที่โหลด | `tshark` | `tshark -r f.pcap -q -z io,phs` |
| `.pcap` (VoIP) | RTP → wav | `rtpdump` | `rtpdump -F payload -f f.pcap > a.raw` |

---

## 7) ข้อความ / รหัส

| ลักษณะ | คืออะไร | tool | คำสั่งแรก |
|---|---|---|---|
| `A-Za-z0-9+/=` ลงท้าย `=` | base64 | `base64 -d` | `base64 -d <<< '…'` |
| `A-Z2-7` + `=` | base32 | `base32 -d` | `base32 -d <<< '…'` |
| `0-9a-f` ยาวคู่ | hex | `xxd -r -p` | `xxd -r -p <<< '…'` |
| `.-` และ `-` | morse | `decode.py` | `python3 scripts/decode.py "…"` |
| ตัวเลข 0/1 กลุ่มละ 8 | binary | `python3` | `python3 scripts/decode.py "…"` |
| ตัวอักษรล้วน อ่านไม่ออก | Caesar/Vigenère | `decode.py` | ดู `Crypto/crypto_cheatsheet.md` |
| 32/40/64 ตัว hex | hash | `hashid` → `john` | `hashid '<hash>'` |
| จำนวนเต็มก้อนใหญ่ + e + c | RSA | `RsaCtfTool` | `RsaCtfTool.py -n N -e E --uncipher C` |
| `$2b$` / `$6$` | bcrypt / sha512crypt | `john` | `john --format=bcrypt h.txt` |

---

## 8) ตารางตัดสินใจแบบเร็ว (พิมพ์ตามนี้)

```
file f
 ├─ image → exiftool → zsteg/steghide → binwalk
 ├─ audio → sox spectrogram → multimon-ng → ฟังย้อนกลับ
 ├─ zip/7z → unzip -l → ลองรหัส → zip2john+john
 ├─ pdf   → pdfid → pdftotext → mutool extract
 ├─ elf   → strings → checksec → ltrace → gdb
 ├─ pe    → die → strings -el → upx -d → dnSpy/capa
 ├─ pyc   → pycdc
 ├─ apk   → jadx
 ├─ disk  → mmls → fls -r → icat / tsk_recover
 ├─ mem   → volatility3 pslist/cmdline
 ├─ evtx  → chainsaw dump
 ├─ pcap  → tshark -z io,phs → export objects
 └─ text  → decode.py (ถอดซ้อน)
```

> ทุกบรรทัดมีคำสั่งเต็มในหน้าหมวดนั้น ๆ และใน `scripts/`
