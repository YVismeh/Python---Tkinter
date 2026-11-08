---
title: "Tkinter - جلسه ۱۱ - جدول با ttk.Treeview"
aliases:
  - Tkinter Session 11
  - Treeview Table
tags:
  - Python
  - Tkinter
  - ttk
  - Treeview
  - آموزش
level: intermediate
status: ready
---

# جلسه ۱۱ — نمایش اطلاعات جدولی با `ttk.Treeview`

## هدف جلسه

در برنامه‌های مدیریتی، اطلاعات معمولاً به شکل جدول نمایش داده می‌شوند؛ برای مثال فهرست دانشجوها، محصولات، کتاب‌ها یا کارهای یک کاربر. در این جلسه با `ttk.Treeview` آشنا می‌شویم و یک جدول دانشجو می‌سازیم.

در پایان جلسه می‌توانیم:

- ستون‌های جدول را تعریف کنیم.
- ردیف اضافه کنیم.
- ردیف انتخاب‌شده را بخوانیم.
- ردیف را ویرایش یا حذف کنیم.
- برای جدول نوار پیمایش بسازیم.
- جدول را از داده‌های یک فهرست پایتون پر کنیم.

## ۱. `Treeview` چیست؟

`ttk.Treeview` ویجتی از ماژول `tkinter.ttk` است که می‌تواند اطلاعات را در قالب ردیف و ستون نمایش دهد. با اینکه نام آن Treeview است، می‌توان از آن به‌عنوان جدول ساده نیز استفاده کرد.

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Treeview Example")
root.geometry("600x300")

columns = ("name", "age", "city")

tree = ttk.Treeview(
    root,
    columns=columns,
    show="headings"
)

tree.heading("name", text="Name")
tree.heading("age", text="Age")
tree.heading("city", text="City")

tree.column("name", width=180)
tree.column("age", width=80, anchor="center")
tree.column("city", width=180)

tree.pack(fill="both", expand=True, padx=10, pady=10)

root.mainloop()
```

### نکته‌های مهم

- `columns` نام داخلی ستون‌ها را تعیین می‌کند.
- `show="headings"` ستون درختی پیش‌فرض را پنهان می‌کند و فقط سرستون‌های جدول را نشان می‌دهد.
- `heading()` عنوان نمایشی ستون را تنظیم می‌کند.
- `column()` عرض و نحوه تراز کردن محتوای ستون را تنظیم می‌کند.

نام داخلی ستون و عنوانی که کاربر می‌بیند لازم نیست یکسان باشند. برای مثال نام داخلی `student_id` می‌تواند عنوان نمایشی `Student ID` داشته باشد.

## ۲. افزودن ردیف با `insert()`

برای اضافه کردن ردیف از `insert()` استفاده می‌کنیم:

```python
tree.insert(
    "",
    tk.END,
    values=("Sara", 20, "Tehran")
)
```

- آرگومان اول، والد ردیف است. رشته خالی یعنی ردیف در سطح اصلی جدول قرار گیرد.
- `tk.END` یعنی ردیف در انتهای جدول قرار بگیرد.
- `values` مقادیر ستون‌ها را به همان ترتیب `columns` مشخص می‌کند.

چند ردیف نمونه:

```python
rows = [
    ("Sara", 20, "Tehran"),
    ("Amir", 22, "Tabriz"),
    ("Neda", 19, "Shiraz")
]

for row in rows:
    tree.insert("", tk.END, values=row)
```

## ۳. شناسه هر ردیف

هر ردیف در `Treeview` یک شناسه دارد که به آن `iid` گفته می‌شود. اگر خودمان شناسه بدهیم، بهتر می‌توانیم ردیف را پیدا کنیم:

```python
tree.insert(
    "",
    tk.END,
    iid="student_101",
    values=("Sara", 20, "Tehran")
)
```

شناسه باید در میان ردیف‌های موجود یکتا باشد. در پروژه‌های واقعی بهتر است شناسه پایدار دانشجو یا شناسه رکورد را به کار ببریم، نه صرفاً شماره ردیف نمایشی.

## ۴. انتخاب و خواندن ردیف

متد `selection()` شناسه ردیف‌های انتخاب‌شده را برمی‌گرداند:

```python
selected_items = tree.selection()

