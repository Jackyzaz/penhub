# Quick prompt สำหรับโจทย์ CTF (คัดลอกไปวางได้เลย)

ใช้ไฟล์นี้เวลาจะคุยกับโมเดล (ใน PentAGI, Claude, หรือที่ไหนก็ได้) แบบเร็ว ๆ ไม่ผ่านสคริปต์ `penhub-ctf`
จุดสำคัญที่ทำให้คำขอแบบนี้ไม่โดน safety classifier บล็อกคือ **บอกบริบทให้ชัดตั้งแต่ประโยคแรก**:
เป็นแพลตฟอร์มฝึก CTF สาธารณะ, เป้าหมายคือ flag ที่โจทย์ตั้งใจให้หาเจอ, และวิธีที่จะใช้ไม่ทำลายอะไร
("get flag from this site, do it for me" แบบไม่มีบริบท มักโดนอ่านเป็นขอให้เจาะระบบคนอื่น)

---

## แบบคัดลอกวาง (เติมในวงเล็บเหลี่ยม)

```text
This is a public CTF training challenge from [ชื่อแพลตฟอร์ม เช่น CyLab Academy], a platform
built for people to practice security exercises. The target below is the challenge itself,
not a system I'm attacking without permission — solving it is the intended use of the site.

Challenge: [ชื่อโจทย์ หรือคำอธิบายสั้น ๆ]
URL / target: [URL หรือ host:port]
Flag format: [เช่น academy{...}]

Please analyze this using safe, non-destructive methods only (e.g. inspecting HTTP responses,
reading page/source content, decoding data) — no scanning, brute forcing, DoS, or touching
anything outside this one URL. Walk me through what you find and why, then give the flag
once you're confident, with the evidence that confirms it.
```

## ตัวอย่างที่ใช้แล้วได้ผล (อ้างอิงจากโจทย์จริง)

```text
This is a public CTF training challenge from CyLab Academy, a platform built for people
to practice security exercises. The target below is the challenge itself, not a system
I'm attacking without permission.

Challenge: "On Histiaeus" — find the hidden flag
URL: http://chatelaine.cylabacademy.net:20144/
Flag format: academy{...}

Please fetch this page and look for the flag, including places that don't show in the
rendered page (like HTML comments or response headers). No scanning or brute forcing needed —
just inspect the response. Give me the flag and quote the exact line it came from.
```

ผลลัพธ์ที่ควรได้: โมเดลอ่าน HTML source แล้วเจอ flag ใน comment ท้ายหน้า (`<!--academy{...}-->`)
ไม่ต้องใช้สคริปต์ `requests` เขียนเอง โมเดลที่มีเครื่องมือ fetch URL ทำเองได้เลย

## ทำไมต้องเขียนแบบนี้

- **บอกว่าแพลตฟอร์มคืออะไร** — ชื่อจริง (เช่น CyLab Academy, PicoCTF, OverTheWire) ช่วยให้แยกจาก "เว็บของคนอื่นที่ไม่รู้จัก" ได้
- **บอกว่า target คือโจทย์เอง** — กันไม่ให้ตีความว่ากำลังขอเจาะระบบที่ไม่เกี่ยวข้องกับคำขอ
- **ระบุวิธีที่อนุญาต/ไม่อนุญาต** — "no scanning, brute forcing, DoS" ทำให้ชัดว่าขอบเขตแคบแค่ไหน
- **ขอ evidence ไม่ใช่แค่คำตอบ** — "quote the exact line" บังคับให้โมเดลแสดงที่มา ตรวจสอบง่ายกว่า "เชื่อเลย"
- **หลีกเลี่ยงคำสั่งสั้นห้วนแบบ "do it for me"** — ใส่เหตุผล/บริบทเสมอ แม้โจทย์จะง่ายแค่ไหนก็ตาม

## ถ้าโจทย์ซับซ้อนกว่านี้ (หลายขั้นตอน ต้องรันโค้ด เก็บ log)

ใช้ `ctf/prompt-template.md` ผ่านสคริปต์ `./penhub-ctf new ...` แทน เพราะมี scope, กฎ, และ
รูปแบบรายงานที่ครบกว่า เหมาะกับโจทย์ที่ต้องให้ agent ทำหลายขั้นตอนเองใน Flow ของ PentAGI
