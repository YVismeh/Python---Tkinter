---
title: "Tkinter - جلسه ۱۶ - مدیریت خطا و اعتبارسنجی حرفه‌ای"
aliases:
  - Tkinter Session 16
  - Error Handling and Validation
tags:
  - Python
  - Tkinter
  - SQLite
  - Validation
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۶ — مدیریت خطا و اعتبارسنجی حرفه‌ای

## هدف جلسه

در جلسات گذشته، برنامه مدیریت دانشجو را با Tkinter و SQLite ساختیم و کد را در چند فایل قرار دادیم. اکنون باید برنامه را در برابر ورودی نامعتبر، خطاهای پایگاه داده و شرایط پیش‌بینی‌نشده مقاوم‌تر کنیم.

در پایان جلسه دانشجو باید بتواند:

- تفاوت اعتبارسنجی ورودی و مدیریت خطا را توضیح دهد.
- ورودی‌های خالی و مقادیر نامعتبر را بررسی کند.
- خطاهای SQLite را با `try/except` مدیریت کند.
- پیام مناسبی به کاربر نشان دهد و جزئیات فنی را از کاربر عادی پنهان نگه دارد.
- عملیات پایگاه داده را در توابع مشخص متمرکز کند.
- از تکرار کد اعتبارسنجی جلوگیری کند.

## ۱. اعتبارسنجی و مدیریت خطا چه تفاوتی دارند؟

**اعتبارسنجی (Validation)** بررسی می‌کند که آیا ورودی از نظر قوانین برنامه قابل قبول است یا نه. برای مثال، نام نباید خالی باشد و سن باید عددی در محدوده مشخص باشد.

**مدیریت خطا (Error Handling)** برای واکنش کنترل‌شده به مشکلاتی است که هنگام اجرای برنامه رخ می‌دهند؛ برای مثال، فایل پایگاه داده قابل دسترسی نباشد یا یک عملیات SQLite با خطا مواجه شود.

نمونه اعتبارسنجی:

```python
age_text = age_entry.get().strip()

try:
    age = int(age_text)
except ValueError:
    messagebox.showwarning(
        "Invalid Age",
        "Enter a whole number for age."
    )
    return
```

نمونه مدیریت خطای پایگاه داده:

```python
try:
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute(
            "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
            (name, age, city)
        )
except sqlite3.Error:
    messagebox.showerror(
        "Database Error",
        "The record could not be saved."
    )
```

در برنامه واقعی، می‌توان خطا را برای عیب‌یابی ثبت کرد؛ اما نباید اطلاعات حساس یا جزئیات غیرضروری را در پیام کاربر نمایش داد.

## ۲. ساخت تابع اعتبارسنجی متمرکز

اگر اعتبارسنجی در چند دکمه تکرار شود، اصلاح قوانین دشوار می‌شود. بهتر است یک تابع مشخص برای خواندن و بررسی ورودی‌های فرم داشته باشیم.

```python
def get_form_data(self):
    name = self.name_entry.get().strip()
    age_text = self.age_entry.get().strip()
    city = self.city_entry.get().strip()

    if not name:
        messagebox.showwarning(
            "Invalid Name",
            "Name cannot be empty."
        )
        self.name_entry.focus_set()
        return None

    if not city:
        messagebox.showwarning(
            "Invalid City",
            "City cannot be empty."
        )
        self.city_entry.focus_set()
        return None

    try:
        age = int(age_text)
    except ValueError:
        messagebox.showwarning(
            "Invalid Age",
            "Age must be a whole number."
        )
        self.age_entry.focus_set()
        return None

    if age < 1 or age > 120:
        messagebox.showwarning(
            "Invalid Age",
            "Age must be between 1 and 120."
        )
        self.age_entry.focus_set()
        return None

    return name, age, city
```

در عملیات افزودن و ویرایش، این تابع را صدا می‌زنیم:

```python
data = self.get_form_data()
if data is None:
    return

name, age, city = data
```

اگر ورودی معتبر نباشد، تابع `None` برمی‌گرداند و عملیات متوقف می‌شود.

