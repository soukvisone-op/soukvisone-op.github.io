# ซอฟต์แวร์ที่อนุญาต — เช็กลิสต์ก่อนใช้

> อ้างอิง: **AJCCBC CTF2026 Tool Review List (r11)** — ชีต "Approved Tool List for Windows", "… for Kali" และ "Unapproved List"
> กฎเหล็ก: **ใช้เฉพาะที่อยู่ใน Approved List** และ **ห้ามใช้บริการออนไลน์วิเคราะห์ไฟล์**

---

## 1) สรุปสถานะชุดเครื่องมือของเรา

ตรวจด้วยสคริปต์เทียบกับไฟล์ Excel แล้ว (58 แพ็กเกจใน `requirements.txt`):

| ผล | จำนวน | หมายเหตุ |
|---|---|---|
| อยู่ใน Approved List | 51 | ใช้งานได้ |
| เป็น Python library / OS utility | 7 | `python3`, `pip`, `coreutils`, `gmpy2`, `sympy`, `markdown` — อนุญาต (ไม่ใช่ AI/ML) |
| ไม่อนุมัติ | **0** | ไม่มีเครื่องมือเถื่อนในชุดนี้ |

ตรวจเองได้:

```bash
bash scripts/check_env.sh        # ดูว่าขาดอะไร
bash install.sh --dry-run          # ดูว่าจะติดตั้งอะไร
```

---

## 2) โปรแกรมแยกตามหมวด (ที่เราใช้จริง)

### Crypto
| tool | ใช้ทำอะไร | คำสั่ง |
|---|---|---|
| `CyberChef` (เบราว์เซอร์) | ถอดรหัสซ้อนหลายชั้น, Magic mode | เปิดใน Chrome |
| `python3` | เขียนสคริปต์ถอดเอง | `python3 scripts/decode.py "…"` |
| `openssl` | base64, hash, RSA | `openssl enc -d -base64 -in f` |
| `john` / `hashcat` | crack hash | `john --format=raw-md5 --wordlist=rockyou.txt h.txt` |
| `hashid` | บอกชนิด hash | `hashid -m '<hash>'` |
| `RsaCtfTool` | โจมตี RSA อัตโนมัติ | `RsaCtfTool.py -n N -e E --uncipher C` |
| `sage` / `sympy` / `gmpy2` | คณิตศาสตร์/RSA | `sage -python solve.sage` |

### Forensics / Stegano
| tool | ใช้ทำอะไร | คำสั่ง |
|---|---|---|
| `file` | ดูชนิดจริง | `file f` |
| `xxd` | ดัมป์/แก้ไบต์ | `xxd -l 64 f` |
| `strings` | ข้อความในไบนารี | `strings -n 6 f` |
| `exiftool` | metadata (รหัส GPS ฯลฯ) | `exiftool -a -u -g1 f` |
| `binwalk` | หา/แตกไฟล์ซ่อน | `binwalk -e f` |
| `foremost` | กู้ไฟล์จากอิมเมจ | `foremost -t all -i f -o out/` |
| `The Sleuth Kit` | disk image | `mmls d.img` → `fls -r -o OFF d.img` |
| `testdisk`/`photorec` | กู้ไฟล์ที่ลบ | `photorec d.img` |
| `zsteg` | LSB ใน PNG/BMP | `zsteg -a img.png` |
| `steghide` | ซ่อน/ถอดใน JPG | `steghide extract -sf img.jpg -p ''` |
| `stegseek` | ไล่รหัส steghide | `stegseek img.jpg rockyou.txt` |
| `OpenStego` | stego อีกตัว | `openstego extract -sf img.png -xd out/` |
| `pngcheck` | ตรวจโครงสร้าง PNG | `pngcheck -v img.png` |
| `zbarimg` | QR / barcode | `zbarimg -q --raw qr.png` |
| `sox` | spectrogram, แปลงเสียง | `sox s.wav -n spectrogram -o spec.png` |
| `multimon-ng` | DTMF/มอสในเสียง | `multimon-ng -t wav -a DTMF s.wav` |
| `tshark` | วิเคราะห์ pcap | `tshark -r c.pcap -q -z io,phs` |
| `volatility3` | memory dump | `vol -f m.raw windows.pslist` |
| `chainsaw` / `hayabusa` | Windows event log | `chainsaw dump f.evtx` |
| `oletools` | macro Office | `olevba f.docm` |
| `pdfid` / `mutool` / `qpdf` | ตรวจ/แตก PDF | `pdfid f.pdf` |
| `sqlite3` | เปิดฐานข้อมูล | `sqlite3 f.db ".tables"` |
| `ImageMagick` | ต่อ/แปลงรูป (จิ๊กซอว์) | `montage *.jpg -tile 4x3 full.png` |

