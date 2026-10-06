# บันทึกการติดตั้ง PentAGI บน `kalla`

ตรวจสถานะล่าสุดเมื่อ 19 กันยายน 2026

## ตำแหน่งและองค์ประกอบ

- เครื่อง: `soravit@kalla`
- ไดเรกทอรีติดตั้ง: `/home/soravit/pentagi`
- ใช้ Docker Compose ทางการร่วมกับไฟล์ `docker-compose.local-llm.yml` สำหรับ Ollama ในเครื่อง
- โมเดล LLM: `llama3.1:8b-instruct-q8_0` (ประมาณ 8.5 GB)
- โมเดล embedding: `nomic-embed-text` (ประมาณ 274 MB)
- ข้อมูลโมเดล Ollama อยู่ที่ `/home/soravit/pentagi/ollama-data`
- ไฟล์ `.env` และ secrets อยู่บนเครื่องปลายทางเท่านั้น ห้ามคัดลอกหรือ commit secrets เข้า workspace

## สถานะที่ตรวจยืนยัน

- PentAGI, Ollama, pgvector, scraper และ pgexporter ทำงาน
- Ollama และ pgvector รายงานสถานะ healthcheck เป็น `healthy`
- เว็บ PentAGI ตอบ HTTP 200 และตรวจสอบ TLS สำเร็จเมื่อใช้ CA ที่สร้างให้บริการนี้
- เว็บผูกพอร์ตไว้ที่ `127.0.0.1:8443`; ไม่ได้เปิดพอร์ตหน้าเว็บสู่เครือข่ายภายนอกโดยตรง
- ใช้ URL `https://pentagi.local:8443` ซึ่งตรงกับชื่อใน certificate SAN; อย่าเข้า `https://localhost:8443` เพราะชื่อนั้นไม่อยู่ใน SAN
- ยังไม่ได้เริ่ม Flow หรือสแกนเป้าหมายใด ๆ

## เข้าใช้งานจากเครื่องผู้ใช้

ทำตามขั้นตอนบนเครื่องที่จะเปิด browser:

1. เพิ่มรายการนี้ในไฟล์ hosts ของเครื่องผู้ใช้ เพื่อให้ชื่อใน certificate ชี้มายังปลายทาง SSH tunnel:

   ```text
   127.0.0.1 pentagi.local
   ```

2. ดึง public CA certificate จากเครื่อง `kalla` แล้วเพิ่มเข้า trust store ของ browser หรือระบบปฏิบัติการ:

   ```sh
   ssh soravit@kalla 'docker exec pentagi cat /opt/pentagi/ssl/service_ca.crt' > pentagi-service-ca.crt
   ```

   certificate นี้เป็น CA สำหรับบริการ PentAGI ที่ติดตั้งไว้ ห้ามข้ามการตรวจ certificate แทนการ trust CA

3. เปิด SSH tunnel ค้างไว้ใน terminal:

   ```sh
   ssh -L 8443:127.0.0.1:8443 soravit@kalla
   ```

   หากไม่มี SSH alias `kalla` ให้ใช้ `soravit@172.30.222.240`

4. เปิด `https://pentagi.local:8443` ใน browser

คู่มือติดตั้งทางการระบุบัญชีเริ่มต้นสำหรับ fresh install ไว้ ให้เปลี่ยนรหัสผ่านทันทีหลังเข้าสู่ระบบครั้งแรก: [คู่มือติดตั้งและตั้งค่า PentAGI](https://github.com/vxcontrol/pentagi/blob/main/examples/guides/installation_configuration.md)

## หมายเหตุการประมวลผล

เครื่องมี NVIDIA GPU แต่ยังไม่ได้กำหนด NVIDIA Container Toolkit/runtime สำหรับ Docker; ให้ถือว่า Ollama ประมวลผลบน CPU จนกว่าจะตั้งค่า GPU passthrough เพิ่ม

## คำสั่งดูสถานะบริการ

```sh
cd /home/soravit/pentagi
docker compose -p penhub-pentagi \
  -f docker-compose.yml \
  -f docker-compose.local-llm.yml ps
```

## Flow สำหรับวิเคราะห์โจทย์ CTF (`penhub-ctf`)

สร้าง Flow ผ่าน GraphQL API `createFlow(modelProvider, input)` ของ PentAGI (ตาม README ทางการ ตรวจเมื่อ 2026-10-06) โดยเติมโจทย์ลงแม่แบบ `ctf/prompt-template.md`

ขั้นตอน:
1. `./penhub-kalla start` ให้ tunnel และเว็บพร้อม
2. สร้าง API token ที่ Settings -> API Tokens (แสดงครั้งเดียว) บันทึกลง `~/.config/penhub/pentagi-token` แล้ว `chmod 600` (อยู่นอก repo)
3. ดู prompt ก่อนส่งจริงด้วย `--dry-run`, แล้วสร้าง Flow:

```sh
./penhub-ctf new --name "ชื่อโจทย์" --category crypto --desc-file โจทย์.txt --attach chall.py --dry-run
./penhub-ctf new --name "ชื่อโจทย์" --category web --desc-file โจทย์.txt --target http://lab-host:8080
./penhub-ctf list
```

ข้อควรรู้:
- ไม่ใส่ `--target` = วิเคราะห์ offline จากข้อความ/ไฟล์ที่แนบเท่านั้น; ใส่ `--target` เฉพาะ host ของการแข่ง/lab ที่อนุญาตให้ทดสอบ
- ไฟล์แนบถูกฝังใน prompt (จำกัดไฟล์ข้อความไม่เกิน 20 KB) ไม่ใช่การอัปโหลดไฟล์เข้า container
- scope ใน prompt เป็นคำสั่งแก่ agent ไม่ใช่ network enforcement
- ชื่อ provider เริ่มต้นคือ `ollama` (ตั้งทับด้วย `--provider` หรือ `PENTAGI_PROVIDER`) ยังไม่ได้ยืนยันกับ instance นี้ ดูชื่อที่ถูกต้องในหน้าเลือก provider ของ UI
- ยังไม่ได้ทดสอบเรียก API จริง (ยังไม่มี token) ทดสอบแล้วเฉพาะ `--dry-run`
- `llama3.1:8b` ขนาดเล็ก อาจวางแผน/เรียกเครื่องมือได้ไม่ดีกับโจทย์ยาก
