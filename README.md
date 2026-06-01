[README.md](https://github.com/user-attachments/files/28484035/README.md)
# نظام إدارة المساجد — الخلفية (Backend)
### جمعية العناية بالمساجد بمحافظة شرورة

واجهة برمجية (REST API) مبنية بـ **Python + FastAPI** مع مصادقة **JWT** ونظام صلاحيات ثلاثي.

---

## التشغيل

```bash
# 1) تثبيت المتطلبات
pip install -r requirements.txt

# 2) تشغيل الخادم
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

عند أول تشغيل يُنشأ حساب المدير تلقائياً:
- **اسم المستخدم:** `admin`
- **كلمة المرور:** `admin123`

توثيق تفاعلي للـ API على: `http://localhost:8000/docs`

> ⚠️ في الإنتاج: غيّر `SECRET_KEY` و`FIRST_ADMIN_PASSWORD` عبر متغيرات البيئة أو ملف `.env`، واضبط نطاقات CORS في `app/main.py`.

---

## الأدوار والصلاحيات

| الإجراء | مدير (admin) | موظف (staff) | مشرف مسجد (supervisor) |
|---|:---:|:---:|:---:|
| إدارة المستخدمين (إنشاء/تعديل/حذف) | ✅ | ❌ | ❌ |
| إضافة/تعديل المساجد | ✅ | ✅ | تعديل مسجده فقط |
| حذف المساجد | ✅ | ❌ | ❌ |
| الطلبات (إضافة) | ✅ | ✅ | مسجده فقط |
| حذف الطلبات | ✅ | ✅ | ❌ |
| المشاريع (إضافة/تعديل) | ✅ | ✅ | عرض مسجده فقط |
| حذف المشاريع | ✅ | ❌ | ❌ |
| المهام (إضافة/تعديل/حذف) | ✅ | ✅ | مسجده فقط |
| عرض البيانات | كل المساجد | كل المساجد | **مسجده فقط** |

**مشرف المسجد** يُربط بمسجد محدد عند إنشائه، وكل ما يضيفه يُربط تلقائياً بمسجده، ولا يرى أو يعدّل بيانات مساجد أخرى.

---

## أهم نقاط النهاية (Endpoints)

```
POST   /api/auth/login          تسجيل الدخول (يرجع JWT)
GET    /api/auth/me             بيانات المستخدم الحالي

GET    /api/users               قائمة المستخدمين        (مدير)
POST   /api/users               إنشاء مستخدم            (مدير)
PUT    /api/users/{id}          تعديل مستخدم            (مدير)
DELETE /api/users/{id}          حذف مستخدم              (مدير)

GET    /api/masajid             قائمة المساجد
POST   /api/masajid             إضافة مسجد              (مدير/موظف)
PUT    /api/masajid/{id}        تعديل مسجد
DELETE /api/masajid/{id}        حذف مسجد                (مدير)

GET|POST|PUT|DELETE /api/requests        الطلبات
GET|POST|PUT|DELETE /api/projects        المشاريع
GET|POST|PUT|DELETE /api/tasks           المهام

GET    /api/stats               إحصائيات لوحة التحكم
```

جميع الطلبات (عدا تسجيل الدخول) تتطلب ترويسة:
```
Authorization: Bearer <token>
```

---

## بنية المشروع

```
backend/
├── requirements.txt
└── app/
    ├── main.py          # نقطة الدخول + تهيئة القاعدة + المدير الأول
    ├── config.py        # الإعدادات
    ├── database.py      # اتصال SQLAlchemy
    ├── models.py        # جداول قاعدة البيانات
    ├── schemas.py       # مخططات Pydantic
    ├── auth.py          # JWT + تشفير + اعتماديات الصلاحيات
    └── routers/
        ├── users.py         # المصادقة + إدارة المستخدمين
        ├── masajid.py
        ├── requests.py
        ├── projects.py
        ├── tasks.py
        └── stats.py
```

قاعدة البيانات الافتراضية **SQLite** (`masajid.db`). للتحويل إلى PostgreSQL غيّر `DATABASE_URL`.
