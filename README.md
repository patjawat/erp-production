# 🏥 Hospital ERP - Docker Deployment

ระบบ ERP โรงพยาบาล (สำหรับใช้งานภายใน)  
ติดตั้งและใช้งานผ่าน **Docker Compose** ได้ง่ายในไม่กี่ขั้นตอน

Promox v.8 ขึ้นไป
---

## ⚙️ ขั้นตอนการติดตั้ง

### 1️⃣ เปลี่ยนชื่อไฟล์ `.env-example` → `.env`
คัดลอกไฟล์ตัวอย่างแล้วเปลี่ยนชื่อเป็น `.env`
```bash
cp .env-example .env

TZ=Asia/Bangkok
IMAGE_NAME=erp:latest
CONTAINER_NAME=dansai
APP_PORT=11447
PAM_PORT=8000
MYSQL_ROOT_PASSWORD=docker
MYSQL_PASSWORD=myPassword
MYSQL_ROOT_HOST=%
MYSQL_DATABASE=db_erp

### 2 เริ่มต้น Container รัน Docker Compose เพื่อสร้างและเปิดใช้งานทุกบริการ
docker-compose up -d

### หรือใช้รูปแบบใหม่ของ Docker Compose:
docker compose -f docker-compose.yml up -d

### 3 ตรวจสอบ Container ที่รันอยู่
docker ps

### 4 Migrate Database (สำหรับ Yii2 หรือ Framework อื่น)

docker exec -it erp bash

# จากนั้นรันคำสั่ง migrate
yii migrate
yii update-table
yii update-table/create-view

# โครงสร้าง Directory
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
