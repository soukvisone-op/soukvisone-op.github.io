# การ์ดคำสั่งลัด (1 หน้า)

> พิมพ์ออกมาแปะข้างจอได้ — 60 คำสั่งที่ใช้บ่อยที่สุด
> ทุกคำสั่งอยู่ในรายการซอฟต์แวร์ที่อนุญาต (ดูหน้า "ซอฟต์แวร์ที่อนุญาต")

---

## 0) สามคำสั่งแรกของทุกไฟล์

```bash
file target                       # มันคืออะไรจริง ๆ
strings -n 6 target | head -40    # มีข้อความอะไร
binwalk target | head -20         # มีอะไรซ่อนข้างใน
```

---

## 1) ไฟล์/ไบต์

```bash
xxd -l 64 f                       # ดู magic bytes
xxd f | head -40                  # ดัมป์ต้นไฟล์
xxd -s 0x200 -l 128 f             # ดัมป์ที่ offset
strings -n 6 f                    # ข้อความ ASCII
strings -el f                     # ข้อความ UTF-16 (ไฟล์ Windows)
strings -t x f | grep -i flag     # พร้อม offset
md5sum f; sha256sum f             # hash ไว้เทียบ
stat f                            # เวลา/สิทธิ์
```

---

## 2) รูป / เสียง

```bash
exiftool -a -u -g1 img.jpg        # metadata ทั้งหมด
exiftool -n -GPSLatitude -GPSLongitude img.jpg
zsteg -a img.png                  # LSB ทุกแบบ
steghide info img.jpg             # มีอะไรซ่อนไหม
steghide extract -sf img.jpg -p '' -f
stegseek img.jpg rockyou.txt      # ไล่รหัสให้
pngcheck -v img.png               # โครงสร้าง PNG
zbarimg -q --raw qr.png           # QR / barcode
sox s.wav -n spectrogram -o s.png -x 2000 -y 1025
multimon-ng -t wav -a DTMF s.wav
convert -list format | head       # ดูฟอร์แมตที่ ImageMagick อ่านได้
montage *.jpg -tile 4x3 full.png  # ต่อภาพ
```

---

## 3) บีบอัด / เอกสาร

```bash
unzip -l f.zip                    # ดูรายชื่อ
unzip -P 'pass' f.zip -d out/     # แตกด้วยรหัส
zip2john f.zip > h && john --wordlist=rockyou.txt h
7z l f.7z && 7z x f.7z -oout/
tar -tvf f.tar && tar -xf f.tar -C out/
gunzip -c f.gz > out.bin
pdfid f.pdf                       # มี JS/OpenAction ไหม
pdftotext -layout f.pdf out.txt
qpdf --qdf --object-streams=disable f.pdf u.pdf
mutool extract f.pdf              # ดึงรูป/ฟอนต์
olevba f.docm                     # macro Office
oledump.py f.docm                 # stream ข้างใน
```

---

## 4) ไบนารี

```bash
checksec --file=f                 # NX/PIE/Canary
readelf -h f; readelf -s f | head
objdump -d -M intel f | head -80
ltrace -s 200 ./f                 # ฟังก์ชันที่เรียก (strcmp!)
strace -f -s 200 ./f              # system call
gdb -q ./f                        # debug
upx -l f; upx -d f -o u           # แพ็ก/แตกแพ็ก
pycdc f.pyc                       # แปลง .pyc
jadx -d out f.apk                 # APK
capa f.exe                        # พฤติกรรม
die f.exe                         # ภาษา/แพ็กเกอร์
```

---

## 5) เครือข่าย

```bash
tshark -r c.pcap -q -z io,phs     # protocol ทั้งหมด
tshark -r c.pcap -q -z conv,tcp   # ใครคุยกับใคร
tshark -r c.pcap -Y http.request -T fields -e http.host -e http.request.uri
tshark -r c.pcap --export-objects http,out/ -q
tshark -r c.pcap -q -z follow,tcp,ascii,0
tcpdump -r c.pcap -nn | head
nmap -sV -p- host
curl -i http://target/
```

---

## 6) ข้อความ/รหัส

```bash
base64 -d <<< 'c2FtcGxl'
base32 -d <<< 'IFBEG==='
xxd -r -p <<< '48656c6c6f'
echo 'text' | tr 'A-Za-z' 'N-ZA-Mn-za-m'      # rot13
rev <<< 'olleh'
python3 scripts/decode.py "<ข้อความ>"          # ถอดซ้อน 3 ชั้น + XOR
hashid '<hash>'
john --format=raw-md5 --wordlist=rockyou.txt h.txt
hashcat -m 0 -a 0 h.txt rockyou.txt
```

---

## 7) ฐานข้อมูล / log

```bash
sqlite3 f.db ".tables"
sqlite3 f.db ".dump" | grep -i -e pass -e key
chainsaw dump f.evtx > ev.txt
hayabusa csv-timber -f f.evtx -o out.csv --no-wizard
grep -E '4624|4625|4688|7045|1102' ev.txt | head
```

---

## 8) ดิสก์ / หน่วยความจำ

```bash
mmls disk.img                     # ตารางพาร์ทิชัน
fls -r -p -o OFF disk.img         # รายชื่อไฟล์
fls -r -d -p -o OFF disk.img      # เฉพาะที่ถูกลบ
icat -o OFF disk.img INODE > out
tsk_recover -e -o OFF disk.img out/
vol -f mem.raw windows.pslist
vol -f mem.raw windows.cmdline
vol -f mem.raw windows.netscan
```

---

## 9) ทำงานเร็ว

```bash
cd $(mktemp -d)                   # โฟลเดอร์ทำงานชั่วคราว
script -a session.log             # บันทึกทุกอย่างที่พิมพ์
tmux new -s work                  # เปิด session (Ctrl+B D ออก)
python3 -m http.server 8000       # รับ-ส่งไฟล์ในวง
nc -lvnp 4444                     # รับ connection
grep -a -r -o 'CSG_FLAG{[^}]*}' . | sort -u
find . -type f -newermt '-1 hour'  # ไฟล์ที่เพิ่งแก้
du -ah . | sort -rh | head -20     # ไฟล์ใหญ่สุด
```

---

## 10) ลำดับคิด 30 วินาที

```
file → strings → exiftool → binwalk
   ↓ ไม่เจอ
ดูชนิด: รูป→zsteg/steghide · เสียง→spectrogram · ซิป→unzip -l
        pdf→pdfid · exe→die/upx · pyc→pycdc · disk→mmls · mem→vol
   ↓ ยังไม่เจอ
ข้อความ? → decode.py · hash? → john · เว็บ? → curl/ffuf
```