### Web
| tool | ใช้ทำอะไร | คำสั่ง |
|---|---|---|
| `curl` | ยิง request เอง | `curl -i http://target/` |
| `Burp Suite CE` | intercept/แก้ request | ตั้ง proxy 127.0.0.1:8080 |
| `ffuf` | fuzz path/param | `ffuf -u http://t/FUZZ -w list.txt` |
| `feroxbuster` / `gobuster` | หา directory | `feroxbuster -u http://t -w list.txt` |
| `sqlmap` | SQLi อัตโนมัติ | `sqlmap -u "…?id=1" --batch --dbs` |
| `jwt_tool` | ถอด/แก้ JWT | `jwt_tool <token>` |
| `nmap` | สแกนพอร์ต/บริการ | `nmap -sV -p- host` |
| `hydra` | brute login | `hydra -l admin -P r.txt host http-post-form …` |
| `katana` | crawl เว็บ | `katana -u http://t -d 3` |

### Reverse
| tool | ใช้ทำอะไร | คำสั่ง |
|---|---|---|
| `checksec` | ดูการป้องกัน ELF | `checksec --file=f` |
| `readelf` / `objdump` | โครงสร้าง/asm | `objdump -d -M intel f` |
| `ltrace` / `strace` | ดูการเรียกฟังก์ชัน/syscall | `ltrace ./f` |
| `gdb` (+ peda/pwndbg/GEF) | debug | `gdb -q ./f` |
| `Ghidra` | decompile | เปิด GUI → Analyze |
| `radare2` / `rizin` | วิเคราะห์ในเทอร์มินัล | `r2 -A f` |
| `UPX` | แก้ไฟล์ที่แพ็ก | `upx -d f -o u` |
| `pycdc` / `uncompyle6` | แปลง .pyc | `pycdc f.pyc` |
| `pyinstxtractor-ng` | แตก PyInstaller | `pyinstxtractor-ng f.exe` |
| `jadx` / `apktool` | APK | `jadx -d out f.apk` |
| `dnSpy` | .NET | เปิด GUI |
| `capa` / `YARA` | วิเคราะห์พฤติกรรม/กฎ | `capa f.exe` |
| `FLOSS` | ข้อความที่ถูกเข้ารหัสใน malware | `floss f.exe` |
| `Detect It Easy` | บอกภาษา/แพ็กเกอร์ | `die f.exe` |
| `ROPgadget` | หา gadget (pwn) | `ROPgadget --binary f > gadgets.txt` |

### OSINT / Misc
| tool | ใช้ทำอะไร | คำสั่ง |
|---|---|---|
| `exiftool` | GPS/กล้อง/เวลา | `exiftool -n -GPSLatitude -GPSLongitude f.jpg` |
| `git` | ประวัติ commit/email | `git log --format='%ae %ad'` |
| `curl` + `dig` + `whois` | โดเมน/DNS | `dig +short txt domain` |
| `subfinder` | หา subdomain | `subfinder -d domain` |
| `jq` | อ่าน JSON | `jq . <(curl -s URL)` |
| `ffmpeg` / `mediainfo` | ข้อมูลสื่อ | `ffprobe f.mp4` |
| `Impacket` | Kerberos/NTLM | `GetNPUsers.py domain/ -dc-ip …` |

---

## 3) ของต้องห้าม (จาก Unapproved List)

| ห้าม | ใช้แทน |
|---|---|
| **dirsearch** | `ffuf` / `feroxbuster` / `gobuster` |
| **IDA Free** | `Ghidra` / `radare2` / `rizin` |
| **PEiD** | `Detect It Easy (die)` |
| **stegdetect** | `zsteg` / `stegseek` / `StegSolve` |
| **VirusTotal / CrackStation / dCode / online decompiler** | ทำในเครื่องด้วย `strings`, `john`, `CyberChef` |
| **PE-sieve / HollowsHunter / MemProcFS** | `volatility3` |
| **Fiddler / Cygwin / ImHex** | `Burp`, WSL, HxD |
| **Time Travel Debugging** | `gdb` / `x64dbg` |

