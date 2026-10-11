# Termux APK Builder Skill

Build Android apps (APK) from your phone with Termux and GitHub Actions. Written and tested with Claude.

ชุดไฟล์คำสั่ง (สกิล) ให้ AI ช่วยสร้างแอป Android จากมือถือล้วน ใช้ Termux กับ GitHub CLI แล้วให้ GitHub Actions เป็นคน build ให้ ไม่ต้องมีคอมพิวเตอร์

## อ่านก่อน: ใครเขียนไฟล์พวกนี้

เจ้าของรีโปไม่มั่นใจในการเขียนของตัวเอง จึงให้ Claude (AI ของ Anthropic) เขียนไฟล์สกิลทั้งหมดและ README นี้ เจ้าของเป็นคนตั้งโจทย์ และนำไปใช้กับ Claude เพื่อสร้างแอปของตัวเอง

ไฟล์เป็นข้อความธรรมดา จึงอาจใช้กับ AI เจ้าอื่นได้ แต่ยังไม่เคยทดสอบ ถ้าคุณลองแล้ว ช่วยแจ้งผลใน Issues

## สกิลนี้ทำอะไร

AI จะพาทำทีละเฟส: กำหนดขอบเขต, สร้างโปรเจกต์, build APK แรก, เพิ่มฟีเจอร์ทีละอย่าง, แก้บั๊ก, ออก release, แล้วเขียนไฟล์สรุปไว้ให้ AI ตัวต่อไปอ่านต่อได้ ทุกอย่างบันทึกอยู่ใน repo (`PROGRESS.md`) ไม่ได้อยู่ในแชต จึงเปลี่ยน AI กลางทางได้

รองรับ 3 แบบ:
- **Capacitor** (เว็บแอปห่อเป็นแอป): เหมาะกับแอปแบบรายการ ฟอร์ม แดชบอร์ด
- **Flutter**: เหมาะกับหน้าตาที่ทำเอง และแอนิเมชัน
- **Kotlin (native)**: เหมาะกับกล้อง เซนเซอร์ วิดีโอ งานที่ต้องเร็ว

## ต้องมีอะไรบ้าง

- มือถือ Android ที่ลง Termux แล้ว
- บัญชี GitHub
- AI ที่รับไฟล์แนบหรืออ่านไฟล์ข้อความยาวได้

## เริ่มใช้ยังไง

1. แนบ `apk.md` ไฟล์เดียวก่อน แล้วบอก AI ว่าอยากทำแอปอะไร
2. AI จะบอกเองว่าต้องแนบไฟล์ไหนเพิ่ม ตามตารางนี้

| ไฟล์ | ใช้เมื่อ |
|---|---|
| `apk.md` | แนบทุกครั้ง (กฎ เฟส เวอร์ชัน) |
| `ui.md` | เลือกหน้าตาและทำหน้าจอ |
| `capacitor.md` / `flutter.md` / `kotlin.md` | ตามแบบที่เลือก |
| `ios.md` | อยากได้ iOS ด้วย (Capacitor หรือ Flutter เท่านั้น) |
| `security.md` | แอปมีล็อกอิน ฐานข้อมูล อัปโหลดไฟล์ หรือ API key |
| `recipes.md` | ชิ้นส่วนที่ลากย้ายได้ หรือแชตกับ AI (Capacitor) |
| `errors.md` | build แดงหรือแอปเด้ง |

3. ทำทีละเฟสตามที่ AI บอก ทุกขั้นมีบล็อกคำสั่งให้วางใน Termux ได้เลย

## จุดที่มั่นใจ และจุดที่ไม่มั่นใจ

**ตรวจแล้ว (รันจริงหรือเช็คกับ GitHub)**
- ไฟล์ YAML (workflow ทั้งหมด), JSON, JavaScript และสคริปต์ bash ผ่านการตรวจ syntax
- เวอร์ชัน Java, Node, Gradle, AGP, Kotlin และแท็กของ GitHub Actions ตรงกันทุกไฟล์ และแท็กเหล่านั้นมีอยู่จริงบน GitHub
- ไฟล์ที่อ้างถึงกันและหัวข้อที่อ้างถึงกันมีอยู่จริงทุกที่
- คำสั่งเพิ่ม `versionCode` ใช้ได้กับ Capacitor และ Kotlin
- สคริปต์ตรวจความปลอดภัยไม่ฟ้องผิดกับไฟล์ตัวอย่าง และจับของที่ใส่ไว้เองได้

