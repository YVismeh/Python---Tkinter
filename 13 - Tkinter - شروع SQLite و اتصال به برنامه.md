---
title: "Tkinter - جلسه ۱۳ - شروع SQLite و اتصال به برنامه"
aliases:
  - Tkinter Session 13
  - SQLite with Tkinter
tags:
  - Python
  - Tkinter
  - SQLite
  - Database
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۳ — شروع SQLite و اتصال به Tkinter

## هدف جلسه

تا اینجا اطلاعات برنامه‌های Tkinter معمولاً داخل متغیرهای پایتون نگهداری می‌شدند؛ بنابراین با بسته شدن برنامه، داده‌ها از بین می‌رفتند. در این جلسه با SQLite آشنا می‌شویم و یک پایگاه داده محلی می‌سازیم تا اطلاعات دانشجوها پس از بسته شدن برنامه نیز باقی بمانند.

در پایان جلسه دانشجو باید بتواند:

- مفهوم پایگاه داده و جدول را توضیح دهد.
- با ماژول استاندارد `sqlite3` به پایگاه داده متصل شود.
- یک جدول ایجاد کند.
- اطلاعات را با `INSERT` ذخیره کند.
- اطلاعات را با `SELECT` بخواند.
- تغییرات را با `commit()` ثبت کند.
- نتیجه عملیات پایگاه داده را در `Treeview` نمایش دهد.

## ۱. پایگاه داده چیست؟

پایگاه داده روشی سازمان‌یافته برای نگهداری و بازیابی اطلاعات است. برای مثال، برنامه مدیریت دانشجو به اطلاعاتی مانند شناسه، نام، سن و شهر نیاز دارد.

در جلسات قبلی ممکن بود اطلاعات را در یک فهرست ذخیره کنیم:

```python
students = [
    {"id": 1, "name": "Sara", "age": 20, "city": "Tehran"},
    {"id": 2, "name": "Amir", "age": 22, "city": "Tabriz"}
]
```

این فهرست فقط در حافظه برنامه قرار دارد. با بسته شدن برنامه، اطلاعاتی که ذخیره دائمی نشده‌اند از بین می‌روند.

SQLite اطلاعات را در یک فایل محلی نگه می‌دارد؛ برای استفاده معمول از آن، نصب و راه‌اندازی یک سرور جداگانه لازم نیست. ماژول `sqlite3` نیز همراه پایتون ارائه می‌شود.

## ۲. مفاهیم پایه

| اصطلاح | توضیح |
|---|---|
| Database | مجموعه سازمان‌یافته‌ای از اطلاعات |
| Table | جدولی برای نگهداری یک نوع اطلاعات |
| Row / Record | یک رکورد، مانند اطلاعات یک دانشجو |
| Column / Field | یک ویژگی، مانند نام یا سن |
| Primary Key | شناسه یکتای هر رکورد |
| SQL | زبان تعریف، بازیابی و تغییر داده‌ها |

در این جلسه یک جدول به نام `students` ایجاد می‌کنیم.

## ۳. ساخت اولین پایگاه داده

فایل زیر را با نام `database_demo.py` ذخیره و اجرا کنید:

```python
import sqlite3

connection = sqlite3.connect("school.db")
cursor = connection.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS students (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER NOT NULL,
        city TEXT NOT NULL
    )
""")

connection.commit()
connection.close()

print("Database and table are ready.")
```

### توضیح کد

- `sqlite3.connect("school.db")` به فایل پایگاه داده متصل می‌شود. اگر فایل وجود نداشته باشد، معمولاً ساخته می‌شود.
- `cursor()` ابزاری برای اجرای دستورهای SQL ایجاد می‌کند.
- `CREATE TABLE IF NOT EXISTS` جدول را فقط زمانی می‌سازد که از قبل وجود نداشته باشد.
- `INTEGER PRIMARY KEY AUTOINCREMENT` شناسه عددی خودکار تعریف می‌کند.
- `TEXT NOT NULL` مشخص می‌کند مقدار متنی این ستون نباید خالی از نوع `NULL` باشد.
- `commit()` تغییرات را ثبت می‌کند.
- `close()` اتصال را می‌بندد.

