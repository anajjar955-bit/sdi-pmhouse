# SDI® PMHouse — منصة تقييم الشخصية

تطبيق فول-ستاك لاستبيان SDI® (Strength Deployment Inventory) بالعربية، يتضمن:

- **Frontend**: HTML/CSS/JS نقية (RTL عربي) — موجود في `public/`
- **Backend**: Node.js + Express REST API
- **Database**: PostgreSQL
- **PDF Reports**: تقارير احترافية كاملة عبر Puppeteer (للمستخدم والأدمن)
- **Auth**: JWT + bcrypt + سؤال أمان لاستعادة كلمة المرور

---

## 📁 هيكلة المشروع

```
sdi-app/
├── package.json
├── nixpacks.toml              # إعداد نشر Railway (Chromium)
├── .env.example               # قالب متغيرات البيئة
├── server/
│   ├── index.js               # نقطة الدخول لـ Express
│   ├── db/
│   │   ├── pool.js            # اتصال Postgres
│   │   ├── schema.sql         # جداول DB
│   │   └── migrate.js         # سكريبت الترحيل
│   ├── middleware/auth.js     # JWT middleware
│   ├── routes/
│   │   ├── auth.js            # تسجيل/دخول/استعادة
│   │   ├── results.js         # حفظ واسترجاع النتائج
│   │   ├── admin.js           # لوحة الأدمن
│   │   └── pdf.js             # تحميل PDF
│   └── utils/
│       ├── sdi-data.js        # بيانات الأسئلة والتحليل
│       └── pdf-generator.js   # مولّد الـ PDF
└── public/
    └── index.html             # الواجهة (frontend)
```

---

## 🚀 التشغيل محلياً

### المتطلبات
- Node.js 18 أو أحدث
- PostgreSQL 14+ مُشغَّل محلياً

### الخطوات

```bash
# 1) تثبيت المكتبات
npm install

# 2) نسخ ملف البيئة وتحريره
cp .env.example .env
# افتح .env وضع DATABASE_URL الخاص بك + JWT_SECRET قوي

# 3) تشغيل الترحيل (ينشئ الجداول + حساب الأدمن الأولي)
npm run migrate

# 4) تشغيل الخادم
npm start
```

افتح `http://localhost:3000` في المتصفح.

**للتطوير مع إعادة التحميل التلقائي**:
```bash
npm run dev
```

---

## ☁ النشر على Railway

### الخطوات

**1) أنشئ مشروعاً جديداً على Railway** من هذا المستودع (GitHub).

**2) أضف PostgreSQL** من لوحة Railway (زر "New" → "Database" → "PostgreSQL"). سيوفّر Railway متغير `DATABASE_URL` تلقائياً.

**3) أضف متغيرات البيئة** في إعدادات الخدمة على Railway:

| المتغير | القيمة |
|---|---|
| `JWT_SECRET` | سلسلة عشوائية طويلة (32+ حرف). مثال: `openssl rand -base64 48` |
| `NODE_ENV` | `production` |
| `ADMIN_EMAIL` | `anajjar@pmhouse.org` |
| `ADMIN_NAME` | `Akram Elnagar` |
| `ADMIN_PASSWORD` | كلمة مرور قوية للأدمن |
| `CORS_ORIGIN` | `*` أو رابط الموقع المحدد |

**4) Deploy** — Railway سيرى `nixpacks.toml` ويحمّل Chromium تلقائياً.

**5) شغّل الترحيل**:
  - الأسهل: في `nixpacks.toml` أمر البدء يتضمن `npm run migrate` تلقائياً عند أول نشر.
  - بديلاً: افتح Terminal في Railway وشغّل `npm run migrate` يدوياً.

**6) سجّل الدخول كأدمن** بـ `ADMIN_EMAIL` و`ADMIN_PASSWORD` الذي حددته.

### ملاحظة حول Chromium على Railway

ملف `nixpacks.toml` يحمّل Chromium كحزمة نظام ويستخدمه Puppeteer مباشرة (أسرع وأخف من تحميل Chromium الخاص بـ Puppeteer). المتغير `PUPPETEER_EXECUTABLE_PATH` موجّه لـ `/nix/var/nix/profiles/default/bin/chromium`.

إذا لم يعمل الـ PDF generation على Railway، جرّب:
- إضافة المتغير `PUPPETEER_SKIP_DOWNLOAD=false` (يُحمّل Chromium بدل الاعتماد على النظام).
- أو استخدم Dockerfile مخصص (أخبرني إذا احتجت نسخة Docker).

---

## 🔌 مرجع الـ API

