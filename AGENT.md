# Project Context

## เป้าหมาย

โปรเจกต์นี้ใช้ศึกษา AI automation สำหรับงาน penetration testing โดยเริ่มจากการอ่านและทำความเข้าใจ [vxcontrol/pentagi](https://github.com/vxcontrol/pentagi) เอกสารสรุปภาษาไทยอยู่ใน `docs/pentagi-overview-th.md` ขณะนี้ workspace นี้เป็นพื้นที่บันทึกการศึกษา ไม่ใช่ source checkout ของ PentAGI

## วิธีทำงานในโปรเจกต์

- สื่อสารและจัดทำเอกสารเป็นภาษาไทย เว้นแต่ผู้ใช้ขอภาษาอื่น
- เมื่ออธิบายรายละเอียดของ PentAGI ให้ตรวจ README/เอกสารทางการของ revision ปัจจุบัน และบอกวันที่ตรวจเมื่อเนื้อหามีโอกาสเปลี่ยน
- แยกข้อเท็จจริงจาก repo ออกจากการตีความหรือข้อเสนอแนะให้ชัด พร้อมใส่ลิงก์แหล่งข้อมูลไว้ในเอกสาร
- อย่าสันนิษฐานว่าโปรเจกต์นี้ได้ fork, ติดตั้ง หรือเชื่อมต่อกับ PentAGI แล้ว
- รักษาเอกสารและบันทึกการศึกษาไว้ใต้ `docs/`; อัปเดตไฟล์นี้เมื่อเป้าหมายหรือขอบเขตของโปรเจกต์เปลี่ยน

## ขอบเขตความปลอดภัย

- ใช้คำแนะนำด้าน pentest กับระบบที่ผู้ใช้เป็นเจ้าของหรือมีหนังสืออนุญาตชัดเจนเท่านั้น
- สำหรับการทดลอง ให้แนะนำ lab/staging ที่แยกจาก production และ scope แบบจำกัด พร้อมระบุข้อห้ามและเงื่อนไขหยุด
- อย่าเริ่มสแกนหรือทดสอบเป้าหมายภายนอกโดยไม่มีการอนุญาตและ scope ที่ชัดเจน
- เน้นการศึกษา สถาปัตยกรรม การติดตั้งอย่างปลอดภัย การตรวจสอบผล และการลดความเสี่ยง
- ถือว่า Docker isolation เป็นมาตรการหนึ่งที่ต้องตรวจการตั้งค่า ไม่ใช่การรับประกันว่า host/เครือข่ายจะเข้าถึงไม่ได้
- ปกป้อง API keys, credentials, ข้อมูลส่วนบุคคล และข้อมูลลูกค้า; ห้ามใส่ secrets ลงเอกสารหรือ commit

## สถานะการติดตั้ง PentAGI

- ผู้ใช้เลือกเครื่องปลายทาง `soravit@kalla` และต้องการใช้ local LLM
- working directory บนเครื่องปลายทางคือ `/home/soravit/pentagi`; เก็บไฟล์ติดตั้งและข้อมูล Ollama ไว้ที่นั่น
- เครื่องมี Docker Compose, RAM 46 GB และ 24 CPU; ผู้ใช้เคลียร์พื้นที่บนเครื่องแล้ว
- ติดตั้งด้วย Compose ทางการและ local Ollama sidecar; `.env` สร้าง secrets บนเครื่องปลายทางและตั้ง permission `600` ห้ามคัดลอกค่า secret ออกจากเครื่อง
- โมเดล `llama3.1:8b-instruct-q8_0` (8.5 GB) และ `nomic-embed-text` (274 MB) ดาวน์โหลดและแสดงอยู่ใน Ollama แล้ว
- ตรวจเมื่อ 2026-09-19: PentAGI, Ollama, pgvector, scraper และ pgexporter ทำงาน; Docker รายงาน healthcheck ของ Ollama และ pgvector เป็น healthy ส่วน PentAGI, scraper และ pgexporter อยู่ในสถานะ running
- เว็บ PentAGI ผูกไว้เฉพาะ `127.0.0.1:8443`; URL/CORS ตั้งเป็น `https://pentagi.local:8443` ให้ตรงกับ SAN ใน certificate แล้ว ตรวจได้ HTTP 200 โดยยืนยัน TLS ผ่าน `service_ca.crt`
- การเข้า UI จากเครื่องผู้ใช้ต้องเปิด SSH tunnel `ssh -L 8443:127.0.0.1:8443 soravit@kalla`, ให้เครื่องผู้ใช้ resolve `pentagi.local` เป็น `127.0.0.1` และ trust CA จาก `/opt/pentagi/ssl/service_ca.crt`; อย่าข้ามการตรวจ certificate
- เครื่องมี NVIDIA GPU แต่ยังไม่ได้ตั้ง NVIDIA Container Toolkit/runtime ให้ Docker; คาดว่า Ollama จะประมวลผลด้วย CPU จนกว่าจะตั้งค่า GPU passthrough เพิ่ม
- TLS ไปยัง Docker Hub กลับมาใช้งานได้และดึง images สำเร็จ; ห้ามปิดการตรวจ TLS เพื่อเลี่ยงปัญหา registry
- ก่อนเริ่มการสแกน ให้กำหนด target, scope, หลักฐานอนุญาต, rate limit และเงื่อนไขหยุดให้ชัดเจน; ห้ามสแกนระบบภายนอกโดยไม่มีอนุญาต
- ห้ามคัดลอก secrets จากไฟล์ `.env` บนเครื่องปลายทางเข้า workspace หรือแชต
