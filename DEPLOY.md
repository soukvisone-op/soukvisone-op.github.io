# DEPLOY — ขึ้น GitHub Pages แบบละเอียด

โฟลเดอร์นี้มี `index.html` ที่ root แล้ว → ไม่ต้องตั้งค่าอะไรเพิ่มนอกจากเปิด Pages

---

## 1) อัปโหลด

### ทางเว็บ (ง่ายสุด)

1. `github.com/<user>/<repo>` → **Add file → Upload files**
2. เปิดโฟลเดอร์นี้ → **Ctrl+A** → ลากเข้าไป
   (ต้องลาก **ไฟล์ข้างใน** ไม่ใช่ลากโฟลเดอร์ทั้งก้อน และ **ไม่ใช่ไฟล์ .zip**)
3. **Commit changes**

### ทาง command line

```bash
cd CTF-2026
git init && git add . && git commit -m "site"
git branch -M main
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin main
```

---

## 2) เปิด Pages

```
Settings → Pages
  Source : Deploy from a branch
  Branch : main
  Folder : / (root)
  Save
```

---

## 3) URL

| ชื่อ repo | URL |
|---|---|
| `<user>.github.io` | `https://<user>.github.io/` |
| ชื่ออื่น | `https://<user>.github.io/<repo>/` |

❌ อย่าใช้ `.../blob/main/...` — นั่นคือหน้าโค้ดบน GitHub ไม่ใช่เว็บ

---

## 4) แก้ปัญหา 404

| อาการ | สาเหตุ |
|---|---|
| 404 ทั้งเว็บ | ยังไม่ Save Pages / folder ไม่ใช่ root |
| 404 แต่ repo มีไฟล์ | อัปเป็น .zip → ต้องแตกก่อน |
| ไม่มีสไตล์ | ไม่ได้อัป `assets/` |
| เห็น README แทนเว็บ | ไม่มี `index.html` ที่ root |
| ของเก่าค้าง | รอ 1 นาที + Ctrl+F5 |
| เปิด /docs/ แล้ว 404 | ปกติ — โฟลเดอร์ docs ไม่มีในชุดนี้แล้ว |

---

## 5) ทดสอบในเครื่องก่อนอัป

ดับเบิลคลิก `index.html` ได้เลย — เป็น static ทั้งหมด ไม่ต้องมีเซิร์ฟเวอร์
