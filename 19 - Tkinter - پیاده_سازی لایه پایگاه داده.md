---
title: "Tkinter - جلسه ۱۹ - پیاده‌سازی لایه پایگاه داده"
aliases:
  - Tkinter Session 19
  - Database Layer Implementation
tags:
  - Python
  - Tkinter
  - SQLite
  - Project
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۹ — پیاده‌سازی لایه پایگاه داده

## هدف جلسه

در جلسه ۱۸، پروژه نهایی مدیریت دانشجو را طراحی کردیم و مسئولیت فایل‌های مختلف را مشخص کردیم. در این جلسه اولین بخش واقعی پروژه را پیاده‌سازی می‌کنیم: لایه پایگاه داده.

در پایان جلسه، دانشجو می‌تواند:

- با `sqlite3` به پایگاه داده SQLite متصل شود.
- جدول دانشجویان را با `CREATE TABLE IF NOT EXISTS` بسازد.
- عملیات افزودن، دریافت، جست‌وجو، ویرایش و حذف را در توابع جدا پیاده‌سازی کند.
- از پارامترهای SQL استفاده کند.
- خطاهای رایج SQLite را مدیریت کند.
- توابع پایگاه داده را بدون ساخت رابط Tkinter آزمایش کند.

## ۱. ساختار پروژه

یک پوشه به نام `student_manager` ایجاد کنید:

```text
student_manager/
├── main.py
├── database.py
├── ui.py
├── settings.py
├── settings.json
└── README.md
```

در این جلسه فعلاً فقط روی `database.py` و یک فایل آزمایشی تمرکز می‌کنیم. لازم نیست فایل‌های رابط کاربری را هنوز کامل بسازیم.

## ۲. چرا SQLite؟

SQLite یک پایگاه داده سبک است که اطلاعات را در یک فایل ذخیره می‌کند. برای پروژه آموزشی ما مزیت‌های زیر را دارد:

- نیازی به راه‌اندازی سرور جداگانه ندارد.
- از طریق ماژول استاندارد `sqlite3` در پایتون قابل استفاده است.
- داده‌ها بعد از بسته‌شدن برنامه باقی می‌مانند.
- برای برنامه‌های کوچک و متوسط و تمرین CRUD مناسب است.

در این پروژه نام فایل پایگاه داده را `students.db` انتخاب می‌کنیم.

## ۳. طراحی جدول

جدول `students` شامل این ستون‌ها خواهد بود:

| ستون | نوع | کاربرد |
|---|---|---|
| `id` | `INTEGER` | شناسه یکتا |
| `full_name` | `TEXT` | نام و نام خانوادگی |
| `student_code` | `TEXT` | کد دانشجویی یکتا |
| `phone` | `TEXT` | شماره تماس |
| `course` | `TEXT` | نام دوره |
| `created_at` | `TEXT` | زمان ثبت رکورد |

دستور SQL:

```sql
CREATE TABLE IF NOT EXISTS students (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    full_name TEXT NOT NULL,
    student_code TEXT NOT NULL UNIQUE,
    phone TEXT NOT NULL,
    course TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

`IF NOT EXISTS` باعث می‌شود اجرای دوباره برنامه به‌دلیل وجود جدول، خطا ندهد.

## ۴. ساخت فایل `database.py`

کد زیر تمام عملیات اصلی پایگاه داده را در یک ماژول قرار می‌دهد. کد را در فایل `database.py` ذخیره کنید.

```python
import sqlite3
from pathlib import Path


DATABASE_FILE = Path(__file__).resolve().parent / "students.db"


def get_connection():
    connection = sqlite3.connect(DATABASE_FILE)
    connection.row_factory = sqlite3.Row
    return connection


def initialize_database():
    with get_connection() as connection:
        connection.execute(
            """
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                full_name TEXT NOT NULL,
                student_code TEXT NOT NULL UNIQUE,
                phone TEXT NOT NULL,
                course TEXT NOT NULL,
                created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
            )
            """
        )


def add_student(full_name, student_code, phone, course):
    with get_connection() as connection:
        cursor = connection.execute(
            """
            INSERT INTO students (full_name, student_code, phone, course)
            VALUES (?, ?, ?, ?)
            """,
            (full_name, student_code, phone, course)
        )
        return cursor.lastrowid


def get_all_students():
    with get_connection() as connection:
        rows = connection.execute(
            """
            SELECT id, full_name, student_code, phone, course, created_at
            FROM students
            ORDER BY id DESC
            """
        ).fetchall()

    return [dict(row) for row in rows]


def get_student_by_id(student_id):
    with get_connection() as connection:
        row = connection.execute(
            """
            SELECT id, full_name, student_code, phone, course, created_at
            FROM students
            WHERE id = ?
            """,
            (student_id,)
        ).fetchone()

    return dict(row) if row is not None else None


def search_students(search_term):
    pattern = f"%{search_term}%"

    with get_connection() as connection:
        rows = connection.execute(
            """
            SELECT id, full_name, student_code, phone, course, created_at
            FROM students
            WHERE full_name LIKE ? OR student_code LIKE ?
            ORDER BY full_name COLLATE NOCASE
            """,
            (pattern, pattern)
        ).fetchall()

    return [dict(row) for row in rows]


