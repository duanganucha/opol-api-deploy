# คู่มือการติดตั้งและใช้งาน OPOL-API

โปรเจกต์นี้บรรจุไฟล์ที่จำเป็นสำหรับการรันระบบ **OPOL-API** โดยใช้ Docker เท่านั้น ซึ่งจะทำการดาวน์โหลดไฟล์โปรแกรมจาก GitHub Container Registry (GHCR) โดยตรง ทำให้คุณสามารถรันระบบได้ทันทีโดยไม่ต้องเข้าถึงซอร์สโค้ด

## สิ่งที่ต้องเตรียม (Prerequisites)

- **Docker**: ติดตั้งให้เรียบร้อย (แนะนำ Docker Desktop สำหรับ Windows/Mac)
- **Docker Compose**: ปกติจะมาพร้อมกับ Docker อยู่แล้ว

---

## ขั้นตอนการติดตั้งแบบละเอียด (Step-by-Step)

### 1. นำไฟล์ขึ้น Server/เครื่องรันงาน
คุณสามารถเลือกทำได้ 2 วิธี:
- **วิธีที่ 1 (ใช้ Git)**: รันคำสั่ง 
  ```bash
  git clone https://github.com/duanganucha/opol-api-deploy.git
  cd opol-api-deploy
  ```
- **วิธีที่ 2 (Copy ไฟล์)**: ก๊อปปี้ไฟล์ `docker-compose.yml` และ `.env.example` ไปวางในโฟลเดอร์ที่คุณสร้างขึ้นใหม่

### 2. ตั้งค่าสภาพแวดล้อม (Environment Variables)
ก๊อปปี้ไฟล์ตัวอย่างเพื่อสร้างไฟล์ตั้งค่าจริง:
```bash
cp .env.example .env
```
### His = HOSXP_PGSQL , HOSXP_MYSQL , HIMPRO
### 3. แก้ไขข้อมูลในไฟล์ .env
เปิดไฟล์ `.env` ด้วยโปรแกรมแก้ไขข้อความ (เช่น Notepad, VS Code, หรือ nano) และแก้ไขค่าต่อไปนี้ให้ตรงกับระบบของคุณ:
- **`DB_HOST`**: ใส่ IP ของฐานข้อมูล
- **`DB_USER` / `DB_PASSWORD`**: รหัสผ่านเข้าฐานข้อมูล
- **`DB_NAME`**: ชื่อฐานข้อมูลที่ใช้งาน
- **`HIS`**: ชื่อระบบ HIS ที่ใช้ (เช่น `HOSXP_MYSQL`, `HOSXP_PGSQL`)
- **`JWT_SECRET`**: กำหนดรหัสความปลอดภัยสำหรับ Token (ห้ามเว้นว่าง)

### 4. สั่งรันระบบ
เปิด Terminal หรือ Command Prompt ในโฟลเดอร์นั้นแล้วรันคำสั่ง:
```bash
docker-compose up -d
```
*ระบบจะเริ่มดาวน์โหลด Image จาก GitHub และรันขึ้นมาให้โดยอัตโนมัติ*

### 5. การตั้งค่าผ่านหน้าจอ (UI Configuration)
นอกจากการแก้ไฟล์ `.env` แล้ว คุณยังสามารถตั้งค่าบางส่วนผ่านหน้าจอได้ที่:
- **URL**: `http://localhost:3000/config-page`

---

## วิธีการตรวจสอบสถานะระบบ

1. **เช็คว่า Container รันอยู่หรือไม่**:
   ```bash
   docker ps
   ```
   คุณควรเห็น `opol-api` และ `autoheal` พร้อมสถานะ `Up` หรือ `healthy`

2. **เช็คหน้า Health Check**:
   เปิด Browser ไปที่: `http://localhost:3000/api/status/health`

3. **ดู Log การทำงาน**:
   ```bash
   docker logs -f opol-api
   ```

4. **การอัปเดตระบบ (Update)**:
   หากมีการอัปเดตเวอร์ชันใหม่ ให้รันคำสั่ง:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

5. **การหยุดระบบ (Stop)**:
   หากต้องการหยุดการทำงานทั้งหมด:
   ```bash
   docker-compose down
   ```

---

## ข้อมูลทางเทคนิค
- **Image**: `ghcr.io/duanganucha/opol-api:latest`
- **Owner**: duanganucha
- **Developer**: duanganucha

*หากมีข้อสงสัยเพิ่มเติม สามารถติดต่อทีมพัฒนาได้ทันทีครับ*
