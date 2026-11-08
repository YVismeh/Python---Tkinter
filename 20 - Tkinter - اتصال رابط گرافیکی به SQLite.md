---
title: "Tkinter - جلسه ۲۰ - اتصال رابط گرافیکی به SQLite"
aliases:
  - Tkinter Session 20
  - Student Manager GUI Integration
tags:
  - Python
  - Tkinter
  - SQLite
  - Treeview
  - آموزش
level: intermediate
status: ready
---

# جلسه ۲۰ — اتصال رابط گرافیکی به SQLite

## هدف جلسه

در جلسه ۱۹، لایه پایگاه داده را در فایل `database.py` پیاده‌سازی کردیم. در این جلسه یک رابط گرافیکی واقعی می‌سازیم که از همان توابع استفاده کند.

در پایان جلسه، دانشجو می‌تواند:

- پنجره اصلی پروژه را با Tkinter و `ttk` بسازد.
- فرم ورود اطلاعات دانشجو را طراحی کند.
- اطلاعات SQLite را در `ttk.Treeview` نمایش دهد.
- رکورد جدید را از طریق فرم ثبت کند.
- با انتخاب ردیف، اطلاعات آن را در فرم بارگذاری کند.
- بعد از تغییر اطلاعات، جدول را دوباره بارگذاری کند.
- خطاهای پایگاه داده را به پیام مناسب برای کاربر تبدیل کند.

> پیش‌نیاز: فایل `database.py` از جلسه قبل باید در پوشه پروژه موجود باشد و توابع `initialize_database`، `add_student` و `get_all_students` را داشته باشد.

## ۱. ساختار پروژه

```text
student_manager/
├── main.py
├── database.py
└── students.db
```

فایل `students.db` هنگام اجرای برنامه ساخته می‌شود. آن را به‌صورت دستی ایجاد نکنید.

در این جلسه برای تمرکز روی اتصال رابط و پایگاه داده، بخش رابط را در `main.py` قرار می‌دهیم. در مرحله بعد می‌توانیم آن را به `ui.py` و فایل‌های دیگر تقسیم کنیم.

## ۲. طراحی رابط کاربری

پنجره شامل چهار بخش است:

1. فرم اطلاعات: نام، کد دانشجویی، شماره تماس و دوره
2. دکمه‌های عملیاتی: افزودن، پاک‌کردن فرم و تازه‌سازی
3. جدول دانشجویان: نمایش اطلاعات ذخیره‌شده
4. نوار وضعیت: نمایش نتیجه عملیات

برای چیدمان کلی از `pack` و برای فیلدهای فرم از `grid` استفاده می‌کنیم. هرگز `pack` و `grid` را برای مدیریت فرزندان یک والد یکسان با هم ترکیب نکنید؛ اما می‌توان در والدهای متفاوت از هر دو استفاده کرد.

## ۳. کد کامل فایل `main.py`

کد زیر یک نسخه مستقل و قابل اجرا از رابط این جلسه است. فرض می‌کند فایل `database.py` از جلسه قبل در همان پوشه وجود دارد.