**ยังไม่ได้พิสูจน์ (ต้อง build จริงถึงจะรู้)**
- iOS ยังไม่เคย build จริง และเวอร์ชัน Xcode ที่ Capacitor 8 ต้องการเป็นแค่ "เชื่อว่า 26 ขึ้นไป"
- Flutter และ Kotlin ยังไม่ได้ผ่านการใช้งานจริงมากเท่า Capacitor
- `security.md` ยังไม่มีหน้าล็อกอินและการตั้งค่า client สำหรับ Flutter และ Kotlin
- ลิงก์ดาวน์โหลด Gradle 8.14.3 จาก `services.gradle.org` (แท็กมีจริง แต่ไฟล์ zip ยังไม่ได้เช็ค)
- ชื่อไฟล์ release ของ gitleaks และ osv-scanner อาจเปลี่ยนตามเวอร์ชัน
- `flutter create` ในโฟลเดอร์ที่มี `lib/main.dart` อยู่แล้ว
- ไม่เคยทดสอบกับ AI เจ้าอื่นนอกจาก Claude
- ตัวเลขเวอร์ชันตรวจล่าสุดเมื่อ 2026-10-11 ถ้าเกิน 3 เดือนต้องตรวจซ้ำ (กฎนี้เขียนอยู่ใน `apk.md`)

ถ้า build แรกแดง ดูตารางใน `errors.md` ก่อน

## จุดเด่น

- ทำงานครบวงจรจากมือถือเครื่องเดียว ตั้งแต่โปรเจกต์ build ถึง release
- ใช้ GitHub Actions build ให้ ตรงกับวิธีที่คนใช้ Termux ทำอยู่แล้ว
- มีกฎกันความผิดพลาดที่เจอบ่อย: ห้ามเดาเวอร์ชัน, ใช้ลายเซ็นคงที่ตัวเดียวเพื่อให้ติดตั้งทับได้, ห้ามบอกว่า "ใช้ได้" ถ้ายังไม่มีหลักฐาน
- ประวัติงานอยู่ใน repo เปลี่ยน AI กลางทางได้
- มีตัวช่วยเรื่องหน้าตา (เลือกสีหลักเลขเดียวได้ทั้งชุดสี) และเรื่องความปลอดภัย (ล็อกอิน ฐานข้อมูล การเช็คสิทธิ์)

## จุดด้อย

- ตั้งค่าเริ่มต้นเยอะ (Termux, GitHub, ล็อกอิน)
- ไฟล์รวม 9 ไฟล์ ประมาณ 144 KB อ่านแล้วใช้โทเคนเยอะ ควรเปิดแชตใหม่ทุกเฟสตามที่ `apk.md` แนะนำ
- build ทุกครั้งต้องรอ GitHub Actions และ repo แบบ private มีนาทีฟรีจำกัดต่อเดือน (iOS ใช้นาทีมากกว่า Linux ประมาณ 10 เท่า)
- iOS ได้แค่ไฟล์ `.ipa` ที่ยังไม่เซ็น ต้องเซ็นและติดตั้งเองด้วยเครื่องมือ sideload และแอปที่เซ็นด้วย Apple ID ฟรีจะหมดอายุราว 7 วัน
- APK ที่ได้เป็นรุ่น debug เหมาะกับใช้เองหรือแจกคนใกล้ตัว ถ้าจะขึ้นสโตร์ต้องทำคีย์ release แยกต่างหาก (ไม่อยู่ในขอบเขต)
- ผลลัพธ์ขึ้นกับ AI ที่ใช้ และ AI ยังทดสอบบนมือถือจริงของคุณไม่ได้ ทุกอย่างที่เห็นบนหน้าจอคุณต้องยืนยันเอง

## License

MIT ดูไฟล์ `LICENSE` ใช้ แก้ แจกต่อได้ ขอแค่เก็บชื่อและข้อความ license ไว้ และไม่มีการรับประกันใดๆ

---

## English summary

A set of plain-text skill files that guide an AI assistant through building an Android app (APK) entirely from a phone: Termux and GitHub CLI on the phone, GitHub Actions as the build machine. Stacks: Capacitor, Flutter, Kotlin. Optional unsigned iOS build. Start by attaching `apk.md` only; it tells the AI which other files to ask for.

Honest notes:
- The repo owner is not confident in their own writing, so Claude wrote these files and this README.
- Only tested with Claude. The files are plain text, so other assistants may work, but that is untested.
- Not yet proven by a real build: the iOS workflow (including the Xcode version), parts of Flutter and Kotlin, the Gradle download link, and the release file names of gitleaks and osv-scanner.
- The login screens and client setup in `security.md` are written for Capacitor only.
- Versions were last checked on 2026-10-11. Re-check after three months.

Issues and reports of what worked or failed are very welcome.
