# 01 Senior Project Analysis

# วิเคราะห์โปรเจกต์รุ่นพี่: P'Yui GPT

## 1. ชื่อโปรเจกต์รุ่นพี่

ชื่อโปรเจกต์รุ่นพี่คือ **P'Yui GPT**

โปรเจกต์นี้เป็นระบบผู้ช่วยภาษาไทยแบบ Open Source ที่พัฒนาขึ้นเพื่อช่วยให้การประมวลผลภาษาไทยและการตอบคำถามจากเอกสารมีความง่าย เป็นธรรมชาติ และแม่นยำมากขึ้น โดยระบบนำเทคนิคด้าน Vector Database, Embedding Model, Retrieval-Augmented Generation หรือ RAG และ Large Language Model มาใช้งานร่วมกัน

ระบบนี้ขับเคลื่อนด้วย **Typhoon API** และออกแบบให้สามารถตอบคำถามจากข้อมูลที่จัดเก็บไว้ในฐานข้อมูลเวกเตอร์ได้ โดยมีการค้นหาข้อมูลที่เกี่ยวข้องก่อน แล้วจึงนำข้อมูลนั้นไปประกอบการสร้างคำตอบ

---

## 2. วัตถุประสงค์ของระบบ

วัตถุประสงค์หลักของระบบ P'Yui GPT คือการพัฒนาระบบผู้ช่วยภาษาไทยที่สามารถตอบคำถามจากเอกสารหรือข้อมูลที่มีอยู่ได้อย่างถูกต้องและเกี่ยวข้องกับคำถามของผู้ใช้

วัตถุประสงค์ของระบบสามารถสรุปได้ดังนี้

* เพื่อสร้างระบบแชทบอทภาษาไทยที่สามารถตอบคำถามได้อย่างเป็นธรรมชาติ
* เพื่อใช้ฐานข้อมูลเวกเตอร์ในการจัดเก็บและค้นคืนข้อมูลจากเอกสาร
* เพื่อใช้โมเดล Embedding ที่รองรับหลายภาษาและเหมาะกับภาษาไทย
* เพื่อเพิ่มความแม่นยำของการค้นหาด้วยกระบวนการ Retrieval และ Re-ranking
* เพื่อเชื่อมต่อกับ Typhoon API หรือ OpenAI-compatible endpoint สำหรับสร้างคำตอบ
* เพื่อให้บริการผ่าน RESTful API ที่สามารถนำไปต่อยอดกับระบบอื่นได้
* เพื่อให้ระบบสามารถ Deploy ได้ง่ายผ่าน Docker Compose

---

## 3. โครงสร้างโมดูลหลัก

จาก README ของโปรเจกต์ ระบบมีโครงสร้างหลักดังนี้