## ۳. مدیریت خطاهای SQLite

خطاهای SQLite از نوع `sqlite3.Error` هستند. برای مثال، فایل پایگاه داده ممکن است در دسترس نباشد یا محدودیت یک ستون نقض شود.

```python
try:
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute(
            "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
            (name, age, city)
        )
except sqlite3.IntegrityError:
    messagebox.showwarning(
        "Data Error",
        "The data violates a database constraint."
    )
except sqlite3.Error:
    messagebox.showerror(
        "Database Error",
        "The database operation failed."
    )
```

`sqlite3.IntegrityError` زیرمجموعه‌ای از خطاهای SQLite است و می‌تواند برای نقض محدودیت‌هایی مانند `NOT NULL`, `UNIQUE` یا کلید خارجی رخ دهد.

ترتیب `except`ها اهمیت دارد: خطای خاص‌تر را پیش از خطای عمومی‌تر بررسی می‌کنیم.

## ۴. ثبت جزئیات خطا برای عیب‌یابی

در برنامه‌های بزرگ‌تر بهتر است جزئیات خطا در گزارش برنامه ثبت شوند. ماژول استاندارد `logging` برای این کار مناسب است.

```python
import logging

logging.basicConfig(
    filename="app.log",
    level=logging.ERROR,
    format="%(asctime)s %(levelname)s %(message)s"
)

try:
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute("SELECT * FROM missing_table")
except sqlite3.Error:
    logging.exception("Database operation failed")
    messagebox.showerror(
        "Database Error",
        "An unexpected database error occurred."
    )
```

`logging.exception()` را داخل بلوک `except` فراخوانی کنید تا اطلاعات استثنا نیز ثبت شود. فایل گزارش ممکن است اطلاعات فنی برنامه را در خود داشته باشد؛ آن را در اختیار افراد غیرمجاز قرار ندهید.

## ۵. تراکنش و اطمینان از ثبت تغییرات

وقتی از این الگو استفاده می‌کنیم:

```python
with sqlite3.connect(DATABASE_NAME) as connection:
    connection.execute(
        "UPDATE students SET age = ? WHERE id = ?",
        (21, student_id)
    )
```

بلوک `with` تراکنش را مدیریت می‌کند: اگر عملیات موفق باشد تغییرات ثبت می‌شوند؛ اگر خطایی رخ دهد، تغییرات همان تراکنش بازگشت داده می‌شوند. توجه کنید که این الگو به‌تنهایی اتصال را فوراً نمی‌بندد؛ در کدهای معمول می‌توان اتصال را با `contextlib.closing` یا مدیریت صریح اتصال نیز بست. در این جزوه، توابع کوتاه و مستقل از اتصال استفاده می‌کنند و در پایان بلوک اتصال در چرخه عادی آزاد می‌شود؛ برای کنترل دقیق منابع، نسخه زیر روشن‌تر است:

```python
from contextlib import closing
import sqlite3

with closing(sqlite3.connect(DATABASE_NAME)) as connection:
    with connection:
        connection.execute(
            "UPDATE students SET age = ? WHERE id = ?",
            (21, student_id)
        )
```

در این الگو، `closing` اتصال را می‌بندد و `with connection` تراکنش را مدیریت می‌کند.

## ۶. الگوی پیشنهادی برای لایه پایگاه داده

فایل `database.py` بهتر است عملیات پایگاه داده را انجام دهد، نه نمایش پیام‌های Tkinter. به این ترتیب می‌توان آن را در برنامه‌های دیگر نیز استفاده کرد.

نمونه تابع:

```python
import sqlite3

DATABASE_NAME = "school.db"


def add_student(name, age, city):
    with sqlite3.connect(DATABASE_NAME) as connection:
        cursor = connection.execute(
            "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
            (name, age, city)
        )
        return cursor.lastrowid
```

سپس رابط گرافیکی خطا را مدیریت می‌کند:

