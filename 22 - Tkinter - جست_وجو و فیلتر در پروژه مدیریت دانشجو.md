---
title: "Tkinter - جلسه ۲۲ - جست‌وجو و فیلتر در پروژه مدیریت دانشجو"
aliases:
  - Tkinter Session 22
  - SQL Search and Filtering
tags:
  - Python
  - Tkinter
  - SQLite
  - Treeview
  - Search
level: intermediate
status: ready
---

# جلسه ۲۲ — جست‌وجو و فیلتر در پروژه مدیریت دانشجو

## هدف جلسه

در جلسه‌های قبل، ثبت، نمایش، ویرایش و حذف دانشجو را پیاده‌سازی کردیم. در این جلسه قابلیت جست‌وجو را به برنامه اضافه می‌کنیم تا کاربر بتواند بین رکوردها سریع‌تر به اطلاعات موردنظر برسد.

در پایان جلسه، دانشجو می‌تواند:

- از `LIKE` برای جست‌وجوی بخشی از متن استفاده کند.
- جست‌وجو را با پارامترهای SQL اجرا کند.
- نتایج جست‌وجو را در `Treeview` نمایش دهد.
- بین نمایش همه رکوردها و نمایش نتایج جست‌وجو جابه‌جا شود.
- با پاک‌کردن عبارت جست‌وجو، فهرست کامل را دوباره ببیند.
- وضعیت جدول و شناسه رکورد انتخاب‌شده را بعد از جست‌وجو مدیریت کند.

## ۱. جست‌وجو در SQL با `LIKE`

برای پیدا کردن رکوردهایی که نام یا کد دانشجویی آن‌ها شامل عبارت واردشده است، از `LIKE` استفاده می‌کنیم.

```sql
SELECT id, full_name, student_code, phone, course
FROM students
WHERE full_name LIKE ? OR student_code LIKE ?
ORDER BY full_name COLLATE NOCASE;
```

در پایتون می‌توان عبارت جست‌وجو را به این شکل آماده کرد:

```python
pattern = f"%{search_term}%"
```

علامت `%` در SQL به معنی صفر یا چند نویسه است. بنابراین `%ali%` می‌تواند عبارتی را پیدا کند که در بخشی از آن `ali` وجود داشته باشد.

ورودی کاربر باید به‌صورت پارامتر ارسال شود؛ آن را مستقیم به متن SQL متصل نکنید.

## ۲. افزودن تابع جست‌وجو به `database.py`

اگر تابع `search_students()` از جلسه ۱۹ در فایل شما وجود ندارد، این نسخه را اضافه کنید:

```python
def search_students(search_term):
    pattern = f"%{search_term.strip()}%"

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
```

این تابع فهرستی از دیکشنری‌ها برمی‌گرداند؛ ساختاری مشابه تابع `get_all_students()` که رابط کاربری قبلاً از آن استفاده می‌کرد.

**نکته اتصال:** `with connection:` تراکنش را مدیریت می‌کند اما اتصال را لزوماً نمی‌بندد. در نسخه نهایی پروژه، الگوی بستن اتصال را در تمام توابع `database.py` یکدست کنید.

## ۳. تغییر رابط کاربری

در این جلسه سه تغییر اصلی لازم است:

1. اضافه‌کردن ورودی جست‌وجو و دکمه‌های `Search` و `Show All`.
2. ساخت تابعی برای نمایش هر فهرست از دانشجویان در `Treeview`.
3. تغییر `load_students()` و افزودن `search_students()` برای انتخاب منبع داده.

برای جلوگیری از تکرار کد، بهتر است ساخت ردیف‌های جدول را در تابع جداگانه‌ای مانند `display_students()` قرار دهیم.

## ۴. کد کامل `main.py`

کد زیر نسخه یکپارچه پروژه تا این جلسه است. فایل `database.py` باید توابع `initialize_database`، `add_student`، `get_all_students`، `search_students`، `update_student` و `delete_student` را داشته باشد.