```text
pyui-gpt/
├── app/
│   ├── index_docs.py
│   ├── query_rag_using_openai.py
│   └── web.py
├── docs/
├── .env
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

### 3.1 `app/web.py` — Main Flask Application

ไฟล์ `web.py` เป็นไฟล์หลักของระบบ API ทำหน้าที่เปิดบริการ Web API ด้วย Flask โดยรองรับ endpoint หลัก เช่น

* `/health` สำหรับตรวจสอบสถานะระบบ
* `/index` สำหรับเพิ่มเอกสารเข้าสู่ระบบ
* `/search` สำหรับค้นหาเอกสารที่เกี่ยวข้อง
* `/completions` สำหรับถามคำถามและรับคำตอบจากระบบ RAG
* `/metrics` สำหรับดูข้อมูลสถิติของระบบ

ส่วนนี้ถือเป็นแกนกลางของระบบ เพราะเป็นจุดที่รับ request จากผู้ใช้ แล้วส่งต่อไปยังส่วนค้นหาเอกสารและโมเดลภาษา

---

### 3.2 `app/index_docs.py` — Document Indexing Script

ไฟล์ `index_docs.py` ใช้สำหรับนำเอกสารเข้าสู่ระบบ โดยจะอ่านไฟล์จากโฟลเดอร์ `docs/` แล้วนำไปผ่านกระบวนการเตรียมข้อมูลก่อนจัดเก็บลงฐานข้อมูลเวกเตอร์

หน้าที่ของไฟล์นี้ ได้แก่

* อ่านไฟล์เอกสาร เช่น `.txt`, `.md`, `.pdf`
* แบ่งเอกสารออกเป็น chunk
* สร้าง embedding ของแต่ละ chunk
* จัดเก็บ embedding ลงใน Milvus
* บันทึก metadata ของเอกสารเพื่อใช้ในการค้นคืนข้อมูล

ค่าเริ่มต้นของการแบ่งเอกสารคือ chunk ขนาดประมาณ **1000 characters** และมี overlap ประมาณ **200 characters**

---

### 3.3 `app/query_rag_using_openai.py` — OpenAI Client Example

ไฟล์นี้เป็นตัวอย่างการ query ระบบ RAG ผ่าน OpenAI-compatible client โดยระบบออกแบบให้สามารถใช้ client แบบเดียวกับ OpenAI ได้ แต่เปลี่ยน base URL ให้ชี้มายังระบบของ P'Yui GPT

จุดเด่นคือทำให้การเรียกใช้งานระบบง่ายขึ้น เพราะสามารถใช้รูปแบบคำสั่งคล้ายกับ OpenAI API ได้

---

### 3.4 `docs/` — Document Storage

โฟลเดอร์ `docs/` ใช้เก็บเอกสารตัวอย่างหรือเอกสารที่ต้องการนำเข้าสู่ระบบ

ไฟล์ที่รองรับ ได้แก่

* `.txt`
* `.md`
* `.pdf`

เอกสารในโฟลเดอร์นี้จะถูกนำไปประมวลผลด้วย `index_docs.py` เพื่อสร้าง embedding และจัดเก็บลงใน Milvus

---

### 3.5 `.env` — Environment Configuration

ไฟล์ `.env` ใช้เก็บค่าการตั้งค่าของระบบ เช่น API Key, URL ของโมเดล, ชื่อโมเดล และค่าการเชื่อมต่อ Milvus

ตัวแปรสำคัญ ได้แก่

| Variable          | ความหมาย                                     |
| ----------------- | -------------------------------------------- |
| `MILVUS_HOST`     | Host ของ Milvus                              |
| `MILVUS_PORT`     | Port ของ Milvus                              |
| `OPENAI_API_KEY`  | API Key สำหรับ Typhoon/OpenAI-compatible API |
| `OPENAI_BASE_URL` | URL ของ API ที่ใช้เรียกโมเดล                 |
| `OPENAI_MODEL`    | ชื่อโมเดลที่ใช้                              |
| `SYSTEM_PROMPT`   | System prompt สำหรับกำหนดบทบาทของระบบ        |

---

### 3.6 `docker-compose.yml` — Container Orchestration

ไฟล์ `docker-compose.yml` ใช้สำหรับรัน service หลายตัวพร้อมกัน ได้แก่ Web API, Milvus, etcd, MinIO และ Attu

การใช้ Docker Compose ทำให้ผู้ใช้สามารถเปิดระบบทั้งหมดได้ด้วยคำสั่งเดียว

```bash
docker-compose up -d
```

---

## 4. Tech Stack ที่ใช้

จาก README ของระบบ สามารถสรุป Tech Stack ได้ดังนี้

| ส่วนของระบบ               | เทคโนโลยีที่ใช้                                         |
| ------------------------- | ------------------------------------------------------- |
| Programming Language      | Python                                                  |
| Backend API               | Flask                                                   |
| API Style                 | RESTful API                                             |
| LLM Provider              | Typhoon API / OpenAI-compatible endpoint                |
| Default Model             | `typhoon-v2.1-12b-instruct`                             |
| Embedding Model           | `BAAI/bge-m3`                                           |
| Vector Database           | Milvus                                                  |
| Object Storage for Milvus | MinIO                                                   |
| Metadata Store for Milvus | etcd                                                    |
| Milvus Web UI             | Attu                                                    |
| Deployment                | Docker Compose                                          |
| Document Indexing         | `index_docs.py`                                         |
| Retrieval Method          | Vector Search + Re-ranking                              |
| Security                  | API Key Authentication, Rate Limiting, Input Validation |
| Monitoring                | `/health`, `/metrics`                                   |

ระบบนี้มีการใช้ **BAAI/bge-m3** เป็น multilingual embedding model ซึ่งเหมาะกับการทำ embedding ข้อความหลายภาษา รวมถึงภาษาไทย และใช้ **Milvus** เป็นฐานข้อมูลเวกเตอร์สำหรับจัดเก็บและค้นคืน embedding ของเอกสาร

---

## 5. วิธีรันระบบเดิม

จาก README ขั้นตอนการรันระบบเดิมมีดังนี้

### 5.1 เตรียมเครื่อง

เครื่องที่ใช้ควรมีคุณสมบัติเบื้องต้นดังนี้

* ติดตั้ง Docker
* ติดตั้ง Docker Compose
* มี RAM อย่างน้อย 4GB
* Port `5001` ต้องว่าง

---

### 5.2 Clone Repository

```bash
git clone https://github.com/pyui-gpt.git
cd pyui-gpt
```

---

### 5.3 ตั้งค่า API Credentials

คัดลอกไฟล์ตัวอย่าง `.env`

```bash
cp .env.example .env
```

จากนั้นแก้ไขไฟล์ `.env` โดยใส่ API Key และค่าที่เกี่ยวข้อง เช่น

```env
OPENAI_API_KEY=your-api-key-here
OPENAI_BASE_URL=https://api.opentyphoon.ai/v1
OPENAI_MODEL=typhoon-v2.1-12b-instruct
```

---

### 5.4 รันระบบด้วย Docker Compose

```bash
docker-compose up -d
```

เมื่อรันสำเร็จ API จะเปิดใช้งานที่

```text
http://localhost:5001
```

---

### 5.5 ตรวจสอบสถานะระบบ

ใช้คำสั่ง

```bash
curl http://localhost:5001/health
```

ถ้าระบบทำงานปกติ แสดงว่า service หลักสามารถใช้งานได้แล้ว

---

### 5.6 เพิ่มเอกสารเข้าสู่ระบบ

นำไฟล์เอกสารไปไว้ในโฟลเดอร์ `docs/` จากนั้นรันคำสั่ง

```bash
python app/index_docs.py
```

ระบบจะทำการแบ่งเอกสาร สร้าง embedding และบันทึกข้อมูลลงใน Milvus

---

### 5.7 ทดสอบถามคำถาม

ตัวอย่างการเรียกใช้งานผ่าน `/completions`

```bash
curl -X POST http://localhost:5001/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "สวัสดี",
    "max_tokens": 1024,
    "temperature": 0.7
  }'