**نکته:** فایل `school.db` معمولاً در پوشه کاری فعلی برنامه ایجاد می‌شود. اگر می‌خواهید محل آن مشخص باشد، از مسیر فایل مشخص استفاده کنید.

## ۴. افزودن اطلاعات با INSERT

```python
import sqlite3

connection = sqlite3.connect("school.db")
cursor = connection.cursor()

cursor.execute(
    "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
    ("Sara", 20, "Tehran")
)

connection.commit()
connection.close()

print("Student saved.")
```

علامت‌های `?` جایگاه پارامترها هستند. مقادیر جداگانه به دستور داده می‌شوند؛ این شیوه از اتصال مستقیم ورودی کاربر به متن SQL ایمن‌تر است.

از این روش استفاده نکنید:

```python
# Avoid building SQL by concatenating user input.
query = "INSERT INTO students (name) VALUES ('" + user_name + "')"
```

به‌جای آن از پارامترهای `?` استفاده کنید.

## ۵. خواندن اطلاعات با SELECT

```python
import sqlite3

connection = sqlite3.connect("school.db")
cursor = connection.cursor()

cursor.execute("SELECT id, name, age, city FROM students")
rows = cursor.fetchall()

for row in rows:
    print(row)

connection.close()
```

`fetchall()` همه ردیف‌های نتیجه را برمی‌گرداند. هر ردیف معمولاً به شکل یک tuple است؛ برای مثال:

```text
(1, 'Sara', 20, 'Tehran')
```

## ۶. به‌روزرسانی و حذف رکوردها

به‌روزرسانی:

```python
cursor.execute(
    "UPDATE students SET age = ? WHERE id = ?",
    (21, 1)
)
connection.commit()
```

حذف:

```python
cursor.execute(
    "DELETE FROM students WHERE id = ?",
    (1,)
)
connection.commit()
```

در مثال حذف، `(1,)` یک tuple تک‌عضوی است. ویرگول انتهایی اهمیت دارد.

**هشدار مهم:** قبل از اجرای `UPDATE` یا `DELETE` مطمئن شوید شرط `WHERE` درست است. اجرای این دستورها بدون شرط مناسب ممکن است تعداد زیادی از رکوردها را تغییر دهد یا حذف کند.

## ۷. اتصال SQLite به Treeview

در این مرحله برنامه‌ای می‌سازیم که:

- هنگام اجرا جدول پایگاه داده را آماده می‌کند.
- رکوردهای ذخیره‌شده را در `Treeview` نمایش می‌دهد.
- با فرم ورودی، دانشجو اضافه می‌کند.
- با انتخاب یک ردیف و فشردن دکمه، آن رکورد را حذف می‌کند.
- پس از هر عملیات، جدول را از پایگاه داده دوباره بارگذاری می‌کند.

کد کامل زیر را با نام `student_database_app.py` ذخیره کنید.