```python
import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox

import database


class StudentManagerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Management")
        self.root.geometry("1050x650")
        self.root.minsize(800, 520)

        self.selected_student_id = None

        self.full_name_var = tk.StringVar()
        self.student_code_var = tk.StringVar()
        self.phone_var = tk.StringVar()
        self.course_var = tk.StringVar()
        self.search_var = tk.StringVar()
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

        action_frame = ttk.Frame(main_frame)
        action_frame.pack(fill="x", pady=(0, 12))

        ttk.Button(
            action_frame,
            text="Add Student",
            command=self.add_student
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            action_frame,
            text="Update Selected",
            command=self.update_selected_student
        ).pack(side="left", padx=6)

        ttk.Button(
            action_frame,
            text="Delete Selected",
            command=self.delete_selected_student
        ).pack(side="left", padx=6)

        ttk.Button(
            action_frame,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="left", padx=6)

        search_frame = ttk.LabelFrame(
            main_frame,
            text="Search Students",
            padding=10
        )
        search_frame.pack(fill="x", pady=(0, 12))

        ttk.Label(search_frame, text="Name or Student Code:").pack(
            side="left", padx=(0, 8)
        )

        search_entry = ttk.Entry(
            search_frame,
            textvariable=self.search_var
        )
        search_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))
        search_entry.bind("<Return>", self.search_students)

        ttk.Button(
            search_frame,
            text="Search",
            command=self.search_students
        ).pack(side="left", padx=4)

        ttk.Button(
            search_frame,
            text="Show All",
            command=self.show_all_students
        ).pack(side="left", padx=4)

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
        self.tree.column("full_name", width=230)
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
        values = (
            self.full_name_var.get().strip(),
            self.student_code_var.get().strip(),
            self.phone_var.get().strip(),
            self.course_var.get().strip()
        )

        if not all(values):
            messagebox.showwarning(
                "Missing Information",
                "Please fill in all fields."
            )
            return None

        return values

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

        self.search_var.set("")
        self.load_students()
        self.clear_form()
        self.status_var.set(f"Student added. ID: {student_id}")

    def display_students(self, students):
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

        self.selected_student_id = None

    def load_students(self):
        try:
            students = database.get_all_students()
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "Student records could not be loaded."
            )
            return

        self.display_students(students)
        self.status_var.set(f"Showing all students: {len(students)}")

    def search_students(self, _event=None):
        search_term = self.search_var.get().strip()

        if not search_term:
            self.show_all_students()
            return

        try:
            students = database.search_students(search_term)
        except sqlite3.Error:
            messagebox.showerror(
                "Database Error",
                "The search could not be completed."
            )
            return

        self.display_students(students)
        self.clear_form(clear_search=False)
        self.status_var.set(f"Search results: {len(students)}")

        if not students:
            self.status_var.set("No matching students found.")

    def show_all_students(self):
        self.search_var.set("")
        self.load_students()

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
            self.show_all_students()
            self.clear_form()
            return

        updated_id = self.selected_student_id
        self.show_all_students()
        self.clear_form()
        self.status_var.set(f"Student {updated_id} updated successfully.")

    def delete_selected_student(self):
        if self.selected_student_id is None:
            messagebox.showwarning(
                "No Selection",
                "Select a student from the table first."
            )
            return

        if not messagebox.askyesno(
            "Confirm Deletion",
            "Are you sure you want to delete the selected student?"
        ):
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

        self.show_all_students()
        self.clear_form()

        if deleted:
            self.status_var.set(f"Student {student_id} deleted successfully.")
        else:
            self.status_var.set("The selected student was not found.")

    def clear_form(self, clear_search=True):
        self.selected_student_id = None
        self.full_name_var.set("")
        self.student_code_var.set("")
        self.phone_var.set("")
        self.course_var.set("")

        if clear_search:
            self.search_var.set("")

        for item_id in self.tree.selection():
            self.tree.selection_remove(item_id)


def main():
    database.initialize_database()

    root = tk.Tk()
    StudentManagerApp(root)
    root.mainloop()


if __name__ == "__main__":
    main()
```

## ۵. بررسی رفتار جست‌وجو

1. برنامه را اجرا کنید.
2. چند دانشجو با نام‌ها و کدهای متفاوت ثبت کنید.
3. بخشی از نام یکی از دانشجویان را در کادر جست‌وجو وارد کنید.
4. روی `Search` کلیک کنید.
5. فقط رکوردهای منطبق باید در جدول دیده شوند.
6. عبارت دیگری وارد کنید که هیچ نتیجه‌ای نداشته باشد.
7. پیام `No matching students found.` را بررسی کنید.
8. روی `Show All` کلیک کنید تا تمام رکوردها برگردند.

همچنین می‌توانید عبارت جست‌وجو را وارد کنید و کلید `Enter` را بزنید.

## ۶. چرا تابع `display_students()` مفید است؟

اگر منطق پرکردن `Treeview` هم در تابع بارگذاری همه رکوردها و هم در تابع جست‌وجو تکرار شود، تغییرات بعدی باید در چند جای برنامه اعمال شوند.

با تابع مشترک:

```python
self.display_students(students)
```

هر دو عملیات از یک روش برای نمایش نتیجه‌ها استفاده می‌کنند. این کار از تکرار کد جلوگیری می‌کند.

