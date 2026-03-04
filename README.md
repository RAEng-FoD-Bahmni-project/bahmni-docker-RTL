<div align="center">
  <a href="https://zdoc.app/en/RAEng-FoD-Bahmni-project/bahmni-docker-RTL">English</a> |
  <a href="#top">العربية</a>
</div>

<a name="top"></a>
<div dir="rtl" align="right">

# 📘 باهمني دوكر RTL – دليل التثبيت واستعادة قاعدة البيانات

يصف هذا الدليل الإجراء خطوة بخطوة لتثبيت **باهمني (إصدار RTL)** باستخدام **دوكر** على **أوبونتو**، والتحقق من الخدمات، وإجراء **إصلاحات قاعدة البيانات المطلوبة بعد التثبيت** لكل من **OpenMRS وOpenELIS وOdoo**.

---

## 📋 ١. المتطلبات الأساسية

تأكد من توفر ما يلي على الخادم.

### ١.١ نظام التشغيل
- Ubuntu LTS

للتحقق:
```bash
cat /etc/os-release
```

### ١.٢ دوكر و Docker Compose

* Docker Engine
* Docker Compose (إضافة + مستقل)

---

## 🐳 ٢. تثبيت دوكر و Docker Compose (أوبونتو)

### ٢.١ تثبيت Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
```

إضافة مفتاح Docker GPG:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

إضافة مستودع دوكر:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

تثبيت مكونات دوكر:

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
```

---

### ٢.٢ تثبيت Docker Compose (المستقل)

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
-o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

التحقق من التثبيت:

```bash
docker version
docker compose version
```

---

## 🔧 ٣. تثبيت Git (إذا لزم الأمر)

```bash
sudo apt update
sudo apt install -y git
git --version
```

---

## 📁 ٤. إعداد Bahmni Docker

### ٤.١ إنشاء المجلد الأساسي

```bash
mkdir -p /App
cd /App
```

### ٤.٢ استنساخ مستودع Bahmni Docker RTL

```bash
git clone https://github.com/RAEng-FoD-Bahmni-project/bahmni-docker-RTL.git
cd /App/bahmni-docker-RTL/
```

تعيين الصلاحيات:

```bash
chmod -R 0755 /App/bahmni-docker-RTL/
chown -R root:root /App/bahmni-docker-RTL/
```

---

## 📦 ٥. استنساخ وتكوين وحدات Odoo

```bash
mkdir -p /opt/bahmni-erp/
cd /opt/bahmni-erp/
```

استنساخ وحدات Odoo RTL:

```bash
git clone https://github.com/RAEng-FoD-Bahmni-project/odoo-modules-rtl.git
mv odoo-modules-rtl bahmni-addons
```

تعيين الصلاحيات:

```bash
chmod -R 0755 /opt/bahmni-erp/bahmni-addons
chown -R root:root /opt/bahmni-erp/bahmni-addons
```

---

## ▶️ ٦. تشغيل خدمات باهمني

```bash
cd /App/bahmni-docker-RTL/
docker compose up -d
```

سيبدأ دوكر بسحب الصور المطلوبة.

### 🔄 استكشاف الأخطاء أثناء التشغيل الأولي

إذا فشلت بعض الحاويات أثناء التشغيل الأول، أعد تشغيلها يدوياً:

```bash
docker compose restart <container-name>
```

مثال:

```bash
docker compose restart odoodb
```

كرر العملية حتى تعمل جميع الخدمات، ثم شغّل مرة أخرى:

```bash
docker compose up -d
```

---

## ⏳ ٦.١ وقت الانتظار الإلزامي (قبل إصلاحات قاعدة البيانات)

⚠️ **مهم**

بعد تشغيل جميع الحاويات، **انتظر ٣٠ دقيقة على الأقل** قبل المتابعة.

هذا مطلوب من أجل:

* تهيئة قاعدة البيانات
* مزامنة الخدمات
* التشغيل السليم لـ OpenMRS وOdoo وOpenELIS وDCM4CHEE

❗ **لا تتخطَّ هذه الخطوة**

---

## 🎨 ٧. إصلاح: مشكلة CSS المفقود في Odoo

**السبب:** مرفقات غير صالحة بعد استعادة قاعدة البيانات.

```bash
docker compose exec -it odoodb sh
psql -U odoo odoo
```

نفّذ:

```sql
DELETE FROM ir_attachment WHERE name LIKE '%web/content%';
```

اخرج:

```bash
\q
exit
```

أعد تشغيل Odoo:

```bash
docker compose restart odoo
```

## 📂 إصلاح: خطأ مستندات المرضى (إذا حدث لاحقاً)

⚠️ طبّق فقط إذا حدثت أخطاء في رفع/عرض مستندات المرضى.

```bash
docker compose exec -it patient-documents sh
```

ثبّت ACL:

```bash
apk add acl
```

```bash
setfacl -dRm o::rwx /usr/share/nginx/html/document_images/
chmod -R 777 /usr/share/nginx/html/document_images/
ls -al /usr/share/nginx/html/document_images/
```