if selected_items:
    item_id = selected_items[0]
    values = tree.item(item_id, "values")
    print(values)
```

از آنجا که جدول در این مثال تک‌انتخابی است، معمولاً فقط یک شناسه داریم. با این حال، همیشه باید حالت انتخاب‌نشدن ردیف را مدیریت کنیم.

برای واکنش فوری به انتخاب ردیف می‌توانیم از رویداد استفاده کنیم:

```python
tree.bind("<<TreeviewSelect>>", on_select)
```

تابع رویداد باید با آرگومان اختیاری `event=None` تعریف شود.

## ۵. نوار پیمایش عمودی

وقتی ردیف‌ها زیاد شوند، نوار پیمایش مفید است.

```python
container = ttk.Frame(root)
container.pack(fill="both", expand=True, padx=10, pady=10)

scrollbar = ttk.Scrollbar(container, orient="vertical")
scrollbar.pack(side="right", fill="y")

tree = ttk.Treeview(
    container,
    columns=("name", "age", "city"),
    show="headings",
    yscrollcommand=scrollbar.set
)
tree.pack(side="left", fill="both", expand=True)

scrollbar.config(command=tree.yview)
```

دو اتصال لازم‌اند:

- `yscrollcommand=scrollbar.set`: جدول وضعیت پیمایش را به نوار اطلاع می‌دهد.
- `scrollbar.config(command=tree.yview)`: نوار پیمایش به جدول فرمان می‌دهد.

## ۶. پروژه کامل: مدیریت ساده دانشجوها

در این پروژه امکان افزودن دانشجو، انتخاب ردیف، نمایش اطلاعات انتخاب‌شده در ورودی‌ها، ویرایش و حذف دانشجو را داریم. داده‌ها در حافظه نگهداری می‌شوند؛ در جلسه‌های بعد می‌توانیم آن‌ها را به SQLite متصل کنیم.

```python
import tkinter as tk
from tkinter import messagebox, ttk