كل المسارات تحت `/api`. المسارات المحمية تتطلب هيدر `Authorization: Bearer <JWT>`.

### Auth

| Method | Path | وصف |
|---|---|---|
| POST | `/api/auth/register` | إنشاء حساب + إرجاع JWT |
| POST | `/api/auth/login` | تسجيل دخول + JWT |
| POST | `/api/auth/forgot/check-email` | إرجاع سؤال الأمان |
| POST | `/api/auth/forgot/reset` | إعادة تعيين كلمة المرور |
| GET | `/api/auth/me` | بيانات المستخدم الحالي |

### Results

| Method | Path | Auth | وصف |
|---|---|---|---|
| POST | `/api/results/submit` | مستخدم | إرسال الإجابات وحفظ النتيجة |
| GET | `/api/results/latest` | مستخدم | آخر نتيجة للمستخدم |
| GET | `/api/results/mine` | مستخدم | كل نتائج المستخدم |
| GET | `/api/results/:id` | مالك/أدمن | نتيجة محددة |

### Admin

| Method | Path | وصف |
|---|---|---|
| GET | `/api/admin/stats` | إحصاءات عامة |
| GET | `/api/admin/users` | قائمة المستخدمين مع آخر نتيجة |
| DELETE | `/api/admin/users/:id` | حذف مستخدم (cascade) |
| GET | `/api/admin/export/csv` | تصدير CSV |

### PDF

| Method | Path | وصف |
|---|---|---|
| GET | `/api/pdf/my/latest` | PDF لآخر نتيجة للمستخدم |
| GET | `/api/pdf/my/:resultId` | PDF لنتيجة محددة |
| GET | `/api/pdf/admin/user/:userId` | PDF لأي مستخدم (أدمن) |
| GET | `/api/pdf/admin/result/:resultId` | PDF لنتيجة محددة (أدمن) |

### Health

- `GET /health` — فحص اتصال DB

---

## 🔐 الأمان

- كل كلمات المرور وإجابات سؤال الأمان مُشفّرة بـ **bcrypt** (cost 10)
- JWT يبقى 7 أيام
- Rate limiting على نقاط الـ auth (20 طلب / 15 دقيقة)
- Helmet لأمان HTTP headers
- Body parser محدد بـ 200KB
- SQL injection مستحيل (استعلامات parameterized فقط)

---

## 📊 تقارير PDF

كل تقرير PDF يحتوي على:
1. بيانات المستخدم (الاسم، البريد، الجوال، الدولة)
2. منظومة بواعث القيم (MVS) — النقطة في المثلث
3. **مثلث SDI® كامل** مرسوم بـ SVG مع:
   - شبكة بـ 10 مستويات
   - مناطق ملوّنة لكل نمط
   - النقطة (MVS) ورأس السهم (تسلسل الصراع)
4. تسلسل الصراع (3 مراحل متصاعدة)
5. التحليل الشامل للشخصية (نقاط القوة + الحذر)
6. جدول التجربة الداخلية في الصراع
7. جدول السلوك الظاهر في الصراع

الخط العربي في الـ PDF: **Tajawal** (من Google Fonts).

تقارير الأدمن تحمل علامة مميزة ("تقرير أدمن") مع اسم الأدمن الذي أنشأ التقرير.

---

## 🛠 التطوير والصيانة

### تغيير كلمة مرور الأدمن

1. سجّل دخول كأدمن.
2. حالياً لا يوجد UI لتغيير الكلمة. الطريقة اليدوية:
   ```sql
   -- في psql
   UPDATE users SET password_hash = '<bcrypt-hash>' WHERE email = 'anajjar@pmhouse.org';
   ```
   لتوليد bcrypt hash:
   ```bash
   node -e "const b=require('bcryptjs'); console.log(b.hashSync('كلمة المرور الجديدة', 10));"
   ```

### نسخ احتياطي لقاعدة البيانات

```bash
# تصدير
pg_dump $DATABASE_URL > backup-$(date +%Y%m%d).sql

# استرجاع
psql $DATABASE_URL < backup-20260101.sql
```

### إعادة بناء الجداول

```bash
# حذر: سيحذف كل البيانات
psql $DATABASE_URL -c "DROP TABLE IF EXISTS results, sessions, users CASCADE; DROP VIEW IF EXISTS v_latest_results CASCADE;"
npm run migrate
```

---

## 📄 الترخيص والحقوق

- الكود: استخدام داخلي لـ PMHouse.
- SDI® علامة تجارية مسجلة لـ **Personal Strengths Publishing** (Dr. Elias H. Porter).
- هذه الأداة تعليمية ولا تحل محل التقييم الرسمي المعتمد.
