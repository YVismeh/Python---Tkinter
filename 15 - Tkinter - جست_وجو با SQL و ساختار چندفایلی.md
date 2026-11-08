---
title: "Tkinter - جلسه ۱۵ - جست‌وجو با SQL و ساختار چندفایلی"
aliases:
  - Tkinter Session 15
  - SQL Search and Multi-file Architecture
tags:
  - Python
  - Tkinter
  - SQLite
  - SQL
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۵ — جست‌وجو با SQL و ساختاردهی برنامه در چند فایل

## هدف جلسه

در جلسه‌های قبل، برنامه مدیریت دانشجو را به SQLite وصل کردیم و عملیات CRUD را پیاده‌سازی کردیم. در این جلسه جست‌وجو و مرتب‌سازی را به خود پایگاه داده می‌سپاریم و سپس کد را به چند فایل تقسیم می‌کنیم.

در پایان جلسه دانشجو باید بتواند:

- با `WHERE` داده‌ها را فیلتر کند.
- از `LIKE` برای جست‌وجوی بخشی از متن استفاده کند.
- با `ORDER BY` نتایج را مرتب کند.
- مقادیر ورودی را به‌صورت پارامتری به SQL بدهد.
- منطق پایگاه داده را از رابط گرافیکی جدا کند.
- یک پروژه چندفایلی ساده را اجرا کند.

## ۱. چرا جست‌وجو را به SQLite بسپاریم؟

در برنامه کوچک، می‌توان همه رکوردها را از پایگاه داده خواند و با پایتون فیلتر کرد. اما اگر تعداد رکوردها زیاد شود، خواندن همه داده‌ها در هر بار جست‌وجو می‌تواند غیرضروری باشد.

به‌جای این کار، شرط را به SQL می‌دهیم تا پایگاه داده فقط نتیجه‌های لازم را برگرداند.

مثلاً:

```sql
SELECT id, name, age, city
FROM students
WHERE city = ?
ORDER BY name ASC;
```

این دستور فقط دانشجوهای شهر مشخص‌شده را برمی‌گرداند و نتیجه را بر اساس نام مرتب می‌کند.

## ۲. استفاده از WHERE

نمونه جست‌وجوی شهر:

```python
with sqlite3.connect(DATABASE_NAME) as connection:
    rows = connection.execute(
        """
        SELECT id, name, age, city
        FROM students
        WHERE city = ?
        ORDER BY name ASC
        """,
        ("Tehran",)
    ).fetchall()
```

در این مثال، `"Tehran"` به‌صورت پارامتر به دستور داده می‌شود. برای یک پارامتر نیز باید tuple تک‌عضوی بسازیم؛ به همین دلیل بعد از مقدار ویرگول آمده است: `("Tehran",)`.

## ۳. جست‌وجوی بخشی از متن با LIKE

اگر کاربر `sa` را وارد کند و بخواهیم نام‌هایی مانند `Sara` را پیدا کنیم، می‌توان از `LIKE` و علامت `%` استفاده کرد.

```python
query = "sa"

with sqlite3.connect(DATABASE_NAME) as connection:
    rows = connection.execute(
        """
        SELECT id, name, age, city
        FROM students
        WHERE name LIKE ?
        ORDER BY name ASC
        """,
        (f"%{query}%",)
    ).fetchall()
```

- `sa%` یعنی متن با `sa` شروع شود.
- `%sa` یعنی متن با `sa` تمام شود.
- `%sa%` یعنی متن در هر جای مقدار ظاهر شود.

در SQLite، `LIKE` برای حروف انگلیسی در حالت پیش‌فرض معمولاً نسبت به بزرگی و کوچکی حروف حساس نیست؛ این رفتار را برای همه زبان‌ها و همه نیازها یکسان فرض نکنید.

## ۴. مرتب‌سازی با ORDER BY

مرتب‌سازی صعودی:

```sql
SELECT id, name, age, city
FROM students
ORDER BY age ASC;
```

مرتب‌سازی نزولی:

```sql
SELECT id, name, age, city
FROM students
ORDER BY age DESC;
```

مرتب‌سازی چندمرحله‌ای:

```sql
SELECT id, name, age, city
FROM students
ORDER BY city ASC, name ASC;
```

ابتدا شهر مرتب می‌شود و در داخل هر شهر، نام‌ها مرتب می‌شوند.

### نکته امنیتی درباره نام ستون

مقادیر ورودی را می‌توان با `?` پارامتری کرد؛ اما نام ستون را نباید مستقیماً از ورودی ناشناس کاربر در SQL قرار داد. برای انتخاب ستون مرتب‌سازی، از فهرست مجاز استفاده کنید:

