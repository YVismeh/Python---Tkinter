---
title: "Tkinter - جلسه ۱۴ - ویرایش رکوردها و تکمیل CRUD"
aliases:
  - Tkinter Session 14
  - SQLite Update CRUD
tags:
  - Python
  - Tkinter
  - SQLite
  - CRUD
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۴ — ویرایش رکوردها و تکمیل CRUD

## هدف جلسه

در جلسه قبل، برنامه مدیریت دانشجو را به SQLite وصل کردیم و امکان افزودن، نمایش و حذف اطلاعات را ساختیم. در این جلسه قابلیت ویرایش رکورد را اضافه می‌کنیم و چرخه کامل CRUD را مرور می‌کنیم.

CRUD مخفف چهار عملیات اصلی کار با داده‌هاست:

| حرف | عملیات | دستور رایج SQL |
|---|---|---|
| C | Create — ایجاد | `INSERT` |
| R | Read — خواندن | `SELECT` |
| U | Update — ویرایش | `UPDATE` |
| D | Delete — حذف | `DELETE` |

در پایان جلسه دانشجو باید بتواند رکورد انتخاب‌شده را از جدول بخواند، اطلاعات آن را در ورودی‌ها قرار دهد، آن را ویرایش کند و تغییرات را در SQLite ثبت کند.

## ۱. منطق ویرایش رکورد

ویرایش اطلاعات معمولاً این مراحل را دارد:

1. کاربر یک ردیف از `Treeview` انتخاب می‌کند.
2. برنامه شناسه رکورد را از ردیف انتخاب‌شده می‌خواند.
3. برنامه اطلاعات همان رکورد را از پایگاه داده دریافت می‌کند.
4. اطلاعات در ورودی‌های فرم قرار می‌گیرند.
5. کاربر مقادیر را تغییر می‌دهد.
6. با فشردن دکمه Update، برنامه داده‌ها را اعتبارسنجی می‌کند.
7. دستور `UPDATE` با شرط `WHERE id = ?` اجرا می‌شود.
8. جدول دوباره از پایگاه داده بارگذاری می‌شود.

**نکته کلیدی:** شناسه رکورد باید مشخص باشد تا فقط همان رکورد ویرایش شود. از نام دانشجو به‌عنوان شناسه استفاده نکنید؛ ممکن است دو نفر نام یکسان داشته باشند.

## ۲. دستور SQL برای ویرایش

```python
cursor.execute(
    """
    UPDATE students
    SET name = ?, age = ?, city = ?
    WHERE id = ?
    """,
    (name, age, city, student_id)
)
```

در این دستور، سه ستون تغییر می‌کنند و `WHERE id = ?` مشخص می‌کند کدام رکورد باید تغییر کند.

اگر شرط `WHERE` را حذف کنیم، ممکن است تمام رکوردهای جدول به مقادیر جدید تغییر کنند. به همین دلیل وجود شرط مناسب بسیار مهم است.

## ۳. انتخاب رکورد و قرار دادن اطلاعات در فرم

برای خواندن ردیف انتخاب‌شده از `Treeview` می‌توان از این الگو استفاده کرد:

```python
selection = self.tree.selection()

if not selection:
    messagebox.showinfo(
        "No Selection",
        "Select a student first."
    )
    return

student_id = int(selection[0])
```

در پروژه جلسه قبل، مقدار `iid` هر ردیف برابر شناسه پایگاه داده قرار داده شده بود:

```python
self.tree.insert(
    "",
    tk.END,
    iid=str(row[0]),
    values=row
)
```

بنابراین `selection[0]` شناسه ردیف انتخاب‌شده را در اختیار ما می‌گذارد.

سپس اطلاعات رکورد را از پایگاه داده می‌خوانیم:

```python
with sqlite3.connect(DATABASE_NAME) as connection:
    row = connection.execute(
        "SELECT id, name, age, city FROM students WHERE id = ?",
        (student_id,)
    ).fetchone()
```

`fetchone()` یک ردیف را برمی‌گرداند یا اگر رکوردی پیدا نشود، `None` را برمی‌گرداند.

## ۴. پروژه کامل: مدیریت دانشجو با CRUD

این نسخه شامل افزودن، نمایش، انتخاب، ویرایش و حذف رکورد است. اطلاعات در فایل `school.db` ذخیره می‌شوند و پس از بستن برنامه باقی می‌مانند.

فایل را با نام `student_crud_app.py` ذخیره کنید.

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


class StudentCRUDApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student CRUD Manager")
        self.root.geometry("800x540")
        self.root.minsize(680, 450)

        self.selected_student_id = None

        create_database()
        self.build_ui()
        self.load_students()

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
        self.age_entry.grid(
            row=0, column=3, padx=5, pady=5, sticky="ew"
        )

        ttk.Label(form, text="City:").grid(
            row=1, column=0, padx=5, pady=5, sticky="w"
        )
        self.city_entry = ttk.Entry(form)
        self.city_entry.grid(
            row=1, column=1, padx=5, pady=5, sticky="ew"
        )

        form.columnconfigure(1, weight=1)

        button_frame = ttk.Frame(form)
        button_frame.grid(
            row=1, column=2, columnspan=2, padx=5, pady=5, sticky="ew"
        )

        ttk.Button(
            button_frame,
            text="Add",
            command=self.add_student
        ).pack(side="left", padx=(0, 5))

        ttk.Button(
            button_frame,
            text="Load Selected",
            command=self.load_selected
        ).pack(side="left", padx=5)

        ttk.Button(
            button_frame,
            text="Update",
            command=self.update_student
        ).pack(side="left", padx=5)

        ttk.Button(
            button_frame,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="left", padx=5)

        table_frame = ttk.Frame(self.root)
        table_frame.pack(fill="both", expand=True, padx=12, pady=(0, 8))

        columns = ("id", "name", "age", "city")
        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            selectmode="browse"
        )

        headings = {
            "id": "ID",
            "name": "Name",
            "age": "Age",
            "city": "City"
        }

        for column, title in headings.items():
            self.tree.heading(column, text=title)

        self.tree.column("id", width=60, anchor="center")
        self.tree.column("name", width=220)
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

        self.tree.bind("<<TreeviewSelect>>", self.on_row_selected)

        bottom_frame = ttk.Frame(self.root)
        bottom_frame.pack(fill="x", padx=12, pady=(0, 12))

        ttk.Button(
            bottom_frame,
            text="Delete Selected",
            command=self.delete_student
        ).pack(side="left")

        self.status_label = ttk.Label(bottom_frame, text="")
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
            self.tree.insert(
                "",
                tk.END,
                iid=str(row[0]),
                values=row
            )

        self.status_label.config(text=f"{len(rows)} student(s)")

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

    def add_student(self):
        data = self.get_form_data()
        if data is None:
            return

        name, age, city = data

        with sqlite3.connect(DATABASE_NAME) as connection:
            connection.execute(
                "INSERT INTO students (name, age, city) VALUES (?, ?, ?)",
                (name, age, city)
            )

        self.load_students()
        self.clear_form()
        messagebox.showinfo("Saved", "Student added successfully.")

    def on_row_selected(self, event):
        selection = self.tree.selection()
        if not selection:
            return

        self.selected_student_id = int(selection[0])

    def load_selected(self):
        selection = self.tree.selection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student first."
            )
            return

        student_id = int(selection[0])

        with sqlite3.connect(DATABASE_NAME) as connection:
            row = connection.execute(
                """
                SELECT id, name, age, city
                FROM students
                WHERE id = ?
                """,
                (student_id,)
            ).fetchone()

        if row is None:
            messagebox.showwarning(
                "Not Found",
                "The selected student no longer exists."
            )
            self.load_students()
            return

        self.selected_student_id = row[0]

        self.name_entry.delete(0, tk.END)
        self.name_entry.insert(0, row[1])

        self.age_entry.delete(0, tk.END)
        self.age_entry.insert(0, str(row[2]))

        self.city_entry.delete(0, tk.END)
        self.city_entry.insert(0, row[3])

    def update_student(self):
        if self.selected_student_id is None:
            messagebox.showinfo(
                "No Student",
                "Select a row and load it before updating."
            )
            return

        data = self.get_form_data()
        if data is None:
            return

        name, age, city = data

        with sqlite3.connect(DATABASE_NAME) as connection:
            cursor = connection.execute(
                """
                UPDATE students
                SET name = ?, age = ?, city = ?
                WHERE id = ?
                """,
                (name, age, city, self.selected_student_id)
            )

        if cursor.rowcount == 0:
            messagebox.showwarning(
                "Not Found",
                "The student could not be found."
            )
            self.load_students()
            self.clear_form()
            return

        self.load_students()
        self.clear_form()
        messagebox.showinfo("Updated", "Student updated successfully.")

    def delete_student(self):
        selection = self.tree.selection()

        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student to delete."
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
        self.clear_form()

    def clear_form(self):
        self.name_entry.delete(0, tk.END)
        self.age_entry.delete(0, tk.END)
        self.city_entry.delete(0, tk.END)
        self.selected_student_id = None
        self.tree.selection_remove(self.tree.selection())


if __name__ == "__main__":
    root = tk.Tk()
    app = StudentCRUDApp(root)
    root.mainloop()