```

---

## 6. จุดแข็งของระบบเดิม

จุดแข็งของระบบ P'Yui GPT มีดังนี้

### 6.1 ใช้ Vector Database

ระบบใช้ Milvus สำหรับจัดเก็บและค้นคืน document embeddings ทำให้สามารถค้นหาข้อมูลจากเอกสารจำนวนมากได้มีประสิทธิภาพ

### 6.2 ใช้ Embedding Model ที่รองรับหลายภาษา

ระบบใช้ `BAAI/bge-m3` ซึ่งเป็น multilingual embedding model ทำให้เหมาะกับการประมวลผลข้อความภาษาไทยและข้อมูลหลายภาษา

### 6.3 มีระบบ Retrieval แบบ 2 ขั้นตอน

ระบบมี Advanced Retrieval โดยใช้ vector search ร่วมกับ re-ranking ช่วยให้ข้อมูลที่ดึงมาใช้ตอบคำถามมีความเกี่ยวข้องกับคำถามมากขึ้น

### 6.4 เชื่อมต่อ LLM ได้ผ่าน OpenAI-compatible Endpoint

ระบบสามารถเชื่อมต่อกับ Typhoon API และ endpoint ที่ใช้รูปแบบใกล้เคียงกับ OpenAI ได้ ทำให้การเปลี่ยนโมเดลหรือทดสอบโมเดลอื่นทำได้สะดวก

### 6.5 มี RESTful API

ระบบให้บริการผ่าน Flask-based RESTful API จึงสามารถนำไปเชื่อมต่อกับหน้าเว็บ แอปพลิเคชัน หรือระบบอื่นได้ง่าย

### 6.6 มี Docker Support

ระบบรองรับ Docker Compose ทำให้ติดตั้งและรันระบบง่ายขึ้น ลดปัญหาเรื่อง dependency และ environment

### 6.7 มีระบบ Security และ Monitoring

ระบบมี API Key Authentication, Rate Limiting, Input Validation และ endpoint สำหรับดูสถานะระบบ เช่น `/health` และ `/metrics`

---

## 7. จุดที่เราจะศึกษาเพิ่มเติม

จากระบบเดิม มีประเด็นที่เราควรศึกษาเพิ่มเติมดังนี้

### 7.1 ศึกษาการทำงานของ RAG Pipeline

ต้องดูว่าระบบรับคำถาม ค้นหาเอกสาร ทำ re-ranking และส่งข้อมูลให้โมเดลตอบอย่างไร

### 7.2 ศึกษาการใช้ Milvus

ต้องศึกษาโครงสร้างการเก็บข้อมูลใน Milvus เช่น collection, vector field, metadata และวิธีค้นคืนข้อมูล

### 7.3 ศึกษา Embedding Model `BAAI/bge-m3`

ต้องทดสอบว่า embedding model ตัวนี้เหมาะกับภาษาไทยมากน้อยแค่ไหน และเปรียบเทียบกับ embedding model อื่นได้หรือไม่

### 7.4 ศึกษา Typhoon API

ต้องดูว่าโมเดล `typhoon-v2.1-12b-instruct` ให้คำตอบภาษาไทยดีแค่ไหน มีความเร็วและค่าใช้จ่ายอย่างไร และสามารถเปรียบเทียบกับโมเดล Open Source ที่รันเองได้หรือไม่

### 7.5 ศึกษาการเพิ่มเอกสารเข้าสู่ระบบ

ต้องดูว่าการเพิ่มเอกสารผ่าน `index_docs.py` สะดวกแค่ไหน และถ้าจะทำหน้า Admin ให้ผู้ดูแลเพิ่มเอกสารได้เองต้องปรับส่วนใดบ้าง

### 7.6 ศึกษาระบบ API

ต้องทดสอบ endpoint หลัก เช่น `/index`, `/search`, `/completions`, `/health` และ `/metrics` เพื่อดูว่าระบบทำงานครบถ้วนหรือไม่

### 7.7 ศึกษาข้อจำกัดของระบบเดิม

ควรตรวจสอบเรื่อง RAM, ความเร็วในการตอบ, ความแม่นยำของการค้นหา, การรองรับไฟล์ PDF และข้อจำกัดของ API Key หรือ Rate Limit

---

## 8. จุดที่เราจะเปรียบเทียบกับงานของเรา

เราจะใช้ระบบรุ่นพี่เป็น baseline เพื่อเปรียบเทียบกับระบบที่เราจะพัฒนา โดยมีหัวข้อเปรียบเทียบดังนี้

| หัวข้อเปรียบเทียบ | ระบบรุ่นพี่                                                | ระบบของเรา                                        |
| ----------------- | ---------------------------------------------------------- | ------------------------------------------------- |
| ชื่อระบบ          | P'Yui GPT                                                  | ระบบแชทบอทที่เราพัฒนาต่อยอด                       |
| Backend           | Flask API                                                  | อาจใช้ Flask/FastAPI หรือโครงสร้างใหม่            |
| LLM               | Typhoon API `typhoon-v2.1-12b-instruct`                    | ทดลองโมเดล Open Source หลายขนาด                   |
| Embedding Model   | `BAAI/bge-m3`                                              | เปรียบเทียบกับ embedding model อื่น               |
| Vector Database   | Milvus                                                     | ใช้ Milvus เดิมหรือทดลอง Vector DB อื่น           |
| Retrieval         | Vector Search + Re-ranking                                 | ทดลองปรับ Retrieval หรือใช้แนวทางใหม่             |
| Document Indexing | ผ่าน `index_docs.py`                                       | เพิ่มหน้าต่างสำหรับ Admin จัดการเอกสาร            |
| API               | `/health`, `/index`, `/search`, `/completions`, `/metrics` | ออกแบบ API หรือ UI ให้ใช้งานง่ายขึ้น              |
| Deployment        | Docker Compose                                             | ใช้ Docker Compose ต่อหรือปรับให้เหมาะกับระบบใหม่ |
| Monitoring        | มี `/health` และ `/metrics`                                | อาจเพิ่ม logging หรือ dashboard เพิ่มเติม         |
| Security          | API Key, Rate Limiting, Input Validation                   | ศึกษาและปรับให้เหมาะกับผู้ใช้งานจริง              |
| ความเร็ว          | ต้องทดสอบจากระบบเดิม                                       | เปรียบเทียบเวลาในการตอบของแต่ละโมเดล              |
| ความแม่นยำ        | ใช้ผลจากระบบเดิมเป็น baseline                              | ทดสอบด้วยชุดคำถามเดียวกัน                         |

---
