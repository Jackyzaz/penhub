# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENT.md

## ลักษณะของ repo

ไม่ใช่ source checkout ของ PentAGI และไม่มี build/lint/test ที่นี่ เป็นพื้นที่บันทึกการศึกษา (`docs/`) บวกสคริปต์ควบคุม PentAGI ที่ติดตั้งอยู่บนเครื่องอื่น (`soravit@kalla`) บริบทโปรเจกต์ ขอบเขตความปลอดภัย และสถานะการติดตั้ง อยู่ใน `AGENT.md` (import ไว้ด้านบน) ส่วนขั้นตอนติดตั้ง/เข้าใช้งานอยู่ใน `docs/pentagi-deployment-kalla.md`

## คำสั่งที่ใช้บ่อย

```sh
./penhub-kalla start    # docker compose up -d บน kalla + เปิด SSH tunnel + รอเว็บ HTTP 200 + เปิด Chromium
./penhub-kalla stop     # ปิด tunnel ของสคริปต์ + docker compose stop (ไม่ใช่ down; ข้อมูลและโมเดลยังอยู่)
./penhub-kalla status   # compose ps + สถานะ tunnel + ตรวจเว็บ (exit 0 = พร้อมทั้งหมด)
NO_BROWSER=1 ./penhub-kalla start   # ไม่เปิด browser
./penhub-ctf new --name N --desc-file F [--target URL] [--dry-run]   # สร้าง Flow วิเคราะห์โจทย์ CTF ผ่าน API (ดู docs/pentagi-deployment-kalla.md)
```

## สถาปัตยกรรม (ต้องอ่านหลายที่ถึงจะเห็นภาพ)

PentAGI รันบน `kalla` ไม่ใช่เครื่องนี้ การตรวจสถานะด้วย `docker ps` บนเครื่องนี้จะไม่เห็น container ของมัน ต้องผ่าน SSH เท่านั้น (`penhub-kalla` ทำให้แล้ว) บน `kalla` ใช้ Compose project ชื่อ `penhub-pentagi` ที่ `/home/soravit/pentagi` ด้วย `docker-compose.yml` + `docker-compose.local-llm.yml` (Ollama sidecar) ทุกพอร์ตผูกกับ `127.0.0.1` ของ `kalla` เท่านั้น

เส้นทางเข้า UI จากเครื่องนี้ต้องครบสามอย่างพร้อมกัน ขาดอย่างใดอย่างหนึ่งเว็บจะเข้าไม่ได้หรือเบราว์เซอร์ฟ้อง not secure:
1. SSH tunnel `127.0.0.1:8443` -> `kalla:8443` (สคริปต์ใช้ control socket `$XDG_RUNTIME_DIR/penhub-kalla.sock` เพื่อปิดเฉพาะ tunnel ของตัวเอง)
2. `/etc/hosts` ต้องมี `127.0.0.1 pentagi.local` (สคริปต์ตรวจแต่ไม่แก้ให้ ต้องใช้ sudo)
3. CA ของ PentAGI (`pentagi-service-ca.crt` ในราก repo) ต้อง trust ใน Chromium ผ่าน `~/.pki/nssdb` ไม่ใช่แค่ `update-ca-certificates`

## ข้อควรระวัง

- ต้องเข้าด้วย `https://pentagi.local:8443` เท่านั้น `localhost:8443` ไม่อยู่ใน SAN ของ certificate
- ตรวจ TLS ด้วย CA ไฟล์นี้เสมอ (`curl --cacert pentagi-service-ca.crt`) อย่าใช้ `-k`
- `pentagi-service-ca.crt` เป็น public CA ไม่ใช่ secret แต่ `penhub-kalla` อ้างพาธนี้ อย่าย้ายหรือลบ
- Ollama รันบน CPU (ยังไม่ได้ตั้ง NVIDIA Container Toolkit) ตอบช้าเป็นเรื่องปกติ
