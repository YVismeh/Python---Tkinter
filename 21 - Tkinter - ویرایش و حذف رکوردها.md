---
title: "Tkinter - جلسه ۲۱ - ویرایش و حذف رکوردها"
aliases:
  - Tkinter Session 21
  - Update and Delete Student Records
tags:
  - Python
  - Tkinter
  - SQLite
  - Treeview
  - CRUD
level: intermediate
status: ready
---

# جلسه ۲۱ — ویرایش و حذف رکوردها

## هدف جلسه

در جلسه قبل، فرم ثبت دانشجو و جدول `Treeview` را به SQLite متصل کردیم. در این جلسه قابلیت‌های ویرایش و حذف را اضافه می‌کنیم تا بخش اصلی عملیات CRUD کامل‌تر شود.

در پایان جلسه، دانشجو می‌تواند:

- شناسه واقعی رکورد انتخاب‌شده را نگه دارد.
- اطلاعات رکورد انتخاب‌شده را در فرم بارگذاری کند.
- رکورد را با دستور `UPDATE` تغییر دهد.
- پیش از حذف، از کاربر تأیید بگیرد.
- رکورد را با دستور `DELETE` حذف کند.
- پس از عملیات موفق، جدول و فرم را به‌درستی به‌روزرسانی کند.
- خطاهای رایج مانند کد دانشجویی تکراری را مدیریت کند.

## ۱. مفهوم مهم: شناسه رکورد

هر رکورد در جدول SQLite یک `id` دارد. برای ویرایش و حذف باید از همین شناسه استفاده کنیم، نه از شماره ردیف در جدول گرافیکی.

در این پروژه، هنگام انتخاب ردیف، مقدار `id` از اولین ستون `Treeview` خوانده می‌شود و در متغیر `self.selected_student_id` قرار می‌گیرد.

```python
self.selected_student_id = int(values[0])
```

این متغیر به برنامه می‌گوید کاربر قصد ویرایش یا حذف کدام رکورد را دارد.

## ۲. افزودن تابع‌های پایگاه داده

فایل `database.py` از جلسه ۱۹ باید توابع `update_student()` و `delete_student()` را داشته باشد. اگر آن‌ها را هنوز اضافه نکرده‌اید، این نسخه‌ها را به فایل اضافه کنید.

```python
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

**توضیح:** شرط `WHERE id = ?` ضروری است. بدون شرط مناسب، ممکن است چندین رکورد ناخواسته تغییر کنند یا حذف شوند.

> نکته اتصال: `with connection:` تراکنش را مدیریت می‌کند، اما اتصال را لزوماً نمی‌بندد. در پروژه نهایی، بهتر است الگوی مدیریت اتصال و بستن آن را در `database.py` یکدست کنید.

## ۳. کد کامل رابط کاربری `main.py`

کد زیر نسخه کامل‌تر برنامه جلسات ۱۹ تا ۲۱ است. فایل `database.py` باید کنار `main.py` قرار داشته باشد و توابع `initialize_database`، `add_student`، `get_all_students`، `update_student` و `delete_student` را ارائه کند.

```python
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox

import database


class StudentManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Management")
        self.root.geometry("1000x620")
        self.root.minsize(780, 500)

        self.selected_student_id = None

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

        ttk.Label(
            main_frame,
            text="Student Management",
            font=("TkDefaultFont", 16, "bold")
        ).pack(anchor="w", pady=(0, 14))

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
            textvariable=self.full_name_var
        ).grid(row=0, column=1, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Student Code:").grid(
            row=0, column=2, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.student_code_var
        ).grid(row=0, column=3, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Phone:").grid(
            row=1, column=0, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.phone_var
        ).grid(row=1, column=1, sticky="ew", padx=5, pady=5)

        ttk.Label(form_frame, text="Course:").grid(
            row=1, column=2, sticky="w", padx=5, pady=5
        )
        ttk.Entry(
            form_frame,
            textvariable=self.course_var
        ).grid(row=1, column=3, sticky="ew", padx=5, pady=5)

        form_frame.columnconfigure(1, weight=1)
        form_frame.columnconfigure(3, weight=1)

        buttons_frame = ttk.Frame(main_frame)
        buttons_frame.pack(fill="x", pady=(0, 12))

        ttk.Button(
            buttons_frame,
            text="Add Student",
            command=self.add_student
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            buttons_frame,
            text="Update Selected",
            command=self.update_selected_student
        ).pack(side="left", padx=6)

        ttk.Button(
            buttons_frame,
            text="Delete Selected",
            command=self.delete_selected_student
        ).pack(side="left", padx=6)

        ttk.Button(
            buttons_frame,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="left", padx=6)

        ttk.Button(
            buttons_frame,
            text="Refresh",
            command=self.load_students
        ).pack(side="left", padx=6)

        table_frame = ttk.LabelFrame(
            main_frame,
            text="Registered Students",
            padding=8
        )
        table_frame.pack(fill="both", expand=True)

        columns = ("id", "full_name", "student_code", "phone", "course")

        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings"
        )

        headings = {
            "id": "ID",
            "full_name": "Full Name",
            "student_code": "Student Code",
            "phone": "Phone",
            "course": "Course"
        }

        for column, heading in headings.items():
            self.tree.heading(column, text=heading)

        self.tree.column("id", width=60, anchor="center")
        self.tree.column("full_name", width=220)
        self.tree.column("student_code", width=140, anchor="center")
        self.tree.column("phone", width=150, anchor="center")
        self.tree.column("course", width=200)

        scrollbar = ttk.Scrollbar(
            table_frame,
            orient="vertical",
            command=self.tree.yview
        )
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        self.tree.bind("<<TreeviewSelect>>", self.on_row_selected)

        ttk.Label(
            main_frame,
            textvariable=self.status_var,
            anchor="w"
        ).pack(fill="x", pady=(10, 0))

    def get_form_values(self):
        full_name = self.full_name_var.get().strip()
        student_code = self.student_code_var.get().strip()
        phone = self.phone_var.get().strip()
        course = self.course_var.get().strip()

        if not all((full_name, student_code, phone, course)):
            messagebox.showwarning(
                "Missing Information",
                "Please fill in all fields."
            )
            return None

        return full_name, student_code, phone, course

    def add_student(self):
        values = self.get_form_values()
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
                "The student could not be added."
            )
            return

        self.load_students()
        self.clear_form()
        self.status_var.set(f"Student added. ID: {student_id}")

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
        self.selected_student_id = int(values[0])

        self.full_name_var.set(values[1])
        self.student_code_var.set(values[2])
        self.phone_var.set(values[3])
        self.course_var.set(values[4])

        self.status_var.set(
            f"Selected student ID: {self.selected_student_id}"
        )

    def update_selected_student(self):
        if self.selected_student_id is None:
            messagebox.showwarning(
                "No Selection",
                "Select a student from the table first."
            )
            return

        values = self.get_form_values()
        if values is None:
            return

        try:
            updated = database.update_student(
                self.selected_student_id,
                *values
            )
        except sqlite3.IntegrityError:
            messagebox.showwarning(
                "Duplicate Student Code",
                "Another student already uses this code."
            )
            return
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "The student could not be updated."
            )
            return

        if not updated:
            messagebox.showwarning(
                "Record Not Found",
                "The selected student no longer exists."
            )
            self.load_students()
            self.clear_form()
            return

        updated_id = self.selected_student_id
        self.load_students()
        self.clear_form()
        self.status_var.set(f"Student {updated_id} updated successfully.")

    def delete_selected_student(self):
        if self.selected_student_id is None:
            messagebox.showwarning(
                "No Selection",
                "Select a student from the table first."
            )
            return

        confirmed = messagebox.askyesno(
            "Confirm Deletion",
            "Are you sure you want to delete the selected student?"
        )
        if not confirmed:
            return

        student_id = self.selected_student_id

        try:
            deleted = database.delete_student(student_id)
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "The student could not be deleted."
            )
            return

        if not deleted:
            messagebox.showwarning(
                "Record Not Found",
                "The selected student no longer exists."
            )

        self.load_students()
        self.clear_form()

        if deleted:
            self.status_var.set(f"Student {student_id} deleted successfully.")

    def clear_form(self):
        self.selected_student_id = None
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

## ۴. آزمایش ویرایش

1. برنامه را اجرا کنید و یک دانشجو ثبت کنید.
2. روی ردیف او در جدول کلیک کنید.
3. اطلاعات باید در فیلدهای فرم نمایش داده شوند.
4. نام یا دوره را تغییر دهید.
5. روی `Update Selected` کلیک کنید.
6. بررسی کنید که همان رکورد به‌روزرسانی شده باشد و رکورد تکراری جدیدی ساخته نشده باشد.

### دستور SQL ویرایش

```sql
UPDATE students
SET full_name = ?, student_code = ?, phone = ?, course = ?
WHERE id = ?;
```

`WHERE id = ?` تعیین می‌کند کدام رکورد باید تغییر کند.

## ۵. آزمایش حذف

1. یک ردیف را انتخاب کنید.
2. روی `Delete Selected` کلیک کنید.
3. در پنجره تأیید، ابتدا `No` را انتخاب کنید و بررسی کنید که رکورد باقی بماند.
4. دوباره عملیات را انجام دهید و این بار `Yes` را انتخاب کنید.
5. بررسی کنید رکورد از جدول حذف شده باشد.

### دستور SQL حذف

```sql
DELETE FROM students
WHERE id = ?;
```

هرگز حذف رکورد را فقط بر اساس نام انجام ندهید؛ ممکن است چند دانشجو نام یکسان داشته باشند.