```python
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox

import database


class StudentManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Management")
        self.root.geometry("950x600")
        self.root.minsize(760, 480)

        self.full_name_var = tk.StringVar()
        self.student_code_var = tk.StringVar()
        self.phone_var = tk.StringVar()
        self.course_var = tk.StringVar()
        self.status_var = tk.StringVar(value="Ready")

        self.build_ui()
        self.load_students()

    def build_ui(self):
        main_frame = ttk.Frame(self.root, padding=16)
        main_frame.pack(fill="both", expand=True)

        title_label = ttk.Label(
            main_frame,
            text="Student Management",
            font=("TkDefaultFont", 16, "bold")
        )
        title_label.pack(anchor="w", pady=(0, 14))

        form_frame = ttk.LabelFrame(
            main_frame,
            text="Student Information",
            padding=12
        )
        form_frame.pack(fill="x", pady=(0, 12))

        ttk.Label(form_frame, text="Full Name:").grid(
            row=0, column=0, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.full_name_var,
            width=28
        ).grid(row=0, column=1, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Student Code:").grid(
            row=0, column=2, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.student_code_var,
            width=22
        ).grid(row=0, column=3, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Phone:").grid(
            row=1, column=0, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.phone_var,
            width=28
        ).grid(row=1, column=1, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Course:").grid(
            row=1, column=2, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.course_var,
            width=22
        ).grid(row=1, column=3, sticky="ew", padx=5, pady=5)

        form_frame.columnconfigure(1, weight=1)
        form_frame.columnconfigure(3, weight=1)

        button_frame = ttk.Frame(main_frame)
        button_frame.pack(fill="x", pady=(0, 12))

        ttk.Button(
            button_frame,
            text="Add Student",
            command=self.add_student
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            button_frame,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="left", padx=6)

        ttk.Button(
            button_frame,
            text="Refresh",
            command=self.load_students
        ).pack(side="left", padx=6)

        table_frame = ttk.LabelFrame(
            main_frame,
            text="Registered Students",
            padding=8
        )
        table_frame.pack(fill="both", expand=True)

        columns = (
            "id",
            "full_name",
            "student_code",
            "phone",
            "course"
        )

        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            height=12
        )

        self.tree.heading("id", text="ID")
        self.tree.heading("full_name", text="Full Name")
        self.tree.heading("student_code", text="Student Code")
        self.tree.heading("phone", text="Phone")
        self.tree.heading("course", text="Course")

        self.tree.column("id", width=55, anchor="center")
        self.tree.column("full_name", width=220)
        self.tree.column("student_code", width=130, anchor="center")
        self.tree.column("phone", width=150, anchor="center")
        self.tree.column("course", width=180)

        vertical_scrollbar = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.tree.yview
        )
        self.tree.configure(yscrollcommand=vertical_scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        vertical_scrollbar.pack(side="right", fill="y")

        self.tree.bind("<<TreeviewSelect>>", self.on_row_selected)

        status_label = ttk.Label(
            main_frame,
            textvariable=self.status_var,
            anchor="w"
        )
        status_label.pack(fill="x", pady=(10, 0))

    def validate_form(self):
        full_name = self.full_name_var.get().strip()
        student_code = self.student_code_var.get().strip()
        phone = self.phone_var.get().strip()
        course = self.course_var.get().strip()

        if not full_name or not student_code or not phone or not course:
            messagebox.showwarning(
                "Missing Information",
                "Please fill in all fields."
            )
            return None

        return full_name, student_code, phone, course

    def add_student(self):
        values = self.validate_form()
        if values is None:
            return

        try:
            student_id = database.add_student(*values)
        except sqlite3.IntegrityError:
            messagebox.showwarning(
                "Duplicate Student Code",
                "This student code is already registered."
            )
            return
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "The student could not be saved."
            )
            return

        self.load_students()
        self.clear_form()
        self.status_var.set(f"Student added successfully. ID: {student_id}")

    def load_students(self):
        try:
            students = database.get_all_students()
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "Student records could not be loaded."
            )
            return

        for item_id in self.tree.get_children():
            self.tree.delete(item_id)

        for student in students:
            self.tree.insert(
                "",
                "end",
                values=(
                    student["id"],
                    student["full_name"],
                    student["student_code"],
                    student["phone"],
                    student["course"]
                )
            )

        self.status_var.set(f"Loaded {len(students)} student(s).")

    def on_row_selected(self, _event):
        selected_items = self.tree.selection()
        if not selected_items:
            return

        values = self.tree.item(selected_items[0], "values")
        self.full_name_var.set(values[1])
        self.student_code_var.set(values[2])
        self.phone_var.set(values[3])
        self.course_var.set(values[4])
        self.status_var.set(f"Selected student ID: {values[0]}")

    def clear_form(self):
        self.full_name_var.set("")
        self.student_code_var.set("")
        self.phone_var.set("")
        self.course_var.set("")

        for item_id in self.tree.selection():
            self.tree.selection_remove(item_id)

        self.status_var.set("Form cleared.")


def main():
    database.initialize_database()

    root = tk.Tk()
    StudentManagerApp(root)
    root.mainloop()


if __name__ == "__main__":
    main()
```

