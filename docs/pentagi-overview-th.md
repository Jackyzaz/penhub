# PentAGI: หลักการและวิธีใช้งาน

เอกสารนี้สรุปโปรเจกต์ [vxcontrol/pentagi](https://github.com/vxcontrol/pentagi) จาก README และคู่มือทางการบน `main` ที่ตรวจเมื่อ 19 กันยายน 2026 รายละเอียดการติดตั้งและตัวเลือกอาจเปลี่ยนตามรุ่น ให้ยึดคู่มือของรุ่นที่จะติดตั้งเป็นหลัก

## PentAGI คืออะไร

PentAGI เป็นระบบ AI agent แบบ self-hosted ที่จัดการงานทดสอบความปลอดภัยผ่านเว็บ UI ใช้ LLM วางแผนและประสาน agent หลายบทบาท แล้วเรียกใช้เครื่องมือใน environment ที่จัดเตรียมไว้ เช่น terminal, file operations, web/search tools และเครื่องมือ security testing จุดประสงค์คือช่วยทำ workflow หลายขั้นตอนให้ต่อเนื่อง ตั้งแต่เก็บข้อมูล วางแผน ตรวจสอบ ไปจนถึงสรุปผล ไม่ใช่เพียงหน้าแชตที่ตอบคำถาม

ควรมองผลลัพธ์เป็นข้อค้นพบที่ต้องตรวจซ้ำโดยผู้ทดสอบ ไม่ใช่หลักฐานยืนยันช่องโหว่โดยอัตโนมัติ โมเดลอาจเข้าใจเป้าหมายผิด เลือกเครื่องมือไม่เหมาะ หรือสรุปผลคลาดเคลื่อนได้

## หลักการทำงาน

### 1. แบ่งงานเป็นลำดับชั้น

แนวคิดหลักในระบบคือ `Flow → Task → Subtask → Action`:

- **Flow** คือชุดงานหรือ engagement หนึ่งชุด และเป็นที่เก็บบริบทการทำงาน
- **Task** คือวัตถุประสงค์ที่ผู้ใช้ระบุใน Flow
- **Subtask** คือขั้นตอนย่อยที่ระบบสร้างและปรับแผนตามผลที่ได้
- **Action** คือการค้นคว้า วิเคราะห์ หรือเรียกเครื่องมือระหว่างทำ Subtask

ผู้ใช้ติดตาม Flow และผลที่ agent ทำได้จาก UI; agent อาจหยุดรอข้อมูลเพิ่มเติมได้ การแบ่งงานและเรียกเครื่องมือยังอยู่ภายใต้การตีความของ LLM จึงควรกำหนดขอบเขตและเงื่อนไขหยุดให้ชัดตั้งแต่ต้น

### 2. ใช้ agent หลายบทบาท

มี agent สำหรับการประสานและงานเฉพาะทาง เช่น Primary/Assistant, Generator และ Refiner สำหรับวางหรือปรับแผน, Pentester สำหรับประเมินความปลอดภัย, Searcher/Enricher สำหรับค้นคว้า, Coder สำหรับงานโค้ด, Adviser สำหรับคำแนะนำ, Memorist สำหรับเรียกใช้ความรู้ และ Reporter สำหรับจัดทำรายงาน แต่ละบทบาทมี prompt และเครื่องมือที่เหมาะกับงานของตน

ชื่อและจำนวน agent อาจปรับเปลี่ยนระหว่างรุ่น ให้ตรวจเอกสาร Flow Execution ของรุ่นที่ใช้อยู่หากต้องการรายละเอียดภายใน

### 3. เรียกเครื่องมือผ่าน environment ที่แยกไว้

PentAGI จัดการการทำงานของ terminal และเครื่องมือผ่าน container ของ Docker; เอกสารสถาปัตยกรรมอธิบายการแยก container สำหรับงานทดสอบและ browser scraper รวมถึงการกำหนด timeout และทรัพยากร การแยกด้วย container ลดการปะปนกับ host แต่ไม่ใช่การรับประกันความปลอดภัยสมบูรณ์ การตั้งค่า Docker, network, capabilities และ worker มีผลต่อขอบเขตจริง

ควรติดตั้งบน VM/เครื่องทดสอบแยก จำกัดการเชื่อมต่อเครือข่ายให้เข้าถึงเฉพาะเป้าหมายที่อนุญาต และระวังการเปิด Docker socket หรือใช้ network mode ที่ให้ container เข้าถึง host/network ภายในได้

### 4. ใช้ LLM, embedding และ memory

ต้องตั้งค่า LLM provider อย่างน้อยหนึ่งรายก่อนใช้งาน ส่วน embedding ใช้ค้นหาความรู้และ memory เชิงความหมาย ระบบรองรับ provider หลายแบบ รวมถึงบริการ cloud และ endpoint แบบ local/OpenAI-compatible ตาม configuration ของรุ่นนั้น

ข้อมูลจาก Flow และคำสั่งที่ส่งให้ provider อาจออกจากเครื่องเมื่อใช้บริการ cloud; ตรวจนโยบายข้อมูลของ provider และหลีกเลี่ยงการใส่ secret, ข้อมูลส่วนบุคคล หรือข้อมูล production ลงใน prompt/ไฟล์โดยไม่จำเป็น ระบบอาจใช้ PostgreSQL/pgvector เก็บข้อมูลค้นคืน; Graphiti/Neo4j, Langfuse และ observability เป็นส่วนเสริมที่เปิดใช้ได้

## วิธีติดตั้งและเริ่มระบบ

คู่มือปัจจุบันแนะนำ **interactive installer** ซึ่งช่วยตรวจระบบ ตั้งค่า provider และเริ่ม stack; อีกทางเลือกคือ Docker Compose แบบ manual

ข้อกำหนดขั้นต่ำที่คู่มือระบุ: Docker และ Docker Compose (หรือ Podman), อย่างน้อย 2 vCPU, RAM 4 GB, พื้นที่ว่าง 20 GB และการเชื่อมต่ออินเทอร์เน็ตสำหรับดึง image/ติดต่อ LLM provider

### วิธีแนะนำ: Installer

ดาวน์โหลด installer จากหน้า [เอกสารและดาวน์โหลด PentAGI](https://github.com/vxcontrol/pentagi) ให้ตรงกับระบบปฏิบัติการและสถาปัตยกรรม จากนั้นแตกไฟล์และรันตามขั้นตอนในหน้าจอ installer ตัวอย่างคำสั่ง Linux amd64 ตามคู่มือทางการ:

```sh
mkdir -p pentagi && cd pentagi
wget -O installer.zip https://pentagi.com/downloads/linux/amd64/installer-latest.zip
unzip installer.zip
./installer
```

installer ต้องเข้าถึง Docker API; ทำตามคำแนะนำเรื่องสิทธิ์ของ installer ในคู่มือ และอย่าให้สิทธิ์ Docker แก่ผู้ใช้หรือ service ที่ไม่เชื่อถือ

### วิธี manual: Docker Compose

1. เตรียม working directory และใช้ `docker-compose.yml` กับ `.env.example` จาก revision เดียวกันของ repo ทางการ
2. กรอก provider key และค่าที่จำเป็นใน `.env`; ตั้ง `COOKIE_SIGNING_SALT`, รหัสผ่านฐานข้อมูล และ `PUBLIC_URL` ให้เหมาะกับสภาพแวดล้อม
3. เริ่ม stack ด้วย `docker compose up -d` แล้วติดตามความพร้อมด้วย `docker compose logs -f pentagi`
4. เปิด `https://localhost:8443` หรือ URL ที่กำหนดไว้

เก็บ `.env` และ API keys ให้เป็นความลับ อย่า commit เข้า Git หากใช้บริการภายนอก ตั้ง listener เป็น localhost ก่อน และอ่านคู่มือเรื่อง CORS, TLS และการเข้าถึงจากเครือข่ายก่อนเปิดให้ผู้ใช้อื่นเข้าถึง

### ตรวจค่าตั้งต้นก่อนใช้งาน

- ตั้งค่าและทดสอบ LLM provider อย่างน้อยหนึ่งราย; ตรวจความเข้ากันได้ด้วย `ctester` ตามคู่มือ
- ตั้งค่า embedding provider และตรวจด้วย `etester`
- ตั้งค่า Search provider หากจำเป็น; เป็นส่วนเสริม
- เปลี่ยนรหัสผ่าน admin เริ่มต้นทันทีหลัง login หากรุ่นนั้นยังสร้างบัญชี `admin@pentagi.com` / `admin`
- เปลี่ยน secrets และรหัสผ่านค่าเริ่มต้นก่อนเปิด instance ออกจาก localhost

ชื่อ environment variables, installer, บัญชีเริ่มต้น และวิธีทดสอบเปลี่ยนได้ตามรุ่น ให้ยืนยันกับ installation guide และ `.env.example` ของรุ่นนั้น

## วิธีสร้างงานทดสอบ (Flow)

1. เข้าหน้าเว็บ แล้วเลือก **Flows → New Flow**
2. เลือกโหมด:
   - **Automation** ให้ระบบดำเนินงานตามเป้าหมายที่กำหนดแบบอัตโนมัติ
   - **Assistant** สำหรับโต้ตอบและชี้นำทีละขั้น; เปิด **Use Agents** เมื่อต้องการให้ assistant มอบหมายงานย่อยให้ agent เฉพาะทาง
3. เลือก LLM provider สำหรับ Flow
4. เขียนคำสั่งเป็นภาษาธรรมชาติ โดยระบุเป้าหมาย ประเภทการประเมิน ขอบเขต/ข้อห้าม และรูปแบบผลลัพธ์
5. ตรวจสอบแผนและกิจกรรมระหว่างทำงาน หาก scope หรือการกระทำเริ่มออกนอกข้อตกลง ให้หยุด Flow และทบทวนก่อนเริ่มต่อ
6. อ่านรายงานและ log/tool calls แล้วตรวจยืนยันข้อค้นพบด้วยวิธีที่ได้รับอนุญาต เก็บเฉพาะหลักฐานที่จำเป็นต่อรายงาน

### แม่แบบคำสั่งที่ปลอดภัย

```text
ประเมิน [ระบบ/URL ที่ฉันเป็นเจ้าของหรือได้รับอนุญาตเป็นลายลักษณ์อักษร]
ขอบเขตที่อนุญาต: [hostname, path, IP หรือ environment]
เป้าหมาย: [ประเภทการทดสอบ]
นอกขอบเขต: [ระบบ/ข้อมูล/การกระทำที่ห้ามแตะ]
ข้อจำกัด: ห้าม DoS, brute force, persistence, social engineering,
การเปลี่ยนแปลงข้อมูลจริง หรือการดึงข้อมูลที่ไม่จำเป็น
หากจำเป็นต้องทำขั้นตอนที่มีผลกระทบสูง ให้หยุดและขออนุมัติจากผู้ดูแลก่อน
รายงานเฉพาะข้อค้นพบที่มีหลักฐาน พร้อมระดับความเชื่อมั่น ผลกระทบ
วิธีตรวจยืนยันอย่างปลอดภัย และคำแนะนำแก้ไข
```

เริ่มจาก staging หรือ lab ที่แยกจาก production และใช้เป้าหมายตัวอย่างที่ควบคุมได้ หากระบบต้องการการอนุมัติระหว่างทาง ให้ตอบเฉพาะเมื่อผู้รับผิดชอบ scope เป็นผู้อนุมัติ

## แนวทางประเมินผล

- ตรวจว่าข้อค้นพบมีหลักฐานจาก output/log จริง ไม่ใช่ข้อสันนิษฐานของ agent
- แยกผลยืนยันแล้วออกจากสิ่งที่ยังต้องตรวจเพิ่ม
- ทำซ้ำเฉพาะขั้นตอนที่ไม่ทำลายข้อมูลและยังอยู่ในขอบเขตที่อนุญาต
- บันทึกเวอร์ชัน PentAGI, LLM/provider, prompt, ขอบเขต และข้อจำกัดที่ใช้ เพื่อให้การประเมินทำซ้ำได้
- ใช้ human review ก่อนส่งรายงานหรือแก้ไขระบบ

## ข้อจำกัดและความเสี่ยงที่ควรรู้

- ความเป็นอัตโนมัติไม่รับประกันความถูกต้อง ความครบถ้วน หรือการไม่เกิด false positive/false negative
- LLM อาจสร้างแผนหรือเลือกเครื่องมือที่ไม่เหมาะกับระบบเป้าหมาย
- คำสั่งใน Flow อาจส่งผลต่อระบบจริง; ขอบเขตใน prompt เป็นคำสั่งแก่ agent ไม่ใช่ network enforcement โดยตัวมันเอง
- Docker isolation ขึ้นกับการตั้งค่า daemon, socket, capabilities, worker และเครือข่าย จึงไม่ควรมองว่า container ปลอดภัยจาก host ทุกกรณี
- ค่าใช้จ่ายและการเก็บข้อมูลขึ้นกับ LLM/search provider และบริการเสริมที่เปิดใช้
- ตรวจ LICENSE และ EULA ของ revision ที่ใช้งานก่อนนำไปใช้หรือแจกจ่าย

## แหล่งอ้างอิงทางการ

- [Repository และ README](https://github.com/vxcontrol/pentagi)
- [คู่มือติดตั้งและตั้งค่า](https://github.com/vxcontrol/pentagi/blob/main/examples/guides/installation_configuration.md)
- [Flow Execution และ agent/tool architecture](https://github.com/vxcontrol/pentagi/blob/main/backend/docs/flow_execution.md)
- [Configuration reference](https://github.com/vxcontrol/pentagi/blob/main/backend/docs/config.md)
- [Worker node และแนวทาง isolation](https://github.com/vxcontrol/pentagi/blob/main/examples/guides/worker_node.md)
- [ตัวอย่าง prompt สำหรับ web pentest](https://github.com/vxcontrol/pentagi/blob/main/examples/prompts/base_web_pentest.md)
- [LICENSE](https://github.com/vxcontrol/pentagi/blob/main/LICENSE) และ [EULA](https://github.com/vxcontrol/pentagi/blob/main/EULA.md)