```python
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox


DATABASE_NAME = "school.db"


def create_database():
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                age INTEGER NOT NULL,
                city TEXT NOT NULL
            )
        """)


class StudentDatabaseApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Database")
        self.root.geometry("760x500")
        self.root.minsize(620, 400)

        create_database()
        self.build_ui()
        self.load_students()

    def build_ui(self):
        form = ttk.LabelFrame(self.root, text="Student Information", padding=10)
        form.pack(fill="x", padx=12, pady=12)

        ttk.Label(form, text="Name:").grid(row=0, column=0, padx=5, pady=5)
        self.name_entry = ttk.Entry(form)
        self.name_entry.grid(row=0, column=1, sticky="ew", padx=5, pady=5)

        ttk.Label(form, text="Age:").grid(row=0, column=2, padx=5, pady=5)
        self.age_entry = ttk.Entry(form, width=10)
        self.age_entry.grid(row=0, column=3, padx=5, pady=5)

        ttk.Label(form, text="City:").grid(row=1, column=0, padx=5, pady=5)
        self.city_entry = ttk.Entry(form)
        self.city_entry.grid(row=1, column=1, sticky="ew", padx=5, pady=5)

        ttk.Button(
            form,
            text="Add Student",
            command=self.add_student
        ).grid(row=1, column=3, padx=5, pady=5)

        form.columnconfigure(1, weight=1)

        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=12, pady=(0, 8))

        columns = ("id", "name", "age", "city")
        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            selectmode="browse"
        )

        for column, title in (
            ("id", "ID"),
            ("name", "Name"),
            ("age", "Age"),
            ("city", "City")
        ):
            self.tree.heading(column, text=title)

        self.tree.column("id", width=60, anchor="center")
        self.tree.column("name", width=220)
        self.tree.column("age", width=80, anchor="center")
        self.tree.column("city", width=180)

        scrollbar = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.tree.yview
        )
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        buttons = ttk.Frame(self.root)
        buttons.pack(fill="x", padx=12, pady=(0, 12))

        ttk.Button(
            buttons,
            text="Delete Selected",
            command=self.delete_selected
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            buttons,
            text="Refresh",
            command=self.load_students
        ).pack(side="left", padx=6)

        self.status_label = ttk.Label(buttons, text="")
        self.status_label.pack(side="right")

    def load_students(self):
        with sqlite3.connect(DATABASE_NAME) as connection:
            rows = connection.execute(
                "SELECT id, name, age, city FROM students ORDER BY id"
            ).fetchall()

        children = self.tree.get_children()
        if children:
            self.tree.delete(*children)

        for row in rows:
            self.tree.insert("", tk.END, iid=str(row[0]), values=row)

        self.status_label.config(text=f"{len(rows)} student(s)")

    def add_student(self):
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
            if age < 1 or age > 120:
                raise ValueError
        except ValueError:
            messagebox.showwarning(
                "Invalid Age",
                "Enter an age from 1 to 120."
            )
            return

        with sqlite3.connect(DATABASE_NAME) as connection:
            connection.execute(
                "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
                (name, age, city)
            )

        self.name_entry.delete(0, tk.END)
        self.age_entry.delete(0, tk.END)
        self.city_entry.delete(0, tk.END)
        self.load_students()
        messagebox.showinfo("Saved", "Student saved successfully.")

    def delete_selected(self):
        selection = self.tree.selection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student first."
            )
            return

        student_id = int(selection[0])

        confirmed = messagebox.askyesno(
            "Confirm Delete",
            "Delete the selected student?"
        )
        if not confirmed:
            return

        with sqlite3.connect(DATABASE_NAME) as connection:
            connection.execute(
                "DELETE FROM students WHERE id = ?",
                (student_id,)
            )

        self.load_students()


if __name__ == "__main__":
    root = tk.Tk()
    app = StudentDatabaseApp(root)
    root.mainloop()
```

## ۸. نکات مهم پروژه

### چرا `with sqlite3.connect(...)` استفاده شده است؟

این الگو مدیریت تراکنش را ساده‌تر می‌کند و در پایان بلوک، اتصال را می‌بندد. اگر عملیاتی موفق باشد، تغییرات به‌طور خودکار ثبت می‌شوند؛ اگر خطایی رخ دهد، تراکنش بازگشت داده می‌شود. برای کنترل دقیق‌تر اتصال و تراکنش در برنامه‌های بزرگ‌تر، می‌توان مدیریت آن‌ها را صریح‌تر نوشت.

### چرا از `?` استفاده شده است؟

ورودی کاربر نباید با الحاق رشته‌ها مستقیماً وارد دستور SQL شود. پارامترگذاری از خطاهای نقل‌قول و حملات تزریق SQL جلوگیری می‌کند.

### چرا پس از افزودن یا حذف، `load_students()` اجرا می‌شود؟

پایگاه داده منبع اصلی اطلاعات است. بعد از هر تغییر، دوباره داده‌ها را از پایگاه داده می‌خوانیم تا جدول با وضعیت واقعی ذخیره‌شده هماهنگ بماند.

### آیا اطلاعات با بستن برنامه باقی می‌ماند؟

بله. رکوردها در فایل `school.db` ذخیره می‌شوند. اگر فایل را حذف کنید یا برنامه به فایل دیگری متصل شود، داده‌های قبلی در دسترس نخواهند بود.

## ۹. تمرین‌های کلاسی

### تمرین ۱ — جلوگیری از تکرار نام

جدول را طوری تغییر دهید که دو دانشجو نتوانند دقیقاً با یک نام و شهر یکسان ثبت شوند.

