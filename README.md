# 🏥 Hospital ERP - Docker Deployment

ระบบ ERP โรงพยาบาล (สำหรับใช้งานภายใน)
ติดตั้งและใช้งานผ่าน **Docker Compose** ได้ง่ายในไม่กี่ขั้นตอน

> **Proxmox v8 ขึ้นไป**

---

## ⚙️ ขั้นตอนการติดตั้ง

### 1️⃣ เปลี่ยนชื่อไฟล์ `.env-example` → `.env`

คัดลอกไฟล์ตัวอย่างแล้วเปลี่ยนชื่อเป็น `.env`

```bash
cp .env-example .env
```

ตัวอย่าง `.env`

```env
TZ=Asia/Bangkok
IMAGE_NAME=erp:latest
CONTAINER_NAME=dansai
APP_PORT=11447
PAM_PORT=8000

MYSQL_ROOT_PASSWORD=docker
MYSQL_PASSWORD=myPassword
MYSQL_ROOT_HOST=%
MYSQL_DATABASE=db_erp
```

> ⚠️ กรุณาแก้ไขค่าใน `.env` ให้ตรงกับ Environment ของ Server ก่อนเริ่มใช้งาน

---

### 2️⃣ เริ่มต้น Container

รัน Docker Compose เพื่อสร้างและเปิดใช้งานทุกบริการ

```bash
docker compose up -d
```

หรือถ้า Server ใช้ Docker Compose รุ่นเก่า:

```bash
docker-compose up -d
```

---

### 3️⃣ ตรวจสอบ Container ที่กำลังทำงาน

```bash
docker ps
```

ควรพบ Container ของ ERP เช่น:

```text
erp
```

---

### 4️⃣ Migrate Database

หลังจากเริ่ม Container แล้ว ให้รัน Migration:

```bash
docker exec -it erp yii migrate
```

หากต้องการรันคำสั่งเพิ่มเติม:

```bash
docker exec -it erp yii update-table
```

```bash
docker exec -it erp yii update-table/create-view
```

---

# 🔄 การ Update ระบบ

เมื่อมี Version ใหม่ของ ERP บน Docker Hub สามารถ Update ได้โดยไม่ต้องทำหลายขั้นตอน

## ⚡ Update แบบคำสั่งเดียว

Copy คำสั่งด้านล่างไปวางใน Terminal ได้เลย:

```bash
sudo docker pull patjawat/erp:latest && sudo docker compose up -d && sudo docker exec -it erp yii migrate
```

คำสั่งนี้จะทำงานตามลำดับ:

```text
1. Pull Docker Image ล่าสุด
        ↓
2. Update / Restart Container
        ↓
3. Migrate Database
```

### 🛠️ Update แบบแยกขั้นตอน

หากต้องการตรวจสอบแต่ละขั้นตอน:

#### 1. Pull Image ล่าสุด

```bash
sudo docker pull patjawat/erp:latest
```

#### 2. Update Container

```bash
sudo docker compose up -d
```

#### 3. ตรวจสอบ Container

```bash
sudo docker ps
```

#### 4. Migrate Database

```bash
sudo docker exec -it erp yii migrate
```

#### 5. Update Table

```bash
sudo docker exec -it erp yii update-table
```

#### 6. Create / Update View

```bash
sudo docker exec -it erp yii update-table/create-view
```

---

# 🔍 ตรวจสอบระบบหลัง Update

ตรวจสอบ Container:

```bash
sudo docker ps
```

ดู Log ของ ERP:

```bash
sudo docker logs -f erp
```

ดู Log ล่าสุด 100 บรรทัด:

```bash
sudo docker logs --tail 100 erp
```

กด `Ctrl + C` เพื่อออกจาก Log

---

# 🔄 Restart ระบบ

หากต้องการ Restart Container:

```bash
sudo docker compose restart
```

หรือ Restart เฉพาะ ERP:

```bash
sudo docker restart erp
```