```python
allowed_columns = {
    "name": "name",
    "age": "age",
    "city": "city"
}

column = allowed_columns.get(requested_column, "name")
direction = "ASC" if requested_direction == "asc" else "DESC"

sql = (
    "SELECT id, name, age, city "
    "FROM students "
    f"ORDER BY {column} {direction}"
)
```

این روش فقط زمانی مناسب است که `requested_column` از طریق فهرست مجاز انتخاب شود و جهت مرتب‌سازی نیز به `ASC` یا `DESC` محدود شده باشد.

## ۵. ساختار چندفایلی پروژه

وقتی برنامه بزرگ‌تر می‌شود، قرار دادن همه کدها در یک فایل خوانایی و نگهداری را دشوار می‌کند. در این جلسه منطق پایگاه داده و رابط گرافیکی را جدا می‌کنیم.

ساختار پیشنهادی:

```text
student_app/
├── main.py
├── database.py
└── ui.py
```

- `database.py`: ایجاد جدول و اجرای عملیات پایگاه داده
- `ui.py`: ساخت رابط گرافیکی و مدیریت رویدادها
- `main.py`: نقطه شروع برنامه

سه فایل را در یک پوشه قرار دهید.

## ۶. فایل database.py

```python
import sqlite3

DATABASE_NAME = "school.db"


def initialize_database():
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                age INTEGER NOT NULL,
                city TEXT NOT NULL
            )
        """)


def get_students(search_text="", city="All", sort_by="name", descending=False):
    allowed_columns = {
        "id": "id",
        "name": "name",
        "age": "age",
        "city": "city"
    }
    column = allowed_columns.get(sort_by, "name")
    direction = "DESC" if descending else "ASC"

    conditions = []
    parameters = []

    if search_text.strip():
        conditions.append("(name LIKE ? OR city LIKE ?)")
        pattern = f"%{search_text.strip()}%"
        parameters.extend([pattern, pattern])

    if city != "All":
        conditions.append("city = ?")
        parameters.append(city)

    where_clause = ""
    if conditions:
        where_clause = "WHERE " + " AND ".join(conditions)

    sql = f"""
        SELECT id, name, age, city
        FROM students
        {where_clause}
        ORDER BY {column} {direction}
    """

    with sqlite3.connect(DATABASE_NAME) as connection:
        return connection.execute(sql, parameters).fetchall()


def add_student(name, age, city):
    with sqlite3.connect(DATABASE_NAME) as connection:
        connection.execute(
            "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
            (name, age, city)
        )


def update_student(student_id, name, age, city):
    with sqlite3.connect(DATABASE_NAME) as connection:
        cursor = connection.execute(
            """
            UPDATE students
            SET name = ?, age = ?, city = ?
            WHERE id = ?
            """,
            (name, age, city, student_id)
        )
        return cursor.rowcount > 0


def delete_student(student_id):
    with sqlite3.connect(DATABASE_NAME) as connection:
        cursor = connection.execute(
            "DELETE FROM students WHERE id = ?",
            (student_id,)
        )
        return cursor.rowcount > 0
```

### نکته درباره ساخت کوئری

نام ستون و جهت مرتب‌سازی از مقادیر کنترل‌شده انتخاب می‌شوند. مقدارهای جست‌وجو و شهر از طریق پارامترها ارسال می‌شوند. این تفاوت مهم است: پارامترگذاری برای داده‌هاست، نه برای نام ستون یا کلمه کلیدی SQL.

## ۷. فایل ui.py

