# ระบบสถิติการให้บริการ สำนักงานคลังเขต 7

เว็บหน้าเดียว (index.html) ใช้ Firebase Firestore เป็นฐานข้อมูล วางบน GitHub Pages ใช้งานฟรีทั้งหมด

## 1. สร้างโปรเจกต์ Firebase
1. ไปที่ https://console.firebase.google.com > Add project (ไม่ต้องเปิด Google Analytics)
2. **Build > Firestore Database > Create database** เลือก location `asia-southeast1 (Singapore)` แล้วเลือก Production mode
3. แท็บ **Rules** วางเนื้อหาจากไฟล์ `firestore.rules` แล้วกด Publish
4. **Build > Authentication > Get started > Sign-in method** เปิด **Anonymous**
5. **Project settings (รูปเฟือง) > Your apps > Web (</>)** ตั้งชื่อแอป แล้วคัดลอกค่า `firebaseConfig`
6. เปิด `index.html` แทนค่าในส่วน `firebaseConfig` (ค่า apiKey ของ Firebase เปิดเผยได้ ไม่ใช่รหัสลับ)

## 2. ขึ้น GitHub Pages
1. สร้าง repository ใหม่ (Public) แล้วอัปโหลด `index.html`
2. **Settings > Pages > Source: Deploy from a branch > main / (root)** กด Save
3. รอ 1–2 นาที จะได้ลิงก์ `https://<ชื่อผู้ใช้>.github.io/<ชื่อ repo>/`
4. กลับไป Firebase **Authentication > Settings > Authorized domains** เพิ่ม `<ชื่อผู้ใช้>.github.io`

## 3. ใช้งานครั้งแรก
เปิดลิงก์ ระบบจะให้สร้าง "ผู้ดูแล" คนแรก จากนั้นเข้าเมนู **จัดการผู้ให้บริการ** เพื่อเพิ่มรายชื่อเจ้าหน้าที่ และแก้ตัวเลือกในรายการ (ผู้รับบริการ / เรื่อง / ช่องทาง / ตำแหน่ง)

## โครงสร้างข้อมูล
| Collection | Document ID | ฟิลด์ |
|---|---|---|
| providers | อัตโนมัติ | name, position (id), status (active/inactive), role (general/admin), hasPassword |
| credentials | `<providerId>` | salt, iterations, hash (PBKDF2-SHA256 ของรหัสผ่านผู้ดูแล ไม่เก็บรหัสผ่านจริง) |
| records | `YYYY-MM-DD_<providerId>` | providerId, providerName, date, month (YYYY-MM), items[{client, topic, channel, qty}], count (รวมจำนวนราย), rowCount, updatedAt, updatedByName, editCount |
| locks | `YYYY-MM` | เดือนที่ปิดงวด: lockedAt, lockedByName |
| logs | อัตโนมัติ | ประวัติการแก้ไข: at, action, detail, byName (เพิ่มได้อย่างเดียว) |
| settings | `options` / `meta` | ตัวเลือกในรายการ {id, name, active} / วันที่สำรองข้อมูลล่าสุด |

ข้อมูลอ้างอิงตัวเลือกด้วย **id** ไม่ใช่ข้อความ การแก้ชื่อตัวเลือกจึงไม่ทำให้ยอดรายงานเปลี่ยน ตัวเลือกลบไม่ได้ ใช้การ "ปิดใช้งาน" แทน
วันที่ในฐานข้อมูลเก็บเป็น ค.ศ. (เช่น 2026-10-01) ส่วนหน้าจอแสดงเป็น พ.ศ.

## งานประจำของผู้ดูแล
1. **ทุกต้นเดือน**: ดู "สถานะการบันทึก" ว่าใครยังไม่ได้บันทึก → ออกรายงานเดือนที่แล้ว → **ปิดงวด** เดือนนั้น
2. **หลังปิดงวด**: ดาวน์โหลด **ไฟล์สำรอง** เก็บไว้ในที่ปลอดภัย (เช่น Google Drive ของหน่วยงาน)

## โควตาฟรี (Spark plan)
อ่าน 50,000 / เขียน 20,000 ครั้งต่อวัน และเก็บข้อมูล 1 GB เพียงพอมากสำหรับผู้ใช้ 15–20 คน

## ข้อจำกัดด้านความปลอดภัย
ผู้ใช้ทั่วไปเลือกชื่อเข้าใช้โดยไม่มีรหัสผ่าน ส่วนผู้ดูแลต้องใส่รหัสผ่าน ซึ่งตรวจสอบที่หน้าเว็บ ป้องกันการเข้าเมนูผู้ดูแลจากการใช้งานปกติได้ แต่ผู้ที่มีความรู้ด้านเทคนิคยังเขียนข้อมูลผ่าน Firebase ได้โดยตรง เหมาะกับการใช้งานภายในหน่วยงาน
หากต้องการความปลอดภัยมากขึ้น ให้เปลี่ยนไปใช้ Google Sign-in แล้วผูกอีเมลกับผู้ให้บริการ และเขียน Rules ตรวจสิทธิ admin ฝั่งเซิร์ฟเวอร์
