# Network — อ่านไฟล์ดักจับเครือข่าย

> ใช้เมื่อ: ได้ `.pcap` / `.pcapng` มา แล้วต้องหาว่าเกิดอะไรขึ้น
> โปรแกรม: `tshark`, `tcpdump`, `nmap`, `curl`, `impacket`, `hashcat`, `sox`, `rtpdump`

---

## 0) อันดับที่เจอบ่อย (จากชุดโจทย์ที่ผ่านมา)

| อันดับ | สิ่งที่ต้องทำ | พบ | เริ่มยังไง |
|---|---|---|---|
| **1** | ดูภาพรวม protocol + ใครคุยกับใคร | 4/5 | `tshark -r c.pcap -q -z io,phs` |
| **1** | ดึง HTTP object / ดู request | 4/5 | `--export-objects http,out/` |
| **3** | Kerberos / credential | 2/5 | ดู AS-REP → hashcat 18200 |
| **4** | VoIP / RTP → ไฟล์เสียง | 1/5 | `rtpdump` → `sox` |

---

## 1) ไฟล์แบบไหน → ใช้โปรแกรมอะไร

| ไฟล์/อาการ | โปรแกรม | คำสั่งแรก |
|---|---|---|
| `.pcap` `.pcapng` | `tshark` | `tshark -r c.pcap -q -z io,phs` |
| ไฟล์ `.pcap` ใหญ่หลาย GB | `tshark -Y` + `-c` | `tshark -r c.pcap -Y http -c 50` |
| ต้องการดูใน GUI | Wireshark | เปิดไฟล์ → Statistics → Protocol Hierarchy |
| มีไฟล์โหลดผ่าน HTTP | `tshark --export-objects` | `tshark -r c.pcap --export-objects http,out/ -q` |
| เสียงใน pcap (VoIP) | `rtpdump` / `sox` | `rtpdump -F payload -f c.pcap > a.raw` |
| hash จาก Kerberos | `hashcat` | `hashcat -m 18200 h.txt rockyou.txt` |
| ต้อง replay request | `curl` | คัดลอก header จาก `tshark -T fields` |
| ต้องสแกนเครื่องปลายทาง | `nmap` | `nmap -sV -p- host` |

---

## 2) ภาพรวมใน 30 วินาที

```bash
# protocol ทั้งหมด (ดูว่ามีอะไรบ้าง)
tshark -r c.pcap -q -z io,phs

# ใครคุยกับใคร ปริมาณเท่าไร
tshark -r c.pcap -q -z conv,ip
tshark -r c.pcap -q -z conv,tcp

# endpoints (IP/port ที่มีบทบาท)
tshark -r c.pcap -q -z endpoints,ip
```

---

## 3) HTTP / DNS

```bash
# request ทั้งหมด
tshark -r c.pcap -Y http.request -T fields \
       -e frame.number -e ip.src -e ip.dst \
       -e http.request.method -e http.host -e http.request.uri

# ดึงไฟล์ที่โหลดผ่าน HTTP
mkdir -p out && tshark -r c.pcap --export-objects http,out -q
ls -la out/

# header ที่น่าสนใจ (auth, cookie, custom)
tshark -r c.pcap -Y http -T fields -e http.authorization -e http.cookie | sort -u

# DNS ที่ถูกถาม
tshark -r c.pcap -Y dns.flags.response==0 -T fields -e dns.qry.name | sort -u

# ตามดู stream แบบข้อความ
tshark -r c.pcap -q -z follow,tcp,ascii,0
```

> โจทย์ชอบซ่อนข้อมูลใน `Host:` หรือ `User-Agent:` ที่ผิดปกติ — ไล่ดูทุก field

---

## 4) ข้อมูลรับรองที่ไม่เข้ารหัส

```bash
tshark -r c.pcap -Y 'ftp || telnet || http.authorization || smtp || pop || imap' \
  -T fields -e frame.number -e ip.src -e ip.dst \
  -e http.authorization -e ftp.request.command -e ftp.request.arg \
  -e telnet.data -e smtp.req.parameter
```