## ۶. چرا `rowcount` مهم است؟

توابع پایگاه داده برای ویرایش و حذف مقدار بولی برمی‌گردانند:

```python
return cursor.rowcount > 0
```

اگر رکوردی مطابق شرط پیدا و تغییر داده یا حذف شده باشد، نتیجه معمولاً `True` است؛ اگر رکوردی تحت تأثیر قرار نگرفته باشد، نتیجه `False` خواهد بود.

این نتیجه کمک می‌کند رابط کاربری پیام درست نشان دهد.

## ۷. نکات مهم در طراحی

- انتخاب رکورد، شناسه آن را در `selected_student_id` نگه می‌دارد.
- پس از پاک‌کردن فرم، شناسه انتخاب‌شده نیز باید پاک شود.
- پیش از ویرایش و حذف، انتخاب‌شدن رکورد را بررسی کنید.
- حذف باید پس از تأیید کاربر انجام شود.
- بعد از عملیات موفق، جدول را دوباره از پایگاه داده بخوانید.
- خطاهای `sqlite3.IntegrityError` و `sqlite3.Error` را مدیریت کنید.
- از ورودی کاربر در SQL به‌صورت پارامتری استفاده کنید.

## ۸. تمرین‌های کلاسی

### تمرین ۱ — جلوگیری از ویرایش بدون انتخاب

اگر کاربر پیش از انتخاب ردیف روی `Update Selected` کلیک کند، چه رفتاری باید داشته باشیم؟

**پاسخ:** اگر `self.selected_student_id is None` باشد، پیام `No Selection` نمایش داده شود و تابع بدون اجرای SQL پایان یابد.

### تمرین ۲ — چرا شناسه را ذخیره می‌کنیم؟

**پاسخ:** چون پس از بارگذاری مجدد جدول، ترتیب ردیف‌ها ممکن است تغییر کند. شناسه پایگاه داده مشخص می‌کند عملیات روی کدام رکورد انجام شود.

### تمرین ۳ — لغو حذف

اگر کاربر در پنجره تأیید گزینه `No` را بزند، چه اتفاقی باید بیفتد؟

**پاسخ:** تابع باید پیش از فراخوانی `database.delete_student()` پایان یابد و هیچ تغییری در پایگاه داده انجام نشود.

### تمرین ۴ — کد دانشجویی تکراری هنگام ویرایش

اگر دانشجو کد دانشجویی خود را به کدی تغییر دهد که متعلق به رکورد دیگری است، چه اتفاقی می‌افتد؟

**پاسخ:** قید `UNIQUE` خطای `sqlite3.IntegrityError` ایجاد می‌کند. برنامه باید پیام مناسب نشان دهد و رکورد قبلی را حفظ کند.

### تمرین ۵ — افزودن دکمه انتخاب‌زدایی

یک دکمه اضافه کنید که فرم را پاک کند و انتخاب جدول را نیز بردارد.

**پاسخ:** می‌توان دکمه را به تابع `self.clear_form` متصل کرد.

## ۹. خطاهای رایج

- **ویرایش رکورد اشتباه:** شناسه واقعی `id` را ذخیره و استفاده کنید.
- **حذف بدون تأیید:** قبل از حذف از `messagebox.askyesno()` استفاده کنید.
- **فراموش‌کردن پاک‌کردن شناسه انتخابی:** در `clear_form()` مقدار `selected_student_id` را روی `None` بگذارید.
- **استفاده از `UPDATE` بدون `WHERE`:** ممکن است تمام رکوردها تغییر کنند.
- **جدول قدیمی پس از عملیات:** پس از موفقیت، `load_students()` را فراخوانی کنید.
- **ثبت کد تکراری:** خطای `sqlite3.IntegrityError` را مدیریت کنید.

## ۱۰. چک‌لیست مدرس

- [ ] شناسه رکورد انتخاب‌شده نگهداری می‌شود.
- [ ] اطلاعات ردیف انتخابی در فرم بارگذاری می‌شود.
- [ ] ویرایش رکورد با `UPDATE ... WHERE id = ?` کار می‌کند.
- [ ] حذف با `DELETE ... WHERE id = ?` کار می‌کند.
- [ ] پیش از حذف، تأیید کاربر گرفته می‌شود.
- [ ] عملیات بدون انتخاب رکورد متوقف می‌شود.
- [ ] خطاهای کد دانشجویی تکراری مدیریت می‌شوند.
- [ ] جدول بعد از تغییرات تازه‌سازی می‌شود.

## جلسه بعد

در جلسه ۲۲، قابلیت جست‌وجو را به رابط گرافیکی اضافه می‌کنیم و نتیجه‌های SQL را در `Treeview` نمایش می‌دهیم. سپس جست‌وجو، تازه‌سازی و وضعیت فعلی جدول را با هم هماهنگ می‌کنیم.