## ۴. اجرای پروژه

در ترمینال وارد پوشه پروژه شوید و اجرا کنید:

```bash
python main.py
```

اگر در سیستم شما فرمان `python` در دسترس نیست، از این دستور استفاده کنید:

```bash
python3 main.py
```

### آزمون دستی

1. برنامه را اجرا کنید.
2. یک دانشجو با اطلاعات کامل ثبت کنید.
3. بررسی کنید که رکورد در جدول ظاهر شود.
4. برنامه را ببندید و دوباره اجرا کنید.
5. بررسی کنید که رکورد همچنان در جدول وجود داشته باشد.
6. یک کد دانشجویی تکراری وارد کنید و پیام خطا را ببینید.
7. فرم را خالی بگذارید و روی افزودن کلیک کنید.

## ۵. چگونه رابط کاربری به پایگاه داده وصل می‌شود؟

برای مثال، در تابع `add_student()` این خط اجرا می‌شود:

```python
student_id = database.add_student(*values)
```

در اینجا `values` یک تاپل شامل چهار مقدار اعتبارسنجی‌شده است. عملگر `*` مقادیر تاپل را به‌عنوان آرگومان‌های جداگانه به تابع می‌فرستد.

تابع `database.add_student()` مسئول اجرای دستور SQL است. کلاس رابط کاربری لازم نیست جزئیات `INSERT` را بداند.

## ۶. چگونه داده‌ها در Treeview نمایش داده می‌شوند؟

تابع `load_students()` ابتدا اطلاعات را از پایگاه داده می‌گیرد و سپس ردیف‌های قبلی جدول را پاک می‌کند:

```python
for item_id in self.tree.get_children():
    self.tree.delete(item_id)
```

بعد، هر دانشجو به یک ردیف جدید تبدیل می‌شود:

```python
self.tree.insert(
    "",
    "end",
    values=(
        student["id"],
        student["full_name"],
        student["student_code"],
        student["phone"],
        student["course"]
    )
)
```

این روش باعث می‌شود جدول با آخرین وضعیت پایگاه داده هماهنگ شود. برای پروژه‌های بزرگ‌تر می‌توان به‌جای پاک‌کردن همه ردیف‌ها، فقط تغییرات لازم را اعمال کرد.

## ۷. چرا از `StringVar` استفاده می‌کنیم؟

`StringVar` متغیری است که می‌تواند به ویجت‌هایی مانند `Entry` متصل شود. برای مثال:

```python
self.full_name_var = tk.StringVar()

name_entry = ttk.Entry(
    form_frame,
    textvariable=self.full_name_var
)
```

برای خواندن مقدار:

```python
full_name = self.full_name_var.get()
```

برای تغییر مقدار:

```python
self.full_name_var.set("Alex Morgan")
```

این روش برای پاک‌کردن فرم یا بارگذاری اطلاعات یک ردیف انتخاب‌شده مفید است.

## ۸. نکات فنی و بهبودهای بعدی

- این نسخه قابلیت افزودن، نمایش و انتخاب رکورد را پیاده‌سازی می‌کند؛ ویرایش و حذف در جلسات بعد اضافه خواهند شد.
- `database.py` باید همان نام توابعی را داشته باشد که رابط کاربری فراخوانی می‌کند.
- اگر نام ستون یا ساختار جدول را تغییر دهید، توابع پایگاه داده و رابط کاربری باید با هم سازگار بمانند.
- `Treeview` برای نمایش داده است؛ اطلاعات اصلی در فایل SQLite نگهداری می‌شوند.
- شماره ردیف در `Treeview` را با شناسه پایگاه داده اشتباه نگیرید.
- در این جلسه، با انتخاب ردیف اطلاعات در فرم ظاهر می‌شوند؛ این کار به‌تنهایی رکورد را ویرایش نمی‌کند.

