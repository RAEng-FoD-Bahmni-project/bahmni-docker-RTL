<div align="center">
  <a href="./README.md">English</a> |
  <b>العربية</b>
</div>

---

<div dir="rtl" align="right">

# 📘 نظام البهمني باللغة العربية – دليل التثبيت واستعادة قاعدة البيانات

يصف هذا الدليل الإجراء خطوة بخطوة لتثبيت **نظام البهمني باللغة العربية** باستخدام **دوكر** على **أوبونتو**، والتحقق من الخدمات، وإجراء **إصلاحات قاعدة البيانات المطلوبة بعد التثبيت** لكل من **OpenMRS وOpenELIS وOdoo**.

---
## 🎥 فيديو شرح التثبيت

لمشاهدة فيديو يشرح خطوات تثبيت Bahmni (إصدار RTL) باستخدام Docker على أوبونتو، يرجى الضغط على الرابط التالي:

[فيديو تثبيت نظام البهمني باللغة العربية](https://drive.google.com/file/d/1raZ1SlHzV_emJDp_kc0ZMTaxsVR2JTbe/view?usp=sharing)

---

💻😊 دليل إعداد المنشأة للمستخدم
للاطّلاع على خطوات تهيئة النظام ونشره في المنشأة، يُرجى اتباع الدليل أدناه.

[دليل المستخدم باللغة العربية](https://drive.google.com/file/d/1SPDYwM1kJZUVJjKYME2QF2nd4d8nJLgm/view?usp=sharing)

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