class StudentManager:
    def __init__(self, root):
        self.root = root
        self.root.title("Student Manager")
        self.root.geometry("760x500")
        self.root.minsize(620, 400)

        self.next_id = 1

        self._build_ui()

    def _build_ui(self):
        form = ttk.LabelFrame(
            self.root,
            text="Student Information",
            padding=10
        )
        form.pack(fill="x", padx=12, pady=12)

        ttk.Label(form, text="Name:").grid(
            row=0, column=0, sticky="w", padx=5, pady=5
        )
        self.name_entry = ttk.Entry(form)
        self.name_entry.grid(
            row=0, column=1, sticky="ew", padx=5, pady=5
        )

        ttk.Label(form, text="Age:").grid(
            row=0, column=2, sticky="w", padx=5, pady=5
        )
        self.age_entry = ttk.Entry(form, width=10)
        self.age_entry.grid(
            row=0, column=3, sticky="ew", padx=5, pady=5
        )

        ttk.Label(form, text="City:").grid(
            row=1, column=0, sticky="w", padx=5, pady=5
        )
        self.city_entry = ttk.Entry(form)
        self.city_entry.grid(
            row=1, column=1, sticky="ew", padx=5, pady=5
        )

        form.columnconfigure(1, weight=1)
        form.columnconfigure(3, weight=1)

        button_frame = ttk.Frame(form)
        button_frame.grid(
            row=2, column=0, columnspan=4, sticky="ew", pady=(8, 0)
        )

        ttk.Button(
            button_frame,
            text="Add Student",
            command=self.add_student
        ).pack(side="left", padx=(0, 6))

        ttk.Button(
            button_frame,
            text="Update Selected",
            command=self.update_student
        ).pack(side="left", padx=6)

        ttk.Button(
            button_frame,
            text="Delete Selected",
            command=self.delete_student
        ).pack(side="left", padx=6)

        ttk.Button(
            button_frame,
            text="Clear Form",
            command=self.clear_form
        ).pack(side="right")

        table_frame = ttk.LabelFrame(
            self.root,
            text="Students",
            padding=8
        )
        table_frame.pack(fill="both", expand=True, padx=12, pady=(0, 12))

        columns = ("id", "name", "age", "city")
        self.tree = ttk.Treeview(
            table_frame,
            columns=columns,
            show="headings",
            selectmode="browse"
        )

        self.tree.heading("id", text="ID")
        self.tree.heading("name", text="Name")
        self.tree.heading("age", text="Age")
        self.tree.heading("city", text="City")

        self.tree.column("id", width=60, anchor="center")
        self.tree.column("name", width=200)
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

        self.tree.bind("<<TreeviewSelect>>", self.on_select)

    def _read_form(self):
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
        except ValueError:
            messagebox.showwarning(
                "Invalid Age",
                "Age must be a whole number."
            )
            return None

        if age <= 0 or age > 120:
            messagebox.showwarning(
                "Invalid Age",
                "Please enter an age between 1 and 120."
            )
            return None

        return name, age, city

    def add_student(self):
        data = self._read_form()
        if data is None:
            return

        name, age, city = data
        item_id = f"student_{self.next_id}"
        self.next_id += 1

        self.tree.insert(
            "",
            tk.END,
            iid=item_id,
            values=(item_id, name, age, city)
        )

        self.clear_form()

    def on_select(self, event=None):
        selection = self.tree.selection()
        if not selection:
            return

        values = self.tree.item(selection[0], "values")
        if not values:
            return

        _, name, age, city = values

        self.name_entry.delete(0, tk.END)
        self.name_entry.insert(0, name)

        self.age_entry.delete(0, tk.END)
        self.age_entry.insert(0, age)

        self.city_entry.delete(0, tk.END)
        self.city_entry.insert(0, city)

    def update_student(self):
        selection = self.tree.selection()
        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student to update."
            )
            return

        data = self._read_form()
        if data is None:
            return

        name, age, city = data
        item_id = selection[0]

        self.tree.item(
            item_id,
            values=(item_id, name, age, city)
        )
        self.clear_form()

    def delete_student(self):
        selection = self.tree.selection()
        if not selection:
            messagebox.showinfo(
                "No Selection",
                "Select a student to delete."
            )
            return

        confirmed = messagebox.askyesno(
            "Delete Student",
            "Delete the selected student?"
        )
        if confirmed:
            self.tree.delete(selection[0])
            self.clear_form()

    def clear_form(self):
        self.name_entry.delete(0, tk.END)
        self.age_entry.delete(0, tk.END)
        self.city_entry.delete(0, tk.END)
        self.tree.selection_remove(self.tree.selection())
        self.name_entry.focus_set()


if __name__ == "__main__":
    root = tk.Tk()
    app = StudentManager(root)
    root.mainloop()
```

## ۷. بررسی توابع مهم پروژه

### `_read_form()`

این تابع اطلاعات ورودی‌ها را می‌خواند و اعتبارسنجی می‌کند. اگر اطلاعات ناقص یا نامعتبر باشد، `None` برمی‌گرداند. اگر اطلاعات معتبر باشد، یک tuple شامل نام، سن و شهر برمی‌گرداند.

به این ترتیب، کد اعتبارسنجی در `add_student()` و `update_student()` تکرار نمی‌شود.

### `on_select()`

وقتی کاربر یک ردیف را انتخاب می‌کند، اطلاعات همان ردیف داخل ورودی‌ها قرار می‌گیرد. این کار امکان ویرایش را ساده می‌کند.

### `tree.item()`

برای خواندن اطلاعات ردیف:

```python
values = self.tree.item(item_id, "values")
```

برای جایگزینی اطلاعات ردیف:

```python
self.tree.item(item_id, values=(item_id, name, age, city))
```

### `tree.delete()`

این متد ردیف را از جدول حذف می‌کند. چون پروژه فعلاً دیتابیس ندارد، حذف فقط در حافظه انجام می‌شود و با بسته شدن برنامه از بین می‌رود.

## ۸. تمرین‌های دانشجو

### تمرین ۱ — افزودن ستون رشته تحصیلی

یک ورودی برای رشته تحصیلی اضافه کنید و آن را به ستون جدید جدول تبدیل کنید.

**راهنمای پاسخ:**

- ورودی جدید را به فرم اضافه کنید.
- مقدار آن را در `_read_form()` بخوانید.
- tuple خروجی تابع را گسترش دهید.
- ستون جدید را به `columns`، `heading()`، `column()` و مقادیر `insert()` و `item()` اضافه کنید.

### تمرین ۲ — شمارش دانشجوها

یک دکمه بسازید که تعداد ردیف‌های جدول را نمایش دهد.

**پاسخ:**

```python
def show_student_count(self):
    count = len(self.tree.get_children())
    messagebox.showinfo(
        "Student Count",
        f"Number of students: {count}"
    )
