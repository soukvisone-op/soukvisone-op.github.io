# อันดับโจทย์ที่ออกบ่อย — เรียงจาก writeup จริง

> นับจาก writeup CSG2025 (38 ข้อ) + CSG2024 ที่รวบรวมไว้
> ตัวเลข = จำนวนโจทย์ที่ใช้เทคนิคนั้น (นับซ้ำได้ถ้าโจทย์เดียวใช้หลายเทคนิค)
> ใช้จัดลำดับการอ่าน/ซ้อม: **ถ้ามีเวลาน้อย ฝึกอันดับ 1-2 ก่อน**

---

## Crypto (6 ข้อใน writeup)

| อันดับ | เทคนิค | พบ | ตัวอย่างโจทย์ | เริ่มยังไง |
|---|---|---|---|---|
| **1** | base64 / hex / encoding ซ้อนชั้น | **3/6** | C3 unf0given, C5 A very old odd file, bad-box | `python3 scripts/decode.py "<ข้อความ>"` |
| **2** | Vigenère / classical | **2/6** | C2 Plato, C3 unf0given | หา crib `CSG_FLAG` → `crypto_cheatsheet.md §4` |
| **3** | hash + john/hashcat | 1/6 | C4 Analyze the forgotten password | `hashid` → `john --mask=?d?d?d?d` |
| **4** | XOR | 1/6 | bad-box | `decode.py` (ลอง 256 คีย์ให้) |

**อ่านเพิ่ม:** `Crypto/crypto_cheatsheet.md`

---

## Forensics (3 ข้อใน writeup + ชุด 2024)

| อันดับ | เทคนิค | หลักฐาน | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | metadata / exiftool | พบใน Passport, Simple Forensic | F1 Passport | `exiftool -a -u -g1 file` |
| **2** | strings + carve ไฟล์ซ่อน | พบใน Simple Forensic | — | `strings -n 6` → `binwalk -e` |
| **3** | memory dump (Volatility) | ชุด 2024 (`.dmp` + TLS key) | memory forensics | `bash scripts/triage.sh dump.raw` |
| **4** | disk image / file system | ชุด 2024 | — | `mmls` → `fls -r` |

**อ่านเพิ่ม:** `Forensics/file_signatures.md`, `binwalk_commands.md`, `strings_commands.md`, `metadata.md`

---

## Stegano (4 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | metadata ซ่อนข้อความ | **2/4** | S3 Hidden in Plain Sight, Cat Tail | `exiftool -a -u -g1` |
| **2** | zip/ไฟล์ซ่อนในรูป | 2/4 | S2 Cutest Cat, S3 | `unzip -l img.jpg` |
| **3** | LSB / bit plane | 1/4 | Cat Tail | `zsteg -a img.png` |
| **4** | steghide / passphrase | 1/4 | S2 | `steghide extract -sf img.jpg -p ''` |

**อ่านเพิ่ม:** `Workflow/workflow.md` (สคริปต์ stego) + `Forensics/binwalk_commands.md`

---

## Reversing (7 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | strings / static analysis | **4/7** | Fake C2, Packer, Flag Encoder | `strings -n 6` + `strings -el` |
| **2** | debugger (x64dbg/gdb) | **3/7** | Fake C2, Packer, Gorilla | `gdb` / `ltrace` |
| **3** | Ghidra / radare2 | **3/7** | Fake C2, Packer, Gorilla | เปิด Ghidra → หา `main` |
| **4** | Go binary (GoReSym) | 2/7 | Fake C2, Gorilla | `strings` + `goresym` |
| 5 | UPX / packer | 1/7 | Packer | `upx -d file` |
| 5 | Python .pyc | 1/7 | Gorilla | `pycdc file.pyc` |
| 5 | PHP nested eval | 1/7 | Matryoshka | ถอด `eval(gzinflate(base64))` |
| 5 | YARA / capa | 1/7 | Hunt 4 the C2 | `capa file` / `yara -r rules/` |

**อ่านเพิ่ม:** `Reverse/reversing_notes.md`, `common_commands.md`

---

## Web (4 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | SQL injection | **2/4** | Easy Web Exploit, Admin | `sqlmap -u "…?id=1" --dbs` |
| **1** | LFI / path traversal | **2/4** | File Viewer, Ping pwn | `?file=../../../../etc/passwd` |
| **3** | file upload bypass | 1/4 | Easy Web Exploit | นามสกุล `.php5/.phtml` + magic bytes |
| **3** | command injection | 1/4 | Ping pwn | `127.0.0.1;id` |

**อ่านเพิ่ม:** `Web/web_cheatsheet.md`, `common_payloads.md`, `commands.md`

---

## Network / Packet (6 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | pcap analysis (tshark) | **4/5** | N2 Silent call, N3 Silent signals, SMB3 | `tshark -r cap.pcap -q -z io,phs` |
| **1** | HTTP/DNS dissection | **4/5** | N1 Man-in-the-flag, SMB3 | `--export-objects http,out/` |
| **3** | Kerberos / credential | 2/5 | SMB3overKerberos, Relayed & Betrayed | `impacket` → hashcat 18200 |
| **4** | VoIP / RTP audio | 1/5 | N2 Silent call | `rtpdump` → `sox` spectrogram |

**อ่านเพิ่ม:** `Misc/useful_commands.md §8` + `Workflow/workflow.md`

---

## OSINT (4 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | geolocation จากภาพ | **2/4** | O1 Japanese geography, Guess the geolocation | `exiftool` → อ่านป้าย/ภาษา |
| **1** | github / git history | **2/4** | Secret Lesson, GitHub | `git log` → `git show -s --format='%ae'` |
| 3 | domain / DNS | 0-1 | — | `whois`, `dig TXT` |
| 3 | username / social | 0-1 | — | ค้นในเบราว์เซอร์ |

**อ่านเพิ่ม:** `OSINT/methodology.md`

---

## MISC (4 ข้อ)

| อันดับ | เทคนิค | พบ | ตัวอย่าง | เริ่มยังไง |
|---|---|---|---|---|
| **1** | archive + password | 2/4 | M4 Open your password | `zip2john` → `john` |
| **2** | mail / SMTP artifact | 1/4 | M2 Send an email | ดู header `.eml` / pcap |
| **3** | QR / barcode | 0-1 | — | `zbarimg -q --raw` |
| **4** | audio / spectrogram | 0-1 | — | `sox … spectrogram` |

**อ่านเพิ่ม:** `Misc/useful_commands.md`

---

## สรุป: 10 เทคนิคที่คุ้มที่สุด (ถ้าซ้อมได้แค่ 10 อย่าง)

1. `strings` + `grep` (ทุกหมวด ใช้บ่อยสุด)
2. `exiftool -a -u -g1`
3. `file` + `xxd` (ดู magic bytes)
4. base64/hex ถอดซ้อน (`decode.py`)
5. `binwalk -e` / `unzip -l` (หาไฟล์ซ่อน)
6. `tshark` สรุป pcap
7. `ltrace` / `gdb` (โจทย์เปรียบเทียบรหัสผ่าน)
8. `zsteg -a` (LSB ในรูป)
9. `sqlmap` / LFI payload
10. `git log` (OSINT)

> ทั้ง 10 อย่างมีคำสั่งพร้อมรันในหน้า `filetypes.html` และ `Workflow/workflow.html`