---

# 🧹 กรณี Container ไม่ใช้ Image ล่าสุด

หาก Pull Image ใหม่แล้วต้องการบังคับสร้าง Container ใหม่:

```bash
sudo docker pull patjawat/erp:latest && sudo docker compose up -d --force-recreate && sudo docker exec -it erp yii migrate
```

---

# 🛑 หยุดระบบ

หยุด Container ทั้งหมด:

```bash
sudo docker compose stop
```

---

# ▶️ เปิดระบบ

```bash
sudo docker compose start
```

---

# 🗑️ ลบ Container

หากต้องการหยุดและลบ Container:

```bash
sudo docker compose down
```

> ⚠️ `docker compose down` จะลบ Container และ Network แต่โดยปกติไม่ได้ลบ Volume ที่ไม่ได้ระบุ `-v`

**ไม่แนะนำให้ใช้:**

```bash
docker compose down -v
```

กับ Production โดยไม่ตรวจสอบก่อน เพราะอาจลบ Volume ที่เก็บข้อมูล Database

---

# 💾 Backup Database

ก่อน Update ระบบ **แนะนำให้ Backup Database ทุกครั้ง**

ตัวอย่าง:

```bash
sudo docker exec erp mysqldump \
  -u root \
  -p db_erp > backup_$(date +%Y%m%d_%H%M%S).sql
```

ระบบจะถาม Password ของ MySQL

ไฟล์ Backup จะถูกสร้างใน Directory ปัจจุบัน เช่น:

```text
backup_20260820_102030.sql
```

---

# 📁 โครงสร้าง Directory

```text
project-root/
│
├── app-erp/
│   ├── .env              # Environment ของ ERP
│   ├── fileupload/       # เก็บไฟล์อัปโหลด
│   └── doc_receive/      # เอกสารรับเข้า
│
├── mysql_data/
│   └── mysql8.0/data/    # ข้อมูลฐานข้อมูล MySQL
│
├── docker-compose.yml
└── .env
```

---

# 📋 สรุปคำสั่งที่ใช้บ่อย

| การทำงาน          | คำสั่ง                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| ติดตั้งระบบ       | `docker compose up -d`                                                                                         |
| Update ระบบ       | `sudo docker pull patjawat/erp:latest && sudo docker compose up -d && sudo docker exec -it erp yii migrate` |
| Migrate           | `sudo docker exec -it erp yii migrate`                                                                      |
| Update Table      | `sudo docker exec -it erp yii update-table`                                                                 |
| Create View       | `sudo docker exec -it erp yii update-table/create-view`                                                     |
| ตรวจสอบ Container | `sudo docker ps`                                                                                               |
| ดู Log            | `sudo docker logs -f erp`                                                                                   |
| Restart           | `sudo docker compose restart`                                                                                  |
| Stop              | `sudo docker compose stop`                                                                                     |
| Start             | `sudo docker compose start`                                                                                    |
| Down              | `sudo docker compose down`                                                                                     |

---

# ⚠️ ข้อควรระวังสำหรับ Production

ก่อนดำเนินการ Update ระบบ:

1. ตรวจสอบว่า Server มีพื้นที่ Disk เพียงพอ
2. Backup Database
3. ตรวจสอบ Container ที่กำลังทำงาน
4. Pull Image ใหม่
5. Restart / Recreate Container
6. Run Database Migration
7. ตรวจสอบ Log
8. ทดสอบเข้าใช้งานระบบ

## 🚀 คำสั่ง Update ที่แนะนำ

สำหรับการ Update ตามปกติ:

```bash
sudo docker pull patjawat/erp:latest && sudo docker compose up -d && sudo docker exec -it erp yii migrate
```

หากต้องการ **บังคับสร้าง Container ใหม่**:

```bash
sudo docker pull patjawat/erp:latest && sudo docker compose up -d --force-recreate && sudo docker exec -it dansai yii migrate
```