def update_student(student_id, full_name, student_code, phone, course):
    with get_connection() as connection:
        cursor = connection.execute(
            """
            UPDATE students
            SET full_name = ?, student_code = ?, phone = ?, course = ?
            WHERE id = ?
            """,
            (full_name, student_code, phone, course, student_id)
        )
        return cursor.rowcount > 0


def delete_student(student_id):
    with get_connection() as connection:
        cursor = connection.execute(
            "DELETE FROM students WHERE id = ?",
            (student_id,)
        )
        return cursor.rowcount > 0
```

### نکته مهم درباره اتصال

عبارت `with connection:` تراکنش را مدیریت می‌کند؛ یعنی در صورت موفقیت تغییرات را ثبت و در صورت خطای داخل بلوک آن‌ها را برمی‌گرداند. اما به‌تنهایی اتصال را نمی‌بندد. در کد بالا، تابع `get_connection()` هر بار اتصال تازه‌ای می‌سازد؛ بنابراین بهتر است در نسخه دقیق‌تر، بستن اتصال نیز صریح انجام شود.

برای جلوگیری از بازماندن اتصال، می‌توانیم یک الگوی کمکی بسازیم و از `contextlib.closing` استفاده کنیم. نسخه زیر را به‌عنوان الگوی پیشنهادی در نظر بگیرید:

```python
from contextlib import closing
import sqlite3


def get_connection():
    connection = sqlite3.connect(DATABASE_FILE)
    connection.row_factory = sqlite3.Row
    return connection


def get_all_students():
    with closing(get_connection()) as connection:
        rows = connection.execute(
            """
            SELECT id, full_name, student_code, phone, course, created_at
            FROM students
            ORDER BY id DESC
            """
        ).fetchall()

    return [dict(row) for row in rows]
```

برای عملیات تغییردهنده، باید هم تراکنش و هم بستن اتصال مدیریت شود. یک الگوی ساده و صریح:

```python
from contextlib import closing


def add_student(full_name, student_code, phone, course):
    with closing(get_connection()) as connection:
        try:
            cursor = connection.execute(
                """
                INSERT INTO students (full_name, student_code, phone, course)
                VALUES (?, ?, ?, ?)
                """,
                (full_name, student_code, phone, course)
            )
            connection.commit()
            return cursor.lastrowid
        except sqlite3.Error:
            connection.rollback()
            raise
```

در ادامه درس می‌توانید همین الگو را برای توابع دیگر نیز اعمال کنید. برای پروژه کلاسی، مهم است دانشجو تفاوت مدیریت تراکنش با بستن اتصال را درک کند.

## ۵. آزمایش مستقل پایگاه داده

پیش از ساخت رابط کاربری، یک فایل به نام `test_database.py` بسازید. این فایل را در همان پوشه قرار دهید.

```python
import sqlite3

from database import (
    initialize_database,
    add_student,
    get_all_students,
    get_student_by_id,
    search_students,
    update_student,
    delete_student
)


def run_demo():
    initialize_database()

    try:
        student_id = add_student(
            "Alex Morgan",
            "ST1001",
            "09120000000",
            "Python"
        )
        print("Inserted ID:", student_id)

    except sqlite3.IntegrityError:
        print("The sample student already exists.")

    print("\nAll students:")
    for student in get_all_students():
        print(student)

    student = get_student_by_id(student_id if "student_id" in locals() else -1)
    print("\nStudent by ID:", student)

    print("\nSearch results:")
    for student in search_students("Alex"):
        print(student)

    print("\nUpdate result:")
    updated = update_student(
        student["id"],
        "Alex Morgan",
        "ST1001",
        "09120000000",
        "Advanced Python"
    ) if student else False
    print(updated)

    print("\nDelete result:")
    deleted = delete_student(student["id"]) if student else False
    print(deleted)


if __name__ == "__main__":
    run_demo()
```

این فایل صرفاً برای آزمایش آموزشی است. چون در پایان رکورد نمونه حذف می‌شود، اجرای چندباره آن معمولاً به تکرار دائمی همان رکورد منجر نمی‌شود.

## ۶. اجرای برنامه آزمایشی

از ترمینال، در پوشه پروژه اجرا کنید:

```bash
python test_database.py
```

اگر در سیستم شما فرمان `python` به نسخه مناسب اشاره نمی‌کند، از `python3 test_database.py` استفاده کنید.

بعد از اجرا باید بتوانید نتیجه افزودن، دریافت، جست‌وجو، ویرایش و حذف را در ترمینال ببینید. فایل `students.db` نیز در پوشه پروژه ایجاد می‌شود.

## ۷. مدیریت خطاها

یکی از خطاهای متداول، ثبت کد دانشجویی تکراری است؛ چون `student_code` قید `UNIQUE` دارد.

در رابط کاربری، خطا را می‌توان به پیام قابل فهم تبدیل کرد:

```python
import sqlite3
from tkinter import messagebox