```python
import tkinter as tk
from tkinter import ttk, messagebox

import database


class StudentApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Manager")
        self.root.geometry("850x560")
        self.selected_id = None
        self.sort_by = "name"
        self.descending = False

        self.build_ui()
        self.refresh_table()

    def build_ui(self):
        form = ttk.LabelFrame(
            self.root,
            text="Student Information",
            padding=10
        )
        form.pack(fill="x", padx=12, pady=12)

        ttk.Label(form, text="Name:").grid(
            row=0, column=0, padx=5, pady=5, sticky="w"
        )
        self.name_entry = ttk.Entry(form)
        self.name_entry.grid(
            row=0, column=1, padx=5, pady=5, sticky="ew"
        )

        ttk.Label(form, text="Age:").grid(
            row=0, column=2, padx=5, pady=5, sticky="w"
        )
        self.age_entry = ttk.Entry(form, width=10)
        self.age_entry.grid(row=0, column=3, padx=5, pady=5)

        ttk.Label(form, text="City:").grid(
            row=1, column=0, padx=5, pady=5, sticky="w"
        )
        self.city_entry = ttk.Entry(form)
        self.city_entry.grid(
            row=1, column=1, padx=5, pady=5, sticky="ew"
        )

        ttk.Button(
            form,
            text="Add",
            command=self.add_record
        ).grid(row=1, column=3, padx=5, pady=5)

        form.columnconfigure(1, weight=1)

        filters = ttk.LabelFrame(
            self.root,
            text="Search and Filter",
            padding=10
        )
        filters.pack(fill="x", padx=12, pady=(0, 10))

        ttk.Label(filters, text="Search:").grid(
            row=0, column=0, padx=5, pady=5
        )
        self.search_entry = ttk.Entry(filters)
        self.search_entry.grid(
            row=0, column=1, sticky="ew", padx=5, pady=5
        )
        self.search_entry.bind(
            "<KeyRelease>",
            lambda event: self.refresh_table()
        )

        ttk.Label(filters, text="City:").grid(
            row=0, column=2, padx=5, pady=5
        )
        self.city_combo = ttk.Combobox(
            filters,
            values=("All", "Tehran", "Tabriz", "Shiraz", "Isfahan"),
            state="readonly",
            width=12
        )
        self.city_combo.set("All")
        self.city_combo.grid(row=0, column=3, padx=5, pady=5)
        self.city_combo.bind(
            "<<ComboboxSelected>>",
            lambda event: self.refresh_table()
        )

        ttk.Button(
            filters,
            text="Clear",
            command=self.clear_filters
        ).grid(row=0, column=4, padx=5, pady=5)

        filters.columnconfigure(1, weight=1)

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
            self.tree.heading(
                column,
                text=title,
                command=lambda col=column: self.change_sort(col)
            )

        self.tree.column("id", width=60, anchor="center")
        self.tree.column("name", width=230)
        self.tree.column("age", width=90, anchor="center")
        self.tree.column("city", width=180)

        scrollbar = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.tree.yview
        )
        self.tree.configure(yscrollcommand=scrollbar.set)
        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        self.tree.bind("<<TreeviewSelect>>", self.on_select)

        actions = ttk.Frame(self.root)
        actions.pack(fill="x", padx=12, pady=(0, 12))

        ttk.Button(
            actions,
            text="Update Selected",
            command=self.update_record
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            actions,
            text="Delete Selected",
            command=self.delete_record
        ).pack(side="left", padx=6)

        ttk.Button(
            actions,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="left", padx=6)

        self.status_label = ttk.Label(actions, text="")
        self.status_label.pack(side="right")

    def refresh_table(self):
        rows = database.get_students(
            search_text=self.search_entry.get(),
            city=self.city_combo.get(),
            sort_by=self.sort_by,
            descending=self.descending
        )

        children = self.tree.get_children()
        if children:
            self.tree.delete(*children)

        for row in rows:
            self.tree.insert("", tk.END, iid=str(row[0]), values=row)

        self.status_label.config(text=f"{len(rows)} result(s)")

    def change_sort(self, column):
        if self.sort_by == column:
            self.descending = not self.descending
        else:
            self.sort_by = column
            self.descending = False

        self.refresh_table()

    def clear_filters(self):
        self.search_entry.delete(0, tk.END)
        self.city_combo.set("All")
        self.refresh_table()

    def get_form_data(self):
        name = self.name_entry.get().strip()
        age_text = self.age_entry.get().strip()
        city = self.city_entry.get().strip()

        if not name or not age_text or not city:
            messagebox.showwarning(
                "Missing Information",
                "Please fill in all fields."
            )
            return None

        try:
            age = int(age_text)
            if age < 1 or age > 120:
                raise ValueError
        except ValueError:
            messagebox.showwarning(
                "Invalid Age",
                "Enter an age from 1 to 120."
            )
            return None

        return name, age, city

    def add_record(self):
        data = self.get_form_data()
        if data is None:
            return

        database.add_student(*data)
        self.clear_form()
        self.refresh_table()

    def on_select(self, event):
        selection = self.tree.selection()
        if not selection:
            return

        self.selected_id = int(selection[0])

    def update_record(self):
        if self.selected_id is None:
            messagebox.showinfo(
                "No Selection",
                "Select a row first."
            )
            return

        data = self.get_form_data()
        if data is None:
            return

        updated = database.update_student(self.selected_id, *data)
        self.refresh_table()
        self.clear_form()

        if not updated:
            messagebox.showwarning(
                "Not Found",
                "The selected record no longer exists."
            )

    def delete_record(self):
        selection = self.tree.selection()
        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a row first."
            )
            return

        student_id = int(selection[0])
        confirmed = messagebox.askyesno(
            "Confirm Delete",
            "Delete the selected student?"
        )
        if not confirmed:
            return

        database.delete_student(student_id)
        self.refresh_table()
        self.clear_form()

    def clear_form(self):
        self.name_entry.delete(0, tk.END)
        self.age_entry.delete(0, tk.END)
        self.city_entry.delete(0, tk.END)
        self.selected_id = None
        self.tree.selection_remove(self.tree.selection())
```