```bash
\q
exit
```

أعد تشغيل patient-documents:

```bash
docker compose restart patient-documents
```

## 🛏 إصلاح: مشكلة إدارة الأسرّة (إلزامي)

⚠️ طبّق فقط إذا حدثت أخطاء في إدارة الأسرّة.

```bash
docker compose exec -it openmrsdb sh
mysql -uroot -padminAdmin!123 openmrs
```

نفّذ:

```sql
ALTER TABLE bed_location_map 
ADD COLUMN row_number INT NULL AFTER location_id;

ALTER TABLE bed_location_map 
ADD COLUMN column_number INT NULL AFTER row_number;

UPDATE bed_location_map 
SET row_number = bed_row_number,
    column_number = bed_column_number;
```

اخرج:

```bash
\q
exit
```

أعد تشغيل OpenMRS:

```bash
docker compose restart openmrs
```

---

## 🔁 ٨. إصلاح: مشكلة مزامنة OpenMRS (جدول Markers)

```bash
docker compose exec -it openmrsdb sh
mysql -uroot -padminAdmin!123 openmrs
```

نفّذ:

```sql
DELETE FROM markers WHERE feed_uri LIKE '%feed/patient/recent%';
```

اخرج:

```bash
exit
```

---

## 🔄 ٩. إعادة تشغيل جميع خدمات باهمني

```bash
docker compose down
docker compose build
docker compose up -d
```

---

## ⏳ ١٠. وقت الانتظار الإلزامي (بعد إعادة التشغيل)

⚠️ **مهم**

بعد إعادة تشغيل جميع الخدمات، انتظر **٣٠ دقيقة** أخرى للتأكد من تهيئة جميع الحاويات بالكامل.

استمر بتحديث واجهة المستخدم خلال هذه الفترة.

---

## 🧪 ١١. خطوات ما بعد الاستعادة

١. الوصول إلى OpenMRS:

   ```
   https://<server-ip>/openmrs
   ```
٢. إعادة بناء فهرس البحث:

   * واجهة OpenMRS → **المسؤول (Admin)**
   * **فهرس البحث (Search Index)**
   * انقر على **إعادة بناء فهرس البحث (Rebuild Search Index)**

---

## ⚠️ ملاحظة مهمة

خطوات تنظيف قاعدة البيانات في **الأقسام ٧ و٨** مطلوبة **مرة واحدة فقط** بعد:

* أول سحب للصور
* استعادة قاعدة البيانات

**لا** تحتاج إلى تكرارها عند إعادة التشغيل اللاحقة.

---

## ✅ الإتمام والتحقق

يجب أن يكون باهمني الآن يعمل بشكل كامل.

### 🔍 تفاصيل الوصول إلى التطبيقات