```python
try:
    student_id = database.add_student(name, age, city)
except sqlite3.IntegrityError:
    messagebox.showwarning(
        "Data Error",
        "The record violates a database rule."
    )
except sqlite3.Error:
    logging.exception("Could not add student")
    messagebox.showerror(
        "Database Error",
        "The student could not be saved."
    )
else:
    messagebox.showinfo(
        "Saved",
        f"Student saved with ID {student_id}."
    )
```

این جداسازی باعث می‌شود لایه پایگاه داده مستقل از رابط گرافیکی باقی بماند.

## ۷. پروژه کلاسی: افزودن رکورد با مدیریت خطا

کد زیر نمونه‌ای کوچک و مستقل برای تمرین است. در پروژه چندفایلی قبلی می‌توانید همین الگو را به متد `add_record()` منتقل کنید.

```python
import logging
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox


DATABASE_NAME = "school.db"

logging.basicConfig(
    filename="app.log",
    level=logging.ERROR,
    format="%(asctime)s %(levelname)s %(message)s"
)


def initialize_database():
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                age INTEGER NOT NULL CHECK (age BETWEEN 1 AND 120),
                city TEXT NOT NULL
            )
        """)


def save_student(name, age, city):
    with sqlite3.connect(DATABASE_NAME) as connection:
        cursor = connection.execute(
            "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
            (name, age, city)
        )
        return cursor.lastrowid


class StudentForm:
    def __init__(self, root):
        self.root = root
        self.root.title("Validated Student Form")
        self.root.geometry("420x280")

        ttk.Label(root, text="Name:").grid(
            row=0, column=0, padx=10, pady=10, sticky="w"
        )
        self.name_entry = ttk.Entry(root)
        self.name_entry.grid(
            row=0, column=1, padx=10, pady=10, sticky="ew"
        )

        ttk.Label(root, text="Age:").grid(
            row=1, column=0, padx=10, pady=10, sticky="w"
        )
        self.age_entry = ttk.Entry(root)
        self.age_entry.grid(
            row=1, column=1, padx=10, pady=10, sticky="ew"
        )

        ttk.Label(root, text="City:").grid(
            row=2, column=0, padx=10, pady=10, sticky="w"
        )
        self.city_entry = ttk.Entry(root)
        self.city_entry.grid(
            row=2, column=1, padx=10, pady=10, sticky="ew"
        )

        ttk.Button(
            root,
            text="Save Student",
            command=self.save
        ).grid(row=3, column=1, padx=10, pady=12, sticky="e")

        root.columnconfigure(1, weight=1)

    def save(self):
        name = self.name_entry.get().strip()
        age_text = self.age_entry.get().strip()
        city = self.city_entry.get().strip()

        if not name or not age_text or not city:
            messagebox.showwarning(
                "Missing Information",
                "Please fill in all fields."
            )
            return

        try:
            age = int(age_text)
        except ValueError:
            messagebox.showwarning(
                "Invalid Age",
                "Age must be a whole number."
            )
            self.age_entry.focus_set()
            return

        if not 1 <= age <= 120:
            messagebox.showwarning(
                "Invalid Age",
                "Age must be between 1 and 120."
            )
            self.age_entry.focus_set()
            return

        try:
            student_id = save_student(name, age, city)
        except sqlite3.IntegrityError:
            messagebox.showwarning(
                "Data Error",
                "The data violates a database rule."
            )
        except sqlite3.Error:
            logging.exception("Failed to save student")
            messagebox.showerror(
                "Database Error",
                "The student could not be saved."
            )
        else:
            messagebox.showinfo(
                "Saved",
                f"Student saved with ID {student_id}."
            )
            self.name_entry.delete(0, tk.END)
            self.age_entry.delete(0, tk.END)
            self.city_entry.delete(0, tk.END)
            self.name_entry.focus_set()


if __name__ == "__main__":
    initialize_database()
    root = tk.Tk()
    app = StudentForm(root)
    root.mainloop()
```

### نکته مهم درباره ساختار جدول موجود

