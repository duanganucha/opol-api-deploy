# คู่มือการติดตั้งและใช้งาน OPOL-API

โปรเจกต์นี้บรรจุไฟล์ที่จำเป็นสำหรับการรันระบบ **OPOL-API** โดยใช้ Docker เท่านั้น ซึ่งจะทำการดาวน์โหลดไฟล์โปรแกรมจาก GitHub Container Registry (GHCR) โดยตรง ทำให้คุณสามารถรันระบบได้ทันทีโดยไม่ต้องเข้าถึงซอร์สโค้ด

OPOL-API ต้อง **ทำงานตลอด 24 ชั่วโมง** เพื่อส่งข้อมูลห้องคลอดไปศูนย์ข้อมูลทุก 15 นาที — ติดตั้งเสร็จแล้วอย่าลืม [ข้อ 6](#6-ตั้งค่าให้ทำงานตลอดเวลา-สำคัญ)

## สิ่งที่ต้องเตรียม (Prerequisites)

- **Docker**
  - **Linux (แนะนำ)**: ติดตั้ง Docker Engine — ทำงานเป็น service ของระบบ เปิดเองตอนบูตโดยไม่ต้อง login
  - **Windows**: ติดตั้ง Docker Desktop (ใช้ WSL2) — ต้องตั้งค่าเพิ่มตาม [ข้อ 6](#6-ตั้งค่าให้ทำงานตลอดเวลา-สำคัญ)
- **Docker Compose**: ปกติจะมาพร้อมกับ Docker อยู่แล้ว
- ข้อมูลเชื่อมต่อฐานข้อมูล HIS ของโรงพยาบาล (IP, user, password) และ **รหัสวอร์ดห้องคลอด** ใน HIS

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
> ไฟล์ `.env` มีรหัสผ่านจริง — **ห้าม commit / ห้ามส่งต่อ**

### 3. แก้ไขข้อมูลในไฟล์ .env
เปิดไฟล์ `.env` ด้วยโปรแกรมแก้ไขข้อความ (เช่น Notepad, VS Code, หรือ nano) และแก้ไขค่าต่อไปนี้ให้ตรงกับระบบของคุณ:

| ค่า | ใส่อะไร | ข้อควรระวัง |
|---|---|---|
| `DB_HOST` | IP ของเครื่องฐานข้อมูล HIS | ⚠️ **ถ้า HIS อยู่เครื่องเดียวกับที่รัน Docker ห้ามใส่ `127.0.0.1`** — ภายใน container หมายถึงตัว container เอง ให้ใส่ `host.docker.internal` (Docker Desktop) หรือ IP จริงของเครื่อง |
| `DB_PORT` | พอร์ตฐานข้อมูล | MySQL `3306` / PostgreSQL `5432` |
| `DB_USER` / `DB_PASSWORD` | บัญชีเข้าฐานข้อมูล HIS | แนะนำบัญชีที่ **อ่านได้อย่างเดียว** |
| `DB_NAME` | ชื่อฐานข้อมูลที่ใช้งาน | |
| `HIS` | `HIMPRO`, `HOSXP_MYSQL` หรือ `HOSXP_PGSQL` | ต้องตรงกับ HIS ของโรงพยาบาล |
| `CODE_ROOM` | รหัสวอร์ดห้องคลอดใน HIS (เช่น `LR1`, `WARD4`) | **ใส่ผิด = ไม่มีผู้ป่วยส่งเข้าระบบเลย** |
| `HN_LENGTH` | ความยาว HN ของโรงพยาบาล | |
| `TOKEN_KEY` / `JWT_SECRET` | ค่าสุ่มยาว ๆ **ไม่ซ้ำกับโรงพยาบาลอื่น** | ห้ามใช้ค่าตัวอย่าง — สร้างด้วย `openssl rand -hex 32` |
| `API_ENDPOINT` | ตามที่ศูนย์ข้อมูลกำหนด | |

### 4. สั่งรันระบบ
เปิด Terminal หรือ Command Prompt ในโฟลเดอร์นั้นแล้วรันคำสั่ง:
```bash
docker-compose up -d
```
*ระบบจะเริ่มดาวน์โหลด Image จาก GitHub และรันขึ้นมาให้โดยอัตโนมัติ*

> ต้องมี **`-d`** เสมอ — ถ้าไม่ใส่ ระบบจะผูกกับหน้าต่าง Terminal ปิดหน้าต่างแล้ว API หยุดทันที

### 5. การตั้งค่าผ่านหน้าจอ (UI Configuration)
นอกจากการแก้ไฟล์ `.env` แล้ว คุณยังสามารถตั้งค่าบางส่วนผ่านหน้าจอได้ที่:
- **URL**: `http://localhost:3000/config-page`

### 6. ตั้งค่าให้ทำงานตลอดเวลา (สำคัญ)

`docker-compose.yml` ตั้ง `restart: always` และ `autoheal` ไว้แล้ว — container ดับเมื่อไหร่ Docker จะเปิดให้ใหม่เอง
**แต่ทำงานได้ก็ต่อเมื่อตัว Docker เปิดอยู่**

- **Linux (Docker Engine)**: สั่งครั้งเดียวให้ Docker เปิดเองตอนบูต
  ```bash
  sudo systemctl enable --now docker containerd
  ```
  ตรวจว่าตั้งแล้ว:
  ```bash
  systemctl is-enabled docker                                        # ต้องได้ enabled
  systemctl is-active docker                                         # ต้องได้ active
  docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' opol-api   # ต้องได้ always
  ```
- **Windows (Docker Desktop)**: Docker Desktop จะเริ่มทำงาน **หลังมีคน login เข้า Windows เท่านั้น**
  เครื่องรีบูต / ไฟดับ / Windows Update แล้วไม่มีใคร login = OPOL-API ไม่ทำงาน (เคยเกิดขึ้นจริง บางโรงพยาบาลเงียบไปหลายเดือน) ให้ตั้งค่าดังนี้:
  1. Docker Desktop → **Settings → General → ✅ Start Docker Desktop when you sign in**
  2. ตั้ง Windows ให้ **login อัตโนมัติ** หรืออย่า log off เครื่องนี้ (ควรเป็นเครื่องที่ใช้งานนี้โดยเฉพาะ)
     - กด `Win + R` → พิมพ์ `netplwiz` → เอาเครื่องหมายถูกออกจาก *Users must enter a user name and password to use this computer* → ใส่รหัสผ่าน
     - ถ้าไม่มีตัวเลือกนี้ ใช้โปรแกรม [Autologon](https://learn.microsoft.com/sysinternals/downloads/autologon) ของ Microsoft แทน
  3. **ปิด Sleep / Hibernate** (Settings → System → Power) — เครื่องหลับ = API หยุด
  4. ปิดการอัปเดต Docker Desktop อัตโนมัติ — ระหว่างอัปเดต Docker อาจหยุดรอให้คนกดยืนยัน

**ทดสอบ**: รีบูตเครื่อง 1 ครั้ง (`sudo reboot` บน Linux) โดยไม่ต้องแตะอะไร รอ 1–2 นาที แล้ว
- `docker ps` ต้องเห็น `opol-api` สถานะ `Up` เอง
- โรงพยาบาลขึ้น **ออนไลน์** เองที่หน้าศูนย์ข้อมูล (ด้านล่าง)

> ถ้าเคยสั่ง `docker-compose down` ไว้ ต้องสั่ง `docker-compose up -d` หนึ่งครั้งก่อน — ระบบถึงจะกลับมาเองหลังรีบูต

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
   ควรเห็น `เชื่อมต่อฐานข้อมูลสำเร็จ` และ `พบข้อมูลผู้ป่วย SQL Admit … ราย`

4. **เช็คกับศูนย์ข้อมูล**:
   ภายใน 15 นาทีหลังเปิดระบบ โรงพยาบาลของคุณต้องขึ้น **ออนไลน์** ที่
   https://opol-ssk.datainfo.cloud/online
   ถ้าขึ้น **ออฟไลน์** แปลว่าไม่ได้ส่งข้อมูลเกิน 2 ชั่วโมง — ตรวจตามข้อ 1–3 และ [ข้อ 6](#6-ตั้งค่าให้ทำงานตลอดเวลา-สำคัญ)

5. **การอัปเดตระบบ (Update)**:
   หากมีการอัปเดตเวอร์ชันใหม่ ให้รันคำสั่ง:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```
   ไฟล์ `.env` ไม่ถูกแตะ — ค่าตั้งค่าเดิมยังอยู่

6. **การหยุดระบบ (Stop)**:
   หากต้องการหยุดการทำงานทั้งหมด:
   ```bash
   docker-compose down
   ```
   > หลัง `down` ระบบ **จะไม่เปิดเอง** แม้รีบูตเครื่อง — ต้องสั่ง `docker-compose up -d` อีกครั้ง

---

## แก้ปัญหาที่พบบ่อย

> เครื่องที่ใช้ Docker รุ่นใหม่ ใช้ `docker compose` (มีช่องว่าง) แทน `docker-compose` — คำสั่งอื่นเหมือนกันทุกอย่าง

### `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`
บัญชีที่ใช้อยู่ไม่มีสิทธิ์สั่ง Docker (Docker ยังทำงานปกติ) เลือกทางใดทางหนึ่ง:
- ใส่ `sudo` นำหน้า เช่น `sudo docker ps`, `sudo docker-compose up -d`
- หรือเพิ่มบัญชีเข้ากลุ่ม docker (ครั้งเดียว ไม่ต้องพิมพ์ sudo อีก):
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker          # หรือ logout แล้ว login ใหม่
  docker ps
  ```
  > อยู่ในกลุ่ม docker = มีสิทธิ์เกือบเท่า root ของเครื่อง — เพิ่มเฉพาะบัญชีผู้ดูแลระบบ

ถ้าขึ้น `is not in the sudoers file` แปลว่าบัญชีนี้ไม่มีสิทธิ์ผู้ดูแลเครื่อง ต้องให้ผู้ที่มีสิทธิ์ root ทำให้

### `KeyError: 'ContainerConfig'` (มักเกิดหลังเปลี่ยนค่าใน .env หรือตอนอัปเดต)
```bash
docker-compose down --remove-orphans
docker-compose up -d
```

### `Table 'hosdata.sys_var' doesn't exist` / `Table 'hosdata.ipt' doesn't exist` และ log ขึ้น `PCU: hosxp_00`
ค่า **`HIS` ใน `.env` ไม่ตรงกับ HIS จริง** — เชื่อมฐานข้อมูลได้ แต่ค้นหาตารางของอีกระบบ
- ฐานข้อมูล `hosdata` = **HIMPRO** → ใช้ `HIS=HIMPRO`
- ค่าที่ใช้ได้มีแค่ `HIMPRO`, `HOSXP_MYSQL`, `HOSXP_PGSQL` — `HOSXP` เฉย ๆ ใช้ไม่ได้
- ถ้าไม่มีบรรทัด `HIS=` ระบบจะใช้ `HOSXP_MYSQL` ให้เอง

แก้แล้วต้องสั่ง `docker-compose up -d` (หรือ `down` แล้ว `up -d`) — `restart` เฉย ๆ ไม่อ่าน `.env` ใหม่
ใน log ต้องเห็นชื่อโรงพยาบาลจริง ไม่ใช่ `hosxp_00` / `himpro_00`

### เชื่อมต่อได้ แต่ `พบข้อมูลผู้ป่วย SQL Admit 0 ราย` ตลอด ทั้งที่มีผู้ป่วยนอนอยู่
`CODE_ROOM` ไม่ตรงกับรหัสวอร์ดห้องคลอดใน HIS — HIMPRO ดูรหัสได้จาก
```sql
SELECT roomcode, roomname FROM hos.roomno WHERE roomname LIKE '%คลอด%';
```

### เชื่อมฐานข้อมูลไม่ได้ (`connect ETIMEDOUT` / `ECONNREFUSED`) ทั้งที่ HIS อยู่เครื่องเดียวกัน
`DB_HOST=127.0.0.1` ภายใน container หมายถึงตัว container เอง — ใช้ `host.docker.internal` (Docker Desktop) หรือ IP จริงของเครื่อง

---

## ข้อมูลทางเทคนิค
- **Image**: `ghcr.io/duanganucha/opol-api:latest`
- **Owner**: duanganucha
- **Developer**: duanganucha

*หากมีข้อสงสัยเพิ่มเติม สามารถติดต่อทีมพัฒนาได้ทันทีครับ*