หาคำที่มักเป็นรหัส:

```bash
tshark -r c.pcap -T fields -e data.text 2>/dev/null | tr ',' '\n' | grep -i -E 'pass|user|key'
```

---

## 5) Kerberos / SMB (โจทย์แนว AD)

```bash
# หา AS-REQ / AS-REP
tshark -r c.pcap -Y kerberos -T fields -e kerberos.CNameString -e kerberos.etype | head

# ดู SMB2 operation
tshark -r c.pcap -Y smb2 -T fields -e smb2.filename | sort -u

# ถอด hash จาก pcap (ทำในเครื่อง)
#   AS-REP  -> $krb5asrep$...  -> hashcat -m 18200
#   TGS-REP -> $krb5tgs$...    -> hashcat -m 13100
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 tgs.txt   /usr/share/wordlists/rockyou.txt
```

ถ้ามีไฟล์ `.kirbi` หรือ traffic ที่ replay ได้ ให้ใช้ `impacket`:

```bash
GetNPUsers.py domain.local/ -dc-ip 10.0.0.1 -no-pass -usersfile users.txt
GetUserSPNs.py domain.local/user:pass -dc-ip 10.0.0.1 -request
```

---

## 6) VoIP / RTP → ฟังเสียง

```bash
# ดึง payload ของ RTP ออกมาเป็น raw
rtpdump -F payload -f call.pcap > audio.raw

# แปลงเป็น wav (PCM 8kHz mono ปกติ)
sox -t raw -r 8000 -e mu-law -b 8 -c 1 audio.raw audio.wav

# ฟัง + ทำ spectrogram
sox audio.wav -n spectrogram -o spec.png -x 2000 -y 1025
```

ถ้าเป็น SIP ให้ดู `sdp` เพื่อรู้ codec/port:

```bash
tshark -r call.pcap -Y sip -T fields -e sip.from.user -e sip.to.user -e sdp.media.port
```

---

## 7) ทางที่ข้อมูลออก (exfil)

```bash
# packet ที่ใหญ่ผิดปกติ
tshark -r c.pcap -q -z io,stat,0,"SUM(frame.len)frame.len" | head -30

# DNS ที่ชื่อยาวผิดปกติ (tunnel)
tshark -r c.pcap -Y dns -T fields -e dns.qry.name | awk '{print length, $0}' | sort -rn | head

# ICMP ที่มี payload เยอะ
tshark -r c.pcap -Y icmp -T fields -e data | head
```

---

## 8) เทคนิคจากโจทย์ที่ผ่านมา

| เคส | สิ่งที่เจอ | วิธีแก้ |
|---|---|---|
| Man-in-the-flag | request ไป S3 พร้อม header เข้ารหัสฝั่งลูกค้า | replay ด้วย `curl` + header ชุดเดิม |
| Silent call | RTP ใน pcap | `rtpdump` → `sox` → ฟัง |
| Silent signals | สัญญาณเสียง/DTMF | `multimon-ng -a DTMF` |
| SMB3 + Kerberos | AS-REP hash | `hashcat -m 18200` |
| Relayed & Betrayed | NTLM relay | ดู `ntlmssp` ใน pcap → hash |

---

## 9) คำสั่งที่ใช้บ่อย (คัดลอก)

```bash
tshark -r c.pcap -q -z io,phs
tshark -r c.pcap -q -z conv,tcp
tshark -r c.pcap -Y http.request -T fields -e http.host -e http.request.uri
tshark -r c.pcap --export-objects http,out -q
tshark -r c.pcap -Y dns.flags.response==0 -T fields -e dns.qry.name | sort -u
tshark -r c.pcap -q -z follow,tcp,ascii,0
tshark -r c.pcap -Y 'frame contains "CSG"'
tcpdump -r c.pcap -nn -A | grep -i -e pass -e user
```

> ไฟล์ pcap ที่มีข้อมูลซ่อนแบบไม่เป็น protocol (raw bytes) ให้ `strings c.pcap | grep -i ...` ด้วย