try:
    add_student(full_name, student_code, phone, course)
except sqlite3.IntegrityError:
    messagebox.showwarning(
        "Duplicate Student Code",
        "This student code is already registered."
    )
except sqlite3.Error:
    messagebox.showerror(
        "Database Error",
        "The student could not be saved."
    )
```

این قطعه کد برای زمانی است که تابع `add_student()` در رابط کاربری فراخوانی می‌شود. در یک معماری پیشرفته‌تر، ماژول پایگاه داده می‌تواند خطا را ثبت یا به خطای دامنه‌ای مشخص تبدیل کند و رابط کاربری فقط پیام مناسب را نشان دهد.

## ۸. چرا از `?` استفاده می‌کنیم؟

در کوئری زیر:

```python
connection.execute(
    "SELECT * FROM students WHERE student_code = ?",
    (student_code,)
)
```

علامت `?` جایگاه پارامتر است و مقدار واقعی جداگانه ارسال می‌شود. این روش از اتصال مستقیم متن ورودی به دستور SQL جلوگیری می‌کند و روش استانداردی برای کوئری‌های پارامتری است.

به این نکته توجه کنید که `(student_code,)` یک تاپل تک‌عضوی است؛ ویرگول انتهای آن ضروری است.

## ۹. تمرین‌های کلاسی

### تمرین ۱ — شمارش دانشجویان

تابعی به نام `count_students()` بنویسید که تعداد رکوردهای جدول را برگرداند.

**پاسخ:**

```python
def count_students():
    with get_connection() as connection:
        row = connection.execute(
            "SELECT COUNT(*) AS total FROM students"
        ).fetchone()

    return row["total"]
```

در نسخه نهایی برنامه، اتصال را نیز مطابق الگوی `contextlib.closing` ببندید.

### تمرین ۲ — جست‌وجو بر اساس دوره

تابعی بنویسید که همه دانشجویان یک دوره مشخص را برگرداند.

**پاسخ:**

```python
def get_students_by_course(course):
    with get_connection() as connection:
        rows = connection.execute(
            """
            SELECT id, full_name, student_code, phone, course, created_at
            FROM students
            WHERE course = ?
            ORDER BY full_name COLLATE NOCASE
            """,
            (course,)
        ).fetchall()

    return [dict(row) for row in rows]
```

### تمرین ۳ — حذف امن

چرا تابع `delete_student()` بهتر است مقدار بولی برگرداند؟

**پاسخ:** تا مشخص شود آیا رکوردی واقعاً حذف شده است یا شناسه داده‌شده در پایگاه داده وجود نداشته است. رابط کاربری می‌تواند بر اساس این نتیجه پیام مناسبی نمایش دهد.

### تمرین ۴ — جلوگیری از تکرار

اگر هنگام افزودن، کد دانشجویی تکراری باشد، چه اتفاقی می‌افتد؟

**پاسخ:** SQLite به‌دلیل قید `UNIQUE` خطای `sqlite3.IntegrityError` ایجاد می‌کند. برنامه باید این خطا را مدیریت کند و پیام قابل فهمی به کاربر بدهد.

## ۱۰. خطاهای رایج

- **فراموش‌کردن ویرگول تاپل تک‌عضوی:** `(student_code,)` درست است؛ `(student_code)` فقط یک مقدار داخل پرانتز است.
- **استفاده از اتصال رشته‌ای در SQL:** ورودی کاربر را مستقیم به متن کوئری اضافه نکنید.
- **فرض کردن اینکه شناسه همیشه وجود دارد:** توابع دریافت، ویرایش و حذف باید حالت نبودن رکورد را مدیریت کنند.
- **آزمایش نکردن کد تکراری:** قید `UNIQUE` باید در آزمون بررسی شود.
- **مخلوط کردن SQL با کد رابط کاربری:** دستورات پایگاه داده را در `database.py` نگه دارید.
- **اشتباه گرفتن تراکنش با بستن اتصال:** `with connection:` تراکنش را مدیریت می‌کند، اما اتصال را لزوماً نمی‌بندد.

## ۱۱. چک‌لیست مدرس

- [ ] فایل `database.py` ساخته شد.
- [ ] جدول `students` ایجاد شد.
- [ ] تابع افزودن رکورد آزمایش شد.
- [ ] دریافت همه رکوردها و دریافت با شناسه آزمایش شد.
- [ ] جست‌وجو، ویرایش و حذف آزمایش شدند.
- [ ] کوئری‌ها از پارامتر استفاده کردند.
- [ ] خطای کد دانشجویی تکراری بررسی شد.
- [ ] تفاوت تراکنش و بستن اتصال توضیح داده شد.

## جلسه بعد

در جلسه ۲۰، رابط گرافیکی پروژه را به پایگاه داده وصل می‌کنیم. ابتدا پنجره اصلی، فرم ورود اطلاعات و `Treeview` را می‌سازیم و سپس نمایش و ثبت رکوردها را از طریق رابط گرافیکی پیاده‌سازی می‌کنیم.