> หมายเหตุ: `stegsolve` **อยู่ใน Approved List** (GUI) แต่ `stegdetect` **ไม่อนุมัติ** — อย่าสับสน

---

## 4) กฎการติดตั้ง

| ข้อ | รายละเอียด |
|---|---|
| Python library | เพิ่มได้อิสระ **ยกเว้น** AI/ML (LLM, ML framework, generative AI) |
| `pip install` | ใช้ `--break-system-packages` บน Kali |
| `uv` | อยู่ใน Approved List — ใช้แทน pip ได้ |
| เครื่องมือที่ต้องโหลดเอง | `pycdc`, `RsaCtfTool`, `chainsaw`, `hayabusa`, `sage` (ดูท้าย `requirements.txt`) |
| ติดตั้งไม่ครบ | ไม่เป็นไร — สคริปต์เราข้ามขั้นที่ขาดและบอกวิธีติดตั้งเอง |

---

## 5) คำสั่งตรวจก่อนแข่ง (30 วินาที)

```bash
bash scripts/check_env.sh          # ขาดอะไร
bash install.sh --dry-run            # จะติดตั้งอะไร
bash install.sh                      # ติดตั้งของที่ขาด (core)
python3 scripts/run_all.py --help 2>/dev/null || echo "พร้อมใช้"
```

ถ้า `check_env.sh` ขึ้น `[ขาด]` แต่คุณไม่ต้องการใช้ tool นั้น → ข้ามได้ สคริปต์จะข้ามขั้นให้เอง

---

## 6) ตรวจแล้ว: 25 โปรแกรมที่สคริปต์เรียกใช้จริง

เทียบทีละตัวกับไฟล์ Excel (ชีต Approved + Unapproved) — ผลลัพธ์:

| สถานะ | จำนวน | รายชื่อ |
|---|---|---|
| ✅ ตรงชื่อใน Approved List | **22** | `file` `strings` `xxd` `unzip` `find` `grep` `gunzip` `exiftool` `binwalk` `steghide` `zsteg` `sox` `multimon-ng` `pdfid` `pdftotext` `olevba` `pycdc` `upx` `checksec` `curl` `ffuf` `python3` |
| • ยูทิลิตี้ระบบ (มีทุกเครื่อง) | **2** | `base64` `md5sum` (coreutils) |
| • มาพร้อมชุดที่อนุญาต | **1** | `readelf` (binutils ชุดเดียวกับ `objdump` ที่อยู่ในชีต) |
| ❌ อยู่ในรายการห้าม | **0** | — |

### แยกตามสคริปต์

| สคริปต์ | เรียกใช้ |
|---|---|
| `triage.sh` | file, strings, exiftool, binwalk, unzip, gunzip, zsteg, steghide, sox, multimon-ng, pdfid, pdftotext, olevba, pycdc, upx, checksec, readelf, md5sum |
| `scan.sh` | grep, strings, base64, xxd, find, unzip |
| `web_scan.sh` | curl, ffuf, python3 |
| `run_all.py` | python3 (+ เรียก 3 ตัวข้างบน) |
| `decode.py` | python3 (stdlib ล้วน — ไม่เรียกโปรแกรมภายนอก) |
| `check_env.sh` | ตรวจเฉพาะชื่อใน requirements.txt (ไม่เรียกใช้จริง) |

### หมายเหตุ

- `readelf` ไม่ได้อยู่ในชีตโดยตรง แต่เป็นชุดเดียวกับ `objdump` (binutils) ซึ่งอนุญาต
- `base64` / `md5sum` เป็นยูทิลิตี้พื้นฐานของระบบ ไม่ใช่เครื่องมือวิเคราะห์
- ชื่อของที่ห้าม (dirsearch, IDA Free, PEiD, stegdetect, VirusTotal ฯลฯ) ปรากฏเฉพาะในตาราง "ของต้องห้าม" ข้อ 3 เท่านั้น ไม่ถูกเรียกในโค้ด

ตรวจซ้ำเองได้ด้วย:

```bash
grep -rhoE '\b(dirsearch|ida|peid|stegdetect|virustotal|crackstation|dcode)\b' scripts/ | sort -u
# → ต้องไม่มีผลลัพธ์
```
