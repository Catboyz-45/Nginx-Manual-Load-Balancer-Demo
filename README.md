# ⚖️ Nginx Manual Load Balancer Demo (JSON Server + Docker)

โปรเจกต์สาธิตการทำ **Reverse Proxy / Load Balancer** ด้วย **Nginx** โดยรัน backend เป็น container แยกอิสระหลายตัวด้วยคำสั่ง `docker run` (ไม่ใช้ Docker Compose) แล้วกำหนดรายชื่อ backend ทั้งหมดไว้ตรงๆ ใน `upstream` ของ Nginx เอง เพื่อทดสอบว่า Nginx กระจาย request ไปยัง backend แต่ละตัวแบบ round-robin จริงหรือไม่ โดยดูจาก `server_name` ที่แต่ละ backend ตอบกลับมา

---

## 🎯 แนวคิดของโปรเจกต์

* รัน **JSON Server** (image `imsuriyapi/json-server`) เป็น container 4 ตัวแยกกัน (`my-api-1` ถึง `my-api-4`) แต่ละตัว mount ไฟล์ข้อมูลคนละไฟล์ (`db1.json`–`db4.json`) ที่มีค่า `server_name` ต่างกัน ("Server 1", "Server 2", ...) เพื่อให้แยกได้ว่า request ล่าสุดถูกตอบโดย container ไหน
* แต่ละ container เปิด host port ต่างกัน (`3001`–`3004`) แม็พเข้ากับพอร์ต `3000` ภายใน container
* รัน **Nginx** อีก container หนึ่ง โดยแทนที่ config หลักด้วย `conf/nginx.conf` ที่กำหนด `upstream apis` ให้ชี้ไปที่ `host.docker.internal:3001` ถึง `3004` ตรงๆ แล้ว `proxy_pass` ทุก request มาที่ upstream กลุ่มนี้ → Nginx จะกระจาย request แบบ round-robin ไปยัง backend ทั้ง 4 ตัวให้เอง

---

## 🛠️ เทคโนโลยีที่เลือกใช้ (Tech Stack)

* **[Docker](https://www.docker.com/):** รัน container แต่ละตัวแยกกันด้วย `docker run` (ไม่ใช้ Compose ในเวอร์ชันนี้)
* **[Nginx](https://nginx.org/):** ทำหน้าที่ reverse proxy / load balancer พร้อม `upstream` block แบบกำหนด backend ตรงๆ
* **[JSON Server](https://github.com/typicode/json-server)** (image `imsuriyapi/json-server`): จำลอง REST API ง่ายๆ จากไฟล์ JSON โดยไม่ต้องเขียนโค้ด backend เอง

---

## 📁 โครงสร้างโปรเจกต์ (Project Structure)

```
docker-nginx-reversproxy-loadbalancer/
├── conf/
│   └── nginx.conf     # ตั้งค่า upstream "apis" ชี้ไปที่ 4 backend + proxy_pass
├── db/
│   ├── db1.json        # ข้อมูลจำลอง, server_name = "Server 1"
│   ├── db2.json        # ข้อมูลจำลอง, server_name = "Server 2"
│   ├── db3.json        # ข้อมูลจำลอง, server_name = "Server 3"
│   └── db4.json        # ข้อมูลจำลอง, server_name = "Server 4"
└── command.txt          # รวมคำสั่ง docker run ทั้งหมดที่ใช้รันเดโม
```

> หมายเหตุ: `command.txt` ใช้ path แบบ Windows (`C:\nginx101\...`) ในการ mount volume ซึ่งผูกกับเครื่องที่ใช้ทำตอนแรก หากรันบนเครื่อง/OS อื่น ต้องแก้ path ให้ตรงกับตำแหน่งไฟล์จริงในเครื่องนั้นๆ ก่อน

---

## 🚀 วิธีใช้งาน (Getting Started)

```bash
# รัน backend 4 ตัว แต่ละตัว mount คนละไฟล์ db
docker run -d --name my-api-1 -p 3001:3000 -v <path>/db/db1.json:/usr/app/db.json imsuriyapi/json-server
docker run -d --name my-api-2 -p 3002:3000 -v <path>/db/db2.json:/usr/app/db.json imsuriyapi/json-server
docker run -d --name my-api-3 -p 3003:3000 -v <path>/db/db3.json:/usr/app/db.json imsuriyapi/json-server
docker run -d --name my-api-4 -p 3004:3000 -v <path>/db/db4.json:/usr/app/db.json imsuriyapi/json-server

# รัน Nginx โดยแทนที่ config หลักด้วย conf/nginx.conf
docker run -d --name my-nginx -p 8090:80 -v <path>/conf/nginx.conf:/etc/nginx/nginx.conf nginx

# ทดสอบเรียกซ้ำหลายครั้งผ่าน Nginx
curl http://localhost:8090/servers
```

เรียก `/servers` ซ้ำๆ จะเห็นค่า `server_name` เปลี่ยนวนไปตาม backend 4 ตัวที่ Nginx กระจาย request ไปให้ ยืนยันว่า load balancing ทำงานถูกต้อง

---

## 👥 ผู้พัฒนา (Developer)

* **Nuttaphat** ([@Catboyz-45](https://github.com/Catboyz-45))