## ۷. نکته مهم درباره انتخاب رکورد

پس از جست‌وجو، محتوای جدول عوض می‌شود. به همین دلیل باید شناسه انتخاب‌شده قبلی پاک شود تا برنامه اشتباهاً رکورد قبلی را ویرایش یا حذف نکند.

در این نسخه، `display_students()` مقدار `self.selected_student_id` را روی `None` قرار می‌دهد و تابع جست‌وجو نیز فرم را پاک می‌کند.

## ۸. تمرین‌های کلاسی

### تمرین ۱ — جست‌وجو با کلید Enter

چگونه اجازه می‌دهیم کاربر پس از تایپ عبارت جست‌وجو، به‌جای کلیک روی دکمه، کلید Enter را فشار دهد؟

**پاسخ:**

```python
search_entry.bind("<Return>", self.search_students)
```

پارامتر `_event=None` در تعریف تابع باعث می‌شود تابع هم از طریق رویداد و هم از طریق دکمه قابل فراخوانی باشد.

### تمرین ۲ — جست‌وجو بر اساس شماره تماس

تابع جست‌وجو را طوری گسترش دهید که شماره تماس را هم بررسی کند.

**پاسخ:**

```python
WHERE full_name LIKE ?
   OR student_code LIKE ?
   OR phone LIKE ?
```

و پارامترها:

```python
(pattern, pattern, pattern)
```

### تمرین ۳ — نتیجه ناموجود

چرا بهتر است وقتی جست‌وجو نتیجه‌ای ندارد، پیام مناسب نشان دهیم؟

**پاسخ:** کاربر متوجه می‌شود عملیات انجام شده، اما رکوردی با عبارت واردشده پیدا نشده است؛ این وضعیت با خطای اجرای جست‌وجو فرق دارد.

### تمرین ۴ — پاک‌کردن عبارت جست‌وجو

دکمه‌ای با عنوان `Clear Search` اضافه کنید که عبارت جست‌وجو را پاک کند و همه دانشجویان را نمایش دهد.

**پاسخ:**

```python
def clear_search(self):
    self.search_var.set("")
    self.load_students()
```

سپس دکمه را به این تابع متصل کنید.

### تمرین ۵ — چرا پارامتری‌کردن SQL مهم است؟

**پاسخ:** چون ورودی کاربر جدا از متن SQL ارسال می‌شود و از ساختن مستقیم دستور SQL با اتصال رشته‌ها جلوگیری می‌کند.

## ۹. خطاهای رایج

- **جست‌وجو فقط با تطبیق کامل کار می‌کند:** برای جست‌وجوی بخشی از متن، الگوی `%term%` را به `LIKE` بدهید.
- **نتایج جست‌وجو در جدول دیده نمی‌شوند:** مطمئن شوید `display_students()` پس از دریافت نتیجه فراخوانی می‌شود.
- **با پاک‌شدن جدول، شناسه قبلی باقی می‌ماند:** بعد از تغییر مجموعه ردیف‌ها، انتخاب قبلی را پاک کنید.
- **فشردن Enter باعث خطا می‌شود:** تابع رویداد باید پارامتر رویداد را بپذیرد.
- **نمایش همه رکوردها کار نمی‌کند:** دکمه `Show All` باید عبارت جست‌وجو را پاک و `load_students()` را اجرا کند.
- **SQL با اتصال رشته‌ای ساخته می‌شود:** همیشه از پارامترهای `?` استفاده کنید.

## ۱۰. چک‌لیست مدرس

- [ ] تابع `search_students()` در `database.py` وجود دارد.
- [ ] کادر جست‌وجو به رابط اضافه شده است.
- [ ] جست‌وجو بر اساس نام و کد دانشجویی کار می‌کند.
- [ ] نتایج از طریق تابع مشترک `display_students()` نمایش داده می‌شوند.
- [ ] جست‌وجوی بدون نتیجه پیام مناسب دارد.
- [ ] دکمه `Show All` همه رکوردها را برمی‌گرداند.
- [ ] جست‌وجو با کلید Enter نیز کار می‌کند.
- [ ] انتخاب قبلی هنگام تغییر مجموعه نتایج پاک می‌شود.

## جلسه بعد

در جلسه ۲۳، پروژه را از نظر کیفیت کد و معماری بازبینی می‌کنیم: مسئولیت‌های `main.py`، `database.py` و `ui.py` را جدا می‌کنیم، تنظیمات را به برنامه وصل می‌کنیم و چند آزمون نهایی برای اطمینان از عملکرد صحیح می‌نویسیم.