## ۹. تمرین‌های کلاسی

### تمرین ۱ — نمایش تعداد رکوردها

پس از بارگذاری جدول، تعداد دانشجویان را در نوار وضعیت نمایش دهید.

**پاسخ:** تعداد رکوردهای برگشتی از `database.get_all_students()` را با `len(students)` محاسبه کنید. کد نمونه در تابع `load_students()` همین کار را انجام می‌دهد.

### تمرین ۲ — تغییر عنوان پنجره

عنوان پنجره را به `Training Center - Student Manager` تغییر دهید.

**پاسخ:**

```python
self.root.title("Training Center - Student Manager")
```

### تمرین ۳ — جلوگیری از ثبت داده خالی

چرا قبل از فراخوانی تابع پایگاه داده، از `strip()` استفاده می‌کنیم؟

**پاسخ:** چون ورودی‌ای که فقط شامل فاصله است نباید به‌عنوان مقدار معتبر پذیرفته شود. `strip()` فاصله‌های ابتدا و انتهای رشته را حذف می‌کند.

### تمرین ۴ — به‌روزرسانی جدول

چرا پس از ثبت موفق دانشجو، `load_students()` را فراخوانی می‌کنیم؟

**پاسخ:** تا جدول دوباره از پایگاه داده خوانده شود و رکورد جدید در رابط کاربری نمایش داده شود.

### تمرین ۵ — مدیریت خطا

اگر کد دانشجویی تکراری باشد، چرا نباید برنامه ناگهان بسته شود؟

**پاسخ:** خطای قابل پیش‌بینی باید مدیریت شود و پیام قابل فهمی به کاربر نمایش داده شود؛ سپس برنامه باید قابل استفاده باقی بماند.

## ۱۰. خطاهای رایج

- **`ModuleNotFoundError: No module named 'database'`:** بررسی کنید فایل `database.py` کنار `main.py` باشد و نام آن دقیق باشد.
- **`AttributeError` برای یک تابع پایگاه داده:** نام تابع در `main.py` و `database.py` را مقایسه کنید.
- **خالی‌ماندن جدول:** بررسی کنید `database.initialize_database()` و `load_students()` اجرا شده باشند.
- **دیده‌نشدن تغییرات:** بعد از عملیات موفق، تابع بارگذاری جدول را اجرا کنید.
- **تکرار کد دانشجویی:** خطای `sqlite3.IntegrityError` را مدیریت کنید.
- **استفاده از `grid` و `pack` در یک والد:** هر مدیر چیدمان را برای فرزندان یک والد یکسان به‌صورت جداگانه به‌کار ببرید.

## ۱۱. چک‌لیست مدرس

- [ ] ساختار پروژه ایجاد شد.
- [ ] فرم اطلاعات دانشجو ساخته شد.
- [ ] `Treeview` و نوار پیمایش اضافه شدند.
- [ ] اطلاعات از SQLite خوانده و در جدول نمایش داده شدند.
- [ ] ثبت دانشجو از طریق رابط کاربری کار کرد.
- [ ] انتخاب ردیف، اطلاعات را در فرم نمایش داد.
- [ ] اعتبارسنجی فیلدهای خالی انجام شد.
- [ ] خطای کد دانشجویی تکراری مدیریت شد.
- [ ] ماندگاری اطلاعات پس از اجرای دوباره بررسی شد.

## جلسه بعد

در جلسه ۲۱، عملیات ویرایش و حذف را به رابط اضافه می‌کنیم. تمرکز اصلی روی نگهداری شناسه رکورد انتخاب‌شده، تأیید حذف، اجرای `UPDATE` و `DELETE` و تازه‌سازی صحیح جدول خواهد بود.