**راهنمای پاسخ:** می‌توانید یک محدودیت `UNIQUE(name, city)` به تعریف جدول اضافه کنید. این محدودیت برای جدول موجود به‌سادگی با تغییر دستور `CREATE TABLE` اعمال نمی‌شود؛ برای جدول موجود باید راهکار مهاجرت یا بازسازی جدول را بررسی کنید.

### تمرین ۲ — جست‌وجوی نام در SQLite

یک ورودی جست‌وجو و دکمه Search اضافه کنید تا نام‌های مطابق عبارت واردشده نمایش داده شوند.

**پاسخ پیشنهادی:**

```python
query = self.search_entry.get().strip()

with sqlite3.connect(DATABASE_NAME) as connection:
    rows = connection.execute(
        """
        SELECT id, name, age, city
        FROM students
        WHERE name LIKE ?
        ORDER BY name
        """,
        (f"%{query}%",)
    ).fetchall()
```

سپس جدول را مانند `load_students()` پاک و با `rows` پر کنید. در SQLite، رفتار `LIKE` برای حروف انگلیسی در حالت پیش‌فرض معمولاً نسبت به بزرگی و کوچکی حروف حساس نیست؛ برای همه زبان‌ها و نیازها نباید این رفتار را یکسان فرض کرد.

### تمرین ۳ — نمایش تعداد رکوردها

پس از بارگذاری جدول، تعداد رکوردها را در یک `Label` نمایش دهید.

**پاسخ:** تعداد ردیف‌های بازیابی‌شده برابر `len(rows)` است؛ در پایان `load_students()` متن `status_label` را به‌روزرسانی کنید.

### تمرین ۴ — مرتب‌سازی بر اساس سن

داده‌ها را از پایگاه داده به ترتیب سن نمایش دهید.

**پاسخ:**

```sql
SELECT id, name, age, city
FROM students
ORDER BY age ASC
```

برای ترتیب نزولی از `DESC` استفاده کنید.

## ۱۰. خطاهای رایج

- **استفاده از `INSERT` بدون پارامترگذاری:** از `?` و tuple پارامترها استفاده کنید.
- **فراموش کردن `commit()` در الگوی اتصال دستی:** تغییرات را ثبت کنید. در الگوی `with connection`، تراکنش موفق در پایان بلوک ثبت می‌شود.
- **اجرای `DELETE` بدون `WHERE`:** ممکن است تمام رکوردهای جدول حذف شوند.
- **فرض کردن اینکه فایل پایگاه داده کنار فایل کد است:** مسیر نسبی به پوشه کاری فعلی وابسته است.
- **به‌روزرسانی نکردن Treeview:** پس از تغییر داده‌ها، جدول را دوباره از پایگاه داده بارگذاری کنید.
- **ثبت سن نامعتبر:** مقدار ورودی را قبل از ذخیره به عدد تبدیل و اعتبارسنجی کنید.

## ۱۱. جمع‌بندی جلسه

در این جلسه نخستین اتصال عملی بین Tkinter و SQLite را ایجاد کردیم. برنامه می‌تواند رکوردهای دانشجو را اضافه کند، بخواند و حذف کند؛ اطلاعات نیز پس از بسته شدن برنامه باقی می‌مانند.

## چک‌لیست مدرس

- [ ] تفاوت حافظه موقت و پایگاه داده را توضیح دادم.
- [ ] فایل `school.db` را ایجاد کردم.
- [ ] `CREATE TABLE`, `INSERT`, `SELECT` و `DELETE` را توضیح دادم.
- [ ] پارامترگذاری با `?` را نشان دادم.
- [ ] افزودن و حذف از رابط گرافیکی را آزمایش کردم.
- [ ] برنامه را بستم و دوباره اجرا کردم تا ماندگاری داده‌ها را بررسی کنم.

## جلسه بعد

در جلسه ۱۴ عملیات ویرایش رکورد را اضافه می‌کنیم و چرخه کامل CRUD را می‌سازیم: Create، Read، Update و Delete. سپس می‌توانیم جست‌وجو و مرتب‌سازی را به کوئری‌های SQLite منتقل کنیم.