| التطبيق | الرابط | بيانات الدخول |
| ----------- | ---------------------------------------------------------------------------------- | ------------------- |
| تطبيق باهمني | [https://localhost/](https://localhost/) | — |
| OpenMRS | [https://localhost/openmrs](https://localhost/openmrs) | superman / Admin123 |
| Odoo ERP | [http://localhost:8069](http://localhost:8069) | admin / admin |
| DCM4CHEE | [https://localhost/dcm4chee-web3/](https://localhost/dcm4chee-web3/) | admin / admin |
| OpenELIS | [https://localhost/openelis/LoginPage.do](https://localhost/openelis/LoginPage.do) | admin / adminADMIN! |

---

</div>

---

# 📘 Bahmni Docker RTL – Installation & Database Restoration Guide

This guide describes the step-by-step procedure to install **Bahmni (RTL version)** using **Docker** on **Ubuntu**, verify services, and perform required **post-installation database fixes** for **OpenMRS, OpenELIS, and Odoo**.

---

## 📋 1. Prerequisites

Ensure the following are available on the server.

### 1.1 Operating System
- Ubuntu LTS

Verify:
```bash
cat /etc/os-release
```

### 1.2 Docker & Docker Compose

* Docker Engine
* Docker Compose (plugin + standalone)

---

## 🐳 2. Docker & Docker Compose Installation (Ubuntu)

### 2.1 Install Docker Engine

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
```

Add Docker GPG key:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
-o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker repository:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker components:

```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
docker-buildx-plugin docker-compose-plugin
```

---

### 2.2 Install Docker Compose (Standalone)

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
-o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Verify installation:

```bash
docker version
docker compose version
```

---

## 🔧 3. Git Installation (If Required)

```bash
sudo apt update
sudo apt install -y git
git --version
```

---

## 📁 4. Bahmni Docker Setup

### 4.1 Create Base Directory

```bash
mkdir -p /App
cd /App
```

### 4.2 Clone Bahmni Docker RTL Repository

```bash
git clone https://github.com/RAEng-FoD-Bahmni-project/bahmni-docker-RTL.git
cd /App/bahmni-docker-RTL/
```

Set permissions:

```bash
chmod -R 0755 /App/bahmni-docker-RTL/
chown -R root:root /App/bahmni-docker-RTL/
```

---

## 📦 5. Clone & Configure Odoo Modules

```bash
mkdir -p /opt/bahmni-erp/
cd /opt/bahmni-erp/
```

Clone Odoo RTL modules:

```bash
git clone https://github.com/RAEng-FoD-Bahmni-project/odoo-modules-rtl.git
mv odoo-modules-rtl bahmni-addons
```

Set permissions:

```bash
chmod -R 0755 /opt/bahmni-erp/bahmni-addons
chown -R root:root /opt/bahmni-erp/bahmni-addons
```

---

## ▶️ 6. Start Bahmni Services

```bash
cd /App/bahmni-docker-RTL/
docker compose up -d
```

Docker will start pulling the required images.

### 🔄 Troubleshooting During Initial Startup

If certain containers fail during the first run, restart them manually:

```bash
docker compose restart <container-name>
```

Example:

```bash
docker compose restart odoodb
```

Repeat until all services are running, then run again:

```bash
docker compose up -d
```

---

## ⏳ 6.1 Mandatory Wait Time (Before Database Fixes)

⚠️ **Important**

After all containers are up and running, **wait at least 30 minutes** before proceeding.

This is required for:

* Database initialization
* Inter-service synchronization
* Proper startup of OpenMRS, Odoo, OpenELIS, and DCM4CHEE

❗ **Do NOT skip this step**

---

## 🎨 7. Fix: Odoo CSS Missing Issue

**Cause:** Invalid attachments after database restoration.

```bash
docker compose exec -it odoodb sh
psql -U odoo odoo
```

Run:

```sql
DELETE FROM ir_attachment WHERE name LIKE '%web/content%';
```

Exit:

```bash
\q
exit
```

Restart Odoo:

```bash
docker compose restart odoo
```

##  📂 Fix: Patient Documents Error (If Occurs Later)

⚠️ Apply only if patient document upload/view errors occur.

```bash
docker compose exec -it patient-documents sh

```

Install ACL:

```bash
apk add acl
```

```bash
setfacl -dRm o::rwx /usr/share/nginx/html/document_images/
chmod -R 777 /usr/share/nginx/html/document_images/
ls -al /usr/share/nginx/html/document_images/
```



```bash
\q
exit
```

Restart patient-documents:

```bash
docker compose restart patient-documents
```

## 🛏 Fix: Bed Management Issue (Mandatory)

⚠️ Apply only if Bed Management errors occur.

```bash
docker compose exec -it openmrsdb sh
mysql -uroot -padminAdmin!123 openmrs
```

Run:

```sql
ALTER TABLE bed_location_map 
ADD COLUMN row_number INT NULL AFTER location_id;

ALTER TABLE bed_location_map 
ADD COLUMN column_number INT NULL AFTER row_number;

UPDATE bed_location_map 
SET row_number = bed_row_number,
    column_number = bed_column_number;
```

Exit:

```bash
\q
exit
```

Restart Openmrs:

```bash
docker compose restart openmrs
```

---

## 🔁 8. Fix: OpenMRS Sync Issue (Markers Table)

```bash
docker compose exec -it openmrsdb sh
mysql -uroot -padminAdmin!123 openmrs
```

Run:

```sql
DELETE FROM markers WHERE feed_uri LIKE '%feed/patient/recent%';
```

Exit:

```bash
exit
```

---

## 🔄 9. Restart All Bahmni Services

```bash
docker compose down
docker compose build
docker compose up -d
```

---

## ⏳ 10. Mandatory Wait Time (After Restart)

⚠️ **Important**

After restarting all services, wait another **30 minutes** to ensure all containers are fully initialized.

Keep refreshing the UI during this time.

---

## 🧪 11. Post-Restoration Steps

1. Access OpenMRS:

   ```
   https://<server-ip>/openmrs
   ```
2. Rebuild Search Index:

   * OpenMRS UI → **Admin**
   * **Search Index**
   * Click **Rebuild Search Index**

---

## ⚠️ Important Note

The database cleanup steps in **Sections 7 and 8** are required **only once** after:

* First image pull
* Database restoration

They do **NOT** need to be repeated on subsequent restarts.

---

## ✅ Completion & Verification

Bahmni should now be fully functional.

### 🔍 Application Access Details

| Application | URL                                                                                | Credentials         |
| ----------- | ---------------------------------------------------------------------------------- | ------------------- |
| Bahmni App  | [https://localhost/](https://localhost/)                                           | —                   |
| OpenMRS     | [https://localhost/openmrs](https://localhost/openmrs)                             | superman / Admin123 |
| Odoo ERP    | [http://localhost:8069](http://localhost:8069)                                     | admin / admin       |
| DCM4CHEE    | [https://localhost/dcm4chee-web3/](https://localhost/dcm4chee-web3/)               | admin / admin       |
| OpenELIS    | [https://localhost/openelis/LoginPage.do](https://localhost/openelis/LoginPage.do) | admin / adminADMIN! |

---