## ۸. فایل main.py

```python
import tkinter as tk

import database
from ui import StudentApp


def main():
    database.initialize_database()

    root = tk.Tk()
    StudentApp(root)
    root.mainloop()


if __name__ == "__main__":
    main()
```

### اجرای پروژه

ساختار پوشه باید دقیقاً به این صورت باشد:

```text
student_app/
├── main.py
├── database.py
└── ui.py
```

از داخل همین پوشه اجرا کنید:

```bash
python main.py
```

هر سه فایل باید در یک پوشه باشند تا importها به‌درستی کار کنند. فایل `school.db` نیز در پوشه کاری برنامه ساخته می‌شود.

## ۹. تمرین‌های کلاسی

### تمرین ۱ — جست‌وجوی شناسه

جست‌وجو را طوری توسعه دهید که شناسه دانشجو هم بررسی شود.

**پاسخ پیشنهادی:** در `database.py` شرط جست‌وجو را از این:

```python
conditions.append("(name LIKE ? OR city LIKE ?)")
```

به این تغییر دهید:

```python
conditions.append(
    "(CAST(id AS TEXT) LIKE ? OR name LIKE ? OR city LIKE ?)"
)
pattern = f"%{search_text.strip()}%"
parameters.extend([pattern, pattern, pattern])
```

### تمرین ۲ — مرتب‌سازی پیش‌فرض بر اساس سن

در شروع برنامه، رکوردها بر اساس سن مرتب شوند.

**پاسخ:** در سازنده `StudentApp` مقدار `self.sort_by` را از `"name"` به `"age"` تغییر دهید.

### تمرین ۳ — فیلتر بر اساس شهر جدید

یک شهر جدید به فهرست `Combobox` اضافه کنید. سپس رکوردی با همان شهر بسازید و بررسی کنید که فیلتر کار می‌کند.

**پاسخ:** مقدار شهر را به `values` مربوط به `self.city_combo` اضافه کنید. در برنامه واقعی، بهتر است فهرست شهرها از پایگاه داده به دست آید، نه اینکه دستی نوشته شود.

### تمرین ۴ — شمارش نتایج

بررسی کنید چرا تعداد نمایش‌داده‌شده در `status_label` ممکن است کمتر از تعداد کل رکوردهای پایگاه داده باشد.

**پاسخ:** چون کوئری SQL قبل از بازگرداندن نتیجه، جست‌وجو و فیلتر شهر را اعمال می‌کند.

## ۱۰. خطاهای رایج

- **قرار دادن مستقیم متن کاربر در SQL:** از پارامترهای `?` استفاده کنید.
- **قرار دادن نام ستون از ورودی آزاد:** ستون مرتب‌سازی را از فهرست مجاز انتخاب کنید.
- **مرتب‌سازی عدد به‌عنوان متن:** ستون‌های عددی مانند سن و شناسه باید عددی مرتب شوند.
- **ساخت کوئری بدون توجه به پارامترها:** تعداد `?`ها باید با تعداد مقادیر پارامترها هماهنگ باشد.
- **فراموش کردن `refresh_table()`:** بعد از افزودن، ویرایش، حذف یا تغییر فیلتر، جدول را به‌روزرسانی کنید.
- **اجرای فایل اشتباه:** برنامه چندفایلی را از `main.py` اجرا کنید و همه فایل‌ها را در یک پوشه نگه دارید.

## ۱۱. جمع‌بندی

در این جلسه یاد گرفتیم جست‌وجو، فیلتر و مرتب‌سازی را به SQLite منتقل کنیم و منطق پایگاه داده را در `database.py` از رابط گرافیکی در `ui.py` جدا نگه داریم. این جداسازی، پایه مناسبی برای توسعه برنامه‌های بزرگ‌تر است.

## چک‌لیست مدرس

- [ ] تفاوت جست‌وجو در پایتون و جست‌وجو در SQL را توضیح دادم.
- [ ] `WHERE`، `LIKE` و `ORDER BY` را تمرین کردیم.
- [ ] اهمیت پارامترگذاری SQL را توضیح دادم.
- [ ] ساختار سه‌فایلی پروژه را اجرا کردیم.
- [ ] افزودن، ویرایش، حذف، جست‌وجو و مرتب‌سازی را آزمایش کردیم.

## جلسه بعد

در جلسه ۱۶ می‌توانیم برنامه را از نظر معماری و مدیریت خطا کامل‌تر کنیم: اعتبارسنجی متمرکز، نمایش خطاهای پایگاه داده، جداسازی تنظیمات و آماده‌سازی پروژه برای توسعه و تحویل.