`CREATE TABLE IF NOT EXISTS` ساختار جدولی را که از قبل وجود دارد تغییر نمی‌دهد. اگر فایل `school.db` از جلسه قبل ساخته شده باشد، قید `CHECK` جدید خودبه‌خود به جدول قبلی اضافه نخواهد شد. اعتبارسنجی برنامه همچنان انجام می‌شود؛ برای تغییر ساختار پایگاه داده باید از مهاجرت یا بازسازی کنترل‌شده جدول استفاده کرد.

## ۸. تمرین‌های کلاسی

### تمرین ۱ — جلوگیری از ورودی خالی

تابع ذخیره را طوری تغییر دهید که نام و شهر فقط شامل فاصله نیز پذیرفته نشوند.

**پاسخ:** از `strip()` استفاده کنید و سپس مقدار را بررسی کنید:

```python
name = name_entry.get().strip()
if not name:
    messagebox.showwarning(
        "Invalid Name",
        "Name cannot be empty."
    )
    return
```

### تمرین ۲ — ثبت خطا در فایل

اگر ذخیره‌سازی با خطای SQLite مواجه شد، جزئیات فنی در `app.log` ثبت شود، اما پیام عمومی به کاربر نشان داده شود.

**پاسخ:** در بلوک `except sqlite3.Error` از `logging.exception("Failed to save student")` استفاده کنید.

### تمرین ۳ — جلوگیری از سن نامعتبر در پایگاه داده

توضیح دهید چرا اعتبارسنجی رابط گرافیکی به‌تنهایی کافی نیست.

**پاسخ:** ممکن است داده از مسیر دیگری وارد پایگاه داده شود. محدودیت‌هایی مانند `CHECK` و `NOT NULL` در خود پایگاه داده نیز می‌توانند از ذخیره بعضی داده‌های نامعتبر جلوگیری کنند. البته افزودن محدودیت به جدول موجود نیازمند تغییر ساختار است.

### تمرین ۴ — جداسازی مسئولیت‌ها

تابعی برای افزودن دانشجو در `database.py` بنویسید که هیچ `messagebox`ای در آن وجود نداشته باشد.

**پاسخ:** تابع باید عملیات SQL را انجام دهد و در صورت خطا استثنا را به لایه رابط گرافیکی منتقل کند. نمایش پیام در `ui.py` یا لایه رابط باقی می‌ماند.

## ۹. خطاهای رایج

- **گرفتن همه خطاها بدون ثبت جزئیات:** در زمان توسعه، خطا را با `logging.exception()` ثبت کنید.
- **نمایش متن فنی طولانی به کاربر:** پیام کاربر باید ساده باشد و جزئیات در گزارش ثبت شود.
- **تکرار اعتبارسنجی در چند متد:** منطق مشترک را در تابعی مستقل متمرکز کنید.
- **استفاده از `except` بسیار عمومی:** تا جای ممکن نوع خطای مورد انتظار را مشخص کنید.
- **تصور اینکه قید جدید خودکار به جدول قبلی اضافه می‌شود:** `CREATE TABLE IF NOT EXISTS` ساختار موجود را تغییر نمی‌دهد.
- **قرار دادن پیام‌های Tkinter در `database.py`:** لایه پایگاه داده را از رابط گرافیکی مستقل نگه دارید.

## ۱۰. جمع‌بندی و چک‌لیست مدرس

- [ ] تفاوت اعتبارسنجی و مدیریت خطا را توضیح دادم.
- [ ] ورودی خالی و سن نامعتبر را بررسی کردیم.
- [ ] خطاهای SQLite را به‌صورت کنترل‌شده مدیریت کردیم.
- [ ] استفاده از `logging` را نشان دادم.
- [ ] مسئولیت رابط گرافیکی و لایه پایگاه داده را جدا کردیم.
- [ ] محدودیت‌های پایگاه داده موجود را توضیح دادم.

## جلسه بعد

در جلسه ۱۷ می‌توانیم پنجره‌های تأیید و تنظیمات برنامه، نمایش بهتر پیام‌ها و مدیریت پیکربندی را بررسی کنیم؛ سپس برای پروژه نهایی، طراحی صفحه‌ها و معماری کلی برنامه را تثبیت کنیم.