```

## ۵. بررسی جریان برنامه

### افزودن

1. کاربر فرم را پر می‌کند.
2. `get_form_data()` مقادیر را بررسی می‌کند.
3. دستور `INSERT` رکورد جدید را ثبت می‌کند.
4. `load_students()` جدول را به‌روزرسانی می‌کند.

### خواندن و انتخاب

- `load_students()` همه رکوردها را می‌خواند.
- انتخاب ردیف با `Treeview` انجام می‌شود.
- `load_selected()` اطلاعات کامل رکورد را از SQLite دریافت و در فرم قرار می‌دهد.

### ویرایش

- شناسه رکورد در `self.selected_student_id` نگهداری می‌شود.
- اطلاعات جدید اعتبارسنجی می‌شوند.
- `UPDATE ... WHERE id = ?` فقط همان رکورد را تغییر می‌دهد.

### حذف

- شناسه از ردیف انتخاب‌شده خوانده می‌شود.
- کاربر عملیات را تأیید می‌کند.
- `DELETE` رکورد مشخص را حذف می‌کند.

## ۶. تمرین‌های کلاسی

### تمرین ۱ — نمایش شناسه انتخاب‌شده

پس از انتخاب یک ردیف، شناسه آن را در `status_label` نمایش دهید.

**پاسخ پیشنهادی:** در `on_row_selected()` بعد از تعیین `self.selected_student_id` بنویسید:

```python
self.status_label.config(
    text=f"Selected student ID: {self.selected_student_id}"
)
```

### تمرین ۲ — پاک کردن فرم پس از حذف

پس از حذف رکورد، ورودی‌های فرم هم پاک شوند.

**پاسخ:** در انتهای `delete_student()` پس از `self.load_students()`، تابع `self.clear_form()` را فراخوانی کنید.

### تمرین ۳ — جلوگیری از سن غیرمجاز

ورودی سن را طوری اعتبارسنجی کنید که فقط عدد صحیح بین ۱ تا ۱۲۰ پذیرفته شود.

**پاسخ:** این بررسی در `get_form_data()` پیاده‌سازی شده است. توضیح دهید چرا تبدیل با `int()` باید داخل `try/except` قرار گیرد.

### تمرین ۴ — افزودن ستون ایمیل

ستون `email` را به جدول اضافه کنید و عملیات افزودن، خواندن و ویرایش را برای آن توسعه دهید.

**راهنمای پاسخ:** تغییر ساختار جدولی که قبلاً ساخته شده، فقط با تغییر `CREATE TABLE IF NOT EXISTS` انجام نمی‌شود. برای پایگاه داده موجود، از دستور `ALTER TABLE` یا روش مهاجرت مناسب استفاده کنید؛ سپس کوئری‌ها و ستون‌های Treeview را نیز به‌روزرسانی کنید.

## ۷. خطاهای رایج

- **به‌روزرسانی بدون شناسه:** برنامه باید بداند دقیقاً کدام رکورد تغییر می‌کند.
- **فراموش کردن `WHERE`:** در `UPDATE` و `DELETE` شرط مناسب ضروری است.
- **پاک کردن فرم قبل از ذخیره:** ابتدا عملیات پایگاه داده را انجام دهید و سپس فرم را پاک کنید.
- **نادیده گرفتن ورودی نامعتبر:** مقادیر باید پیش از اجرای SQL اعتبارسنجی شوند.
- **فرض اینکه `CREATE TABLE IF NOT EXISTS` ساختار موجود را تغییر می‌دهد:** این دستور فقط جدول را در صورت نبودن آن می‌سازد.
- **تفاوت شناسه و شماره ردیف:** ترتیب نمایش جدول ممکن است تغییر کند؛ از `id` واقعی رکورد استفاده کنید.

## ۸. جمع‌بندی و چک‌لیست مدرس

- [ ] چهار عملیات CRUD را توضیح دادم.
- [ ] انتخاب ردیف و بارگذاری اطلاعات در فرم را اجرا کردم.
- [ ] تفاوت شناسه رکورد و جایگاه ردیف را توضیح دادم.
- [ ] ویرایش را با `UPDATE` و شرط `WHERE` انجام دادم.
- [ ] حذف رکورد را با تأیید کاربر انجام دادم.
- [ ] برنامه را بستم و دوباره اجرا کردم تا ماندگاری اطلاعات بررسی شود.

## جلسه بعد

در جلسه ۱۵ جست‌وجو و مرتب‌سازی را مستقیماً با کوئری‌های SQLite پیاده‌سازی می‌کنیم؛ سپس درباره مدیریت خطاهای پایگاه داده و سازمان‌دهی کد در چند فایل صحبت خواهیم کرد.