```

دکمه را با `command=self.show_student_count` به رابط اضافه کنید.

### تمرین ۳ — پاک کردن فرم پس از انتخاب ردیف

اکنون با انتخاب یک ردیف، اطلاعات آن در ورودی‌ها ظاهر می‌شود. توضیح دهید چرا دکمه `Clear Form` باید علاوه بر پاک کردن ورودی‌ها، انتخاب جدول را هم پاک کند.

**پاسخ مورد انتظار:** اگر انتخاب باقی بماند، کاربر ممکن است تصور کند هنوز همان دانشجو در حالت انتخاب است و ناخواسته اطلاعات او را تغییر دهد. پاک کردن انتخاب، وضعیت رابط را روشن‌تر می‌کند.

### تمرین ۴ — تأیید قبل از ویرایش

پیش از به‌روزرسانی اطلاعات دانشجو، از کاربر تأیید بگیرید.

**پاسخ پیشنهادی:**

```python
confirmed = messagebox.askyesno(
    "Confirm Update",
    "Apply changes to this student?"
)

if not confirmed:
    return
```

این بخش را پس از اعتبارسنجی فرم و پیش از اجرای `self.tree.item(...)` قرار دهید.

## ۹. خطاهای رایج

- **فراموش کردن `show="headings"`:** در این صورت ممکن است ستون درختی پیش‌فرض نیز نمایش داده شود.
- **ناسازگاری ترتیب `values` و `columns`:** ترتیب مقادیر باید با ترتیب ستون‌ها هماهنگ باشد.
- **استفاده از شماره ردیف به جای شناسه:** برای ویرایش و حذف، از شناسه ردیف انتخاب‌شده استفاده کنید.
- **نادیده گرفتن انتخاب خالی:** `selection()` ممکن است tuple خالی برگرداند.
- **تبدیل نکردن سن به عدد:** ورودی‌های `Entry` رشته هستند؛ برای سن باید اعتبارسنجی انجام شود.
- **تصور ذخیره دائمی داده‌ها:** در این پروژه اطلاعات فقط در حافظه‌اند؛ برای ماندگاری باید ذخیره فایل یا دیتابیس اضافه شود.

## ۱۰. جمع‌بندی جلسه

- `ttk.Treeview` برای نمایش اطلاعات در قالب ردیف و ستون استفاده می‌شود.
- `heading()` عنوان ستون و `column()` ظاهر و عرض آن را تنظیم می‌کند.
- `insert()`، `item()` و `delete()` به‌ترتیب برای افزودن، خواندن/ویرایش و حذف ردیف استفاده می‌شوند.
- `selection()` شناسه ردیف‌های انتخاب‌شده را برمی‌گرداند.
- `Scrollbar` پیمایش جدول‌های بزرگ را ساده می‌کند.
- جداسازی اعتبارسنجی فرم در تابع مستقل، تکرار کد را کاهش می‌دهد.

## ۱۱. چک‌لیست مدرس

- [ ] جدول سه‌ستونه و چهارستونه را اجرا کردم.
- [ ] افزودن ردیف را آموزش دادم.
- [ ] خواندن ردیف انتخاب‌شده را نمایش دادم.
- [ ] ویرایش و حذف ردیف را تمرین کردیم.
- [ ] نوار پیمایش را به جدول وصل کردم.
- [ ] توضیح دادم داده‌های این پروژه هنوز دائمی نیستند.

## جلسه بعد

در جلسه ۱۲ می‌توانیم جست‌وجو و مرتب‌سازی جدول، انتخاب چند ردیف و نمایش جزئیات رکورد را بررسی کنیم. پس از آن، جدول را برای اتصال به SQLite آماده خواهیم کرد.
