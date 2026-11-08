---
title: "Tkinter - جلسه 04 - Listbox و Combobox و Spinbox"
aliases:
  - "Tkinter Session 4"
  - "ویجت‌های فهرست و انتخاب در Tkinter"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
level: Beginner
status: Ready
language: Persian
session: 4
---

# جلسه ۴ — کار با Listbox، Combobox و Spinbox

> [!abstract] هدف جلسه
> در این جلسه یاد می‌گیریم فهرستی از گزینه‌ها نمایش دهیم، انتخاب کاربر را دریافت کنیم، از فهرست کشویی استفاده کنیم و ورودی‌های عددی محدودشده بسازیم. در پایان یک برنامه انتخاب دوره آموزشی پیاده‌سازی می‌کنیم.

## ۱. پیش‌نیازها

- ساخت پنجره با `Tk()`
- استفاده از `Label`، `Entry`، `Button` و `Frame`
- چیدمان با `pack()` و `grid()`
- استفاده از `StringVar`، `BooleanVar` و `messagebox`
- تعریف تابع و مدیریت ورودی‌های ساده

## ۲. آشنایی با `Listbox`

ویجت `Listbox` چند گزینه را به شکل فهرست نمایش می‌دهد و به کاربر اجازه می‌دهد یک یا چند مورد را انتخاب کند.

### مثال پایه

```python
import tkinter as tk

root = tk.Tk()
root.title("Listbox Example")
root.geometry("350x250")

tk.Label(root, text="Choose a programming language:").pack(pady=8)

language_list = tk.Listbox(root, height=5)
language_list.pack(padx=15, pady=5, fill="x")

languages = ["Python", "C#", "Java", "JavaScript", "C++"]

for language in languages:
    language_list.insert(tk.END, language)

def show_selection():
    selection = language_list.curselection()

    if not selection:
        result_label.config(text="No item selected.")
        return

    index = selection[0]
    selected_language = language_list.get(index)
    result_label.config(text=f"Selected: {selected_language}")

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

tk.Button(root, text="Show Selection", command=show_selection).pack(pady=5)

root.mainloop()
```

### متدها و گزینه‌های مهم `Listbox`

| متد یا گزینه | کاربرد |
|---|---|
| `insert(index, item)` | افزودن گزینه |
| `delete(first, last)` | حذف یک یا چند گزینه |
| `get(index)` | دریافت گزینه در یک اندیس |
| `curselection()` | دریافت اندیس گزینه‌های انتخاب‌شده |
| `selection_clear(first, last)` | پاک کردن انتخاب |
| `size()` | تعداد گزینه‌ها |
| `tk.END` | اشاره به انتهای فهرست |
| `selectmode` | تعیین شیوه انتخاب |

اندیس‌های `Listbox` از صفر شروع می‌شوند؛ بنابراین نخستین گزینه اندیس `0` دارد.

## ۳. انتخاب یک مورد یا چند مورد

به‌طور پیش‌فرض، `Listbox` معمولاً برای انتخاب یک مورد تنظیم می‌شود. برای اجازه دادن به انتخاب چند مورد می‌توان از `selectmode` استفاده کرد.

```python
import tkinter as tk

root = tk.Tk()
root.title("Multiple Selection")
root.geometry("350x280")

listbox = tk.Listbox(root, selectmode=tk.MULTIPLE, height=6)
listbox.pack(fill="x", padx=15, pady=10)

for item in ["Networking", "Programming", "Database", "Web Design", "Cybersecurity"]:
    listbox.insert(tk.END, item)

result_label = tk.Label(root, text="Select one or more courses.")
result_label.pack(pady=8)

def show_courses():
    indexes = listbox.curselection()
    selected_items = [listbox.get(index) for index in indexes]

    if not selected_items:
        result_label.config(text="No courses selected.")
        return

    result_label.config(text="Selected: " + ", ".join(selected_items))

tk.Button(root, text="Show Courses", command=show_courses).pack(pady=5)

root.mainloop()
```

نکته: با `selectmode=tk.MULTIPLE` کاربر می‌تواند چند گزینه را انتخاب یا از حالت انتخاب خارج کند. حالت‌های دیگر مانند `tk.SINGLE` و `tk.EXTENDED` نیز وجود دارند؛ رفتار دقیق انتخاب با ماوس و صفحه‌کلید ممکن است بسته به حالت متفاوت باشد.

## ۴. افزودن و حذف گزینه‌ها

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Manage List")
root.geometry("350x300")

entry = tk.Entry(root)
entry.pack(fill="x", padx=15, pady=8)

listbox = tk.Listbox(root, height=6)
listbox.pack(fill="both", expand=True, padx=15, pady=8)

for item in ["Python", "C#", "SQL"]:
    listbox.insert(tk.END, item)

def add_item():
    item = entry.get().strip()

    if not item:
        messagebox.showwarning("Missing Input", "Enter an item first.")
        return

    listbox.insert(tk.END, item)
    entry.delete(0, tk.END)

def delete_selected():
    selection = listbox.curselection()

    if not selection:
        messagebox.showwarning("No Selection", "Select an item to delete.")
        return

    listbox.delete(selection[0])

button_frame = tk.Frame(root)
button_frame.pack(pady=8)

tk.Button(button_frame, text="Add", command=add_item).grid(row=0, column=0, padx=5)
tk.Button(button_frame, text="Delete Selected", command=delete_selected).grid(row=0, column=1, padx=5)

root.mainloop()
```

در این مثال، تابع `delete_selected()` اولین گزینه انتخاب‌شده را حذف می‌کند. چون این `Listbox` برای انتخاب تک‌موردی است، بررسی `selection[0]` کافی است.

## ۵. آشنایی با `Combobox`

`Combobox` یک فهرست کشویی است که از ماژول `tkinter.ttk` می‌آید. برای گزینه‌های محدود مانند شهر، رشته تحصیلی یا نوع پرداخت بسیار مناسب است.

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Combobox Example")
root.geometry("350x180")

tk.Label(root, text="Choose your major:").pack(pady=8)

major_var = tk.StringVar()

major_combo = ttk.Combobox(
    root,
    textvariable=major_var,
    values=["Software", "Networking", "IT", "Accounting"],
    state="readonly"
)
major_combo.pack(padx=15, fill="x")

major_combo.current(0)

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

def show_major():
    result_label.config(text=f"Selected major: {major_combo.get()}")

tk.Button(root, text="Show Selection", command=show_major).pack(pady=5)

root.mainloop()
```

### گزینه‌های مهم `Combobox`

- `values`: فهرست گزینه‌ها
- `textvariable`: متغیر متصل به مقدار انتخاب‌شده
- `state="readonly"`: اجازه انتخاب از فهرست را می‌دهد اما تایپ آزاد را محدود می‌کند.
- `current(index)`: انتخاب گزینه با اندیس
- `get()`: دریافت مقدار فعلی
- `set(value)`: تعیین مقدار فعلی به شکل متنی

نمونه:

```python
major_combo.set("Networking")
print(major_combo.get())
```

اگر `state="normal"` باشد، کاربر می‌تواند علاوه بر انتخاب گزینه‌های موجود، متن دیگری نیز وارد کند. برای فهرست‌هایی با گزینه‌های مشخص معمولاً `readonly` مناسب‌تر است.

## ۶. واکنش به تغییر انتخاب با `bind`

گاهی لازم نیست کاربر روی دکمه جداگانه‌ای کلیک کند؛ می‌توان با تغییر انتخاب `Combobox` تابعی را اجرا کرد.

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("Combobox Event")
root.geometry("350x180")

course_var = tk.StringVar()
course_combo = ttk.Combobox(
    root,
    textvariable=course_var,
    values=["Python", "SQL", "Tkinter"],
    state="readonly"
)
course_combo.pack(padx=15, pady=10, fill="x")

result_label = tk.Label(root, text="Choose a course.")
result_label.pack(pady=8)

def on_course_selected(event):
    result_label.config(text=f"You selected: {course_combo.get()}")

course_combo.bind("<<ComboboxSelected>>", on_course_selected)

root.mainloop()
```

نکته: تابعی که با `bind()` به یک رویداد وصل می‌شود، معمولاً باید پارامتر `event` داشته باشد. در مقابل، تابعی که به `command` متصل می‌شود معمولاً بدون پارامتر رویداد تعریف می‌شود.

## ۷. آشنایی با `Spinbox`

`Spinbox` ورودی‌ای است که کاربر می‌تواند مقدار آن را با فلش‌های بالا و پایین تغییر دهد. برای تعداد، سن، امتیاز یا انتخاب عدد در یک بازه مناسب است.

```python
import tkinter as tk

root = tk.Tk()
root.title("Spinbox Example")
root.geometry("350x180")

tk.Label(root, text="Choose quantity:").pack(pady=8)

quantity_spinbox = tk.Spinbox(root, from_=1, to=20, width=10)
quantity_spinbox.pack(pady=5)

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

def show_quantity():
    result_label.config(text=f"Quantity: {quantity_spinbox.get()}")

tk.Button(root, text="Show Quantity", command=show_quantity).pack(pady=5)

root.mainloop()
```

نکته‌های مهم:

- `from_`: کمترین مقدار
- `to`: بیشترین مقدار
- `increment`: میزان افزایش یا کاهش هر مرحله
- `get()`: دریافت مقدار فعلی

مثال اعشاری:

```python
temperature_spinbox = tk.Spinbox(
    root,
    from_=-10,
    to=40,
    increment=0.5
)
```

مقدار `Spinbox` نیز در نهایت به شکل متن دریافت می‌شود. اگر قرار است محاسبه انجام دهیم، آن را به نوع مناسب تبدیل کنیم و ورودی را اعتبارسنجی کنیم.

## ۸. پروژه کلاسی: فرم انتخاب دوره آموزشی

### امکانات پروژه

- انتخاب نام دانشجو
- انتخاب دوره از `Combobox`
- انتخاب سطح از `Radiobutton`
- انتخاب خدمات جانبی با `Checkbutton`
- تعیین تعداد جلسات با `Spinbox`
- نمایش هزینه و خلاصه ثبت‌نام
- پاک کردن فرم با دکمه `Reset`

### قوانین قیمت‌گذاری نمونه

| دوره | هزینه پایه هر جلسه |
|---|---:|
| Python | 100 |
| SQL | 90 |
| Tkinter | 110 |
| Web Design | 95 |

هزینه هر جلسه بر اساس سطح افزایش پیدا می‌کند:

- Beginner: بدون افزایش
- Intermediate: ۲۰٪ افزایش
- Advanced: ۴۰٪ افزایش

خدمات جانبی:

- جزوه چاپی: 50
- پشتیبانی اضافه: 80

واحد قیمت در این پروژه نمونه است.

```python
import tkinter as tk
from tkinter import ttk, messagebox

root = tk.Tk()
root.title("Course Registration")
root.geometry("480x560")
root.resizable(False, False)

name_var = tk.StringVar()
course_var = tk.StringVar(value="Python")
level_var = tk.StringVar(value="Beginner")
printed_notes_var = tk.BooleanVar(value=False)
extra_support_var = tk.BooleanVar(value=False)
sessions_var = tk.IntVar(value=8)
result_var = tk.StringVar(value="Complete the form to calculate the fee.")

tk.Label(root, text="Course Registration", font=("Arial", 16, "bold")).pack(pady=10)

personal_frame = tk.LabelFrame(root, text="Student Information")
personal_frame.pack(fill="x", padx=15, pady=7)

tk.Label(personal_frame, text="Student name:").grid(row=0, column=0, padx=8, pady=8, sticky="w")
tk.Entry(personal_frame, textvariable=name_var).grid(
    row=0, column=1, padx=8, pady=8, sticky="ew"
)
personal_frame.columnconfigure(1, weight=1)

course_frame = tk.LabelFrame(root, text="Course")
course_frame.pack(fill="x", padx=15, pady=7)

tk.Label(course_frame, text="Select course:").grid(row=0, column=0, padx=8, pady=8, sticky="w")

course_combo = ttk.Combobox(
    course_frame,
    textvariable=course_var,
    values=["Python", "SQL", "Tkinter", "Web Design"],
    state="readonly"
)
course_combo.grid(row=0, column=1, padx=8, pady=8, sticky="ew")
course_frame.columnconfigure(1, weight=1)

level_frame = tk.LabelFrame(root, text="Level")
level_frame.pack(fill="x", padx=15, pady=7)

for level in ["Beginner", "Intermediate", "Advanced"]:
    tk.Radiobutton(
        level_frame,
        text=level,
        variable=level_var,
        value=level
    ).pack(anchor="w", padx=10)

options_frame = tk.LabelFrame(root, text="Additional Options")
options_frame.pack(fill="x", padx=15, pady=7)

tk.Checkbutton(
    options_frame,
    text="Printed notes (+50)",
    variable=printed_notes_var
).pack(anchor="w", padx=10)

tk.Checkbutton(
    options_frame,
    text="Extra support (+80)",
    variable=extra_support_var
).pack(anchor="w", padx=10)

sessions_frame = tk.Frame(root)
sessions_frame.pack(fill="x", padx=15, pady=7)

tk.Label(sessions_frame, text="Number of sessions:").pack(side="left")

sessions_spinbox = tk.Spinbox(
    sessions_frame,
    from_=1,
    to=40,
    textvariable=sessions_var,
    width=8
)
sessions_spinbox.pack(side="left", padx=10)

tk.Label(
    root,
    textvariable=result_var,
    wraplength=420,
    justify="center"
).pack(padx=15, pady=12)

def calculate_fee():
    student_name = name_var.get().strip()

    if not student_name:
        messagebox.showwarning("Missing Name", "Please enter the student name.")
        return

    try:
        sessions = int(sessions_spinbox.get())

        if sessions < 1 or sessions > 40:
            messagebox.showwarning("Invalid Sessions", "Sessions must be between 1 and 40.")
            return

    except ValueError:
        messagebox.showerror("Invalid Input", "Enter a valid number of sessions.")
        return

    base_prices = {
        "Python": 100,
        "SQL": 90,
        "Tkinter": 110,
        "Web Design": 95
    }

    level_multipliers = {
        "Beginner": 1.0,
        "Intermediate": 1.2,
        "Advanced": 1.4
    }

    price_per_session = base_prices[course_var.get()] * level_multipliers[level_var.get()]
    total = price_per_session * sessions

    if printed_notes_var.get():
        total += 50

    if extra_support_var.get():
        total += 80

    result_var.set(f"Estimated fee: {total:,.2f}")

    summary = (
        f"Student: {student_name}\n"
        f"Course: {course_var.get()}\n"
        f"Level: {level_var.get()}\n"
        f"Sessions: {sessions}\n"
        f"Estimated fee: {total:,.2f}"
    )
    messagebox.showinfo("Registration Summary", summary)

def reset_form():
    name_var.set("")
    course_var.set("Python")
    level_var.set("Beginner")
    printed_notes_var.set(False)
    extra_support_var.set(False)
    sessions_var.set(8)
    result_var.set("Complete the form to calculate the fee.")

tk.Button(root, text="Calculate Fee", command=calculate_fee).pack(pady=4)
tk.Button(root, text="Reset", command=reset_form).pack(pady=4)

root.mainloop()
```

## ۹. تحلیل پروژه

1. `ttk.Combobox` برای انتخاب دوره استفاده شده است.
2. `Radiobutton` سطح دوره را مشخص می‌کند.
3. `Checkbutton` خدمات جانبی را فعال یا غیرفعال می‌کند.
4. `Spinbox` تعداد جلسات را بین ۱ تا ۴۰ محدود می‌کند.
5. دیکشنری `base_prices` هزینه پایه هر جلسه را نگه می‌دارد.
6. دیکشنری `level_multipliers` ضریب سطح را مشخص می‌کند.
7. هزینه کل از ضرب هزینه هر جلسه در تعداد جلسات به دست می‌آید.
8. خدمات انتخاب‌شده به هزینه اضافه می‌شوند.
9. تابع `reset_form()` فرم را به حالت اولیه برمی‌گرداند.

## ۱۰. خطاهای رایج

### خطای ۱: فراموش کردن import مربوط به `ttk`

برای استفاده از `Combobox` باید آن را از `tkinter.ttk` وارد کنیم:

```python
from tkinter import ttk
```

### خطای ۲: اشتباه گرفتن `curselection()` با خود گزینه

`curselection()` اندیس‌ها را برمی‌گرداند، نه متن گزینه‌ها. برای گرفتن متن باید از `get(index)` استفاده کنیم.

```python
indexes = listbox.curselection()
if indexes:
    selected_item = listbox.get(indexes[0])
```

### خطای ۳: فرض کردن اینکه مقدار `Spinbox` عدد است

مقدار `get()` معمولاً رشته است؛ برای محاسبه آن را به `int` یا `float` تبدیل کنید.

### خطای ۴: استفاده از `bind()` بدون پارامتر `event`

در اتصال به رویدادهای `bind()`، تابع معمولاً باید یک پارامتر رویداد بپذیرد:

```python
def on_selected(event):
    print("Selection changed")
```

### خطای ۵: انتظار انتخاب از `Combobox` در حالت `readonly` با تایپ آزاد

وقتی `state="readonly"` است، کاربر باید گزینه‌ای از فهرست انتخاب کند. اگر ورود متن دلخواه لازم باشد، از حالت مناسب دیگری استفاده کنید.

## ۱۱. تمرین‌های دانشجو

### تمرین ۱ — فهرست کتاب‌ها

یک `Listbox` با پنج عنوان کتاب بسازید. دکمه‌ای اضافه کنید که عنوان انتخاب‌شده را در یک `Label` نشان دهد. اگر انتخابی انجام نشده است، هشدار نمایش دهید.

### تمرین ۲ — انتخاب شهر

یک `Combobox` با چند نام شهر بسازید. با تغییر انتخاب، نام شهر انتخاب‌شده بدون نیاز به دکمه در یک `Label` نمایش داده شود. از `bind("<<ComboboxSelected>>", ...)` استفاده کنید.

### تمرین ۳ — محاسبه تعداد کالا

با `Spinbox` تعداد کالا را از ۱ تا ۵۰ دریافت کنید. قیمت هر کالا ثابت باشد و مبلغ کل نمایش داده شود.

### تمرین ۴ — فهرست کارها

یک `Entry` برای وارد کردن کار، یک `Listbox` برای نمایش کارها و دو دکمه `Add` و `Delete Selected` بسازید. ورودی خالی و حذف بدون انتخاب را مدیریت کنید.

---

## ۱۲. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Book List")
root.geometry("350x280")

listbox = tk.Listbox(root, height=5)
listbox.pack(fill="x", padx=15, pady=10)

for book in ["Clean Code", "The Pragmatic Programmer", "Python Crash Course", "Automate the Boring Stuff", "Fluent Python"]:
    listbox.insert(tk.END, book)

result_label = tk.Label(root, text="")
result_label.pack(pady=5)

def show_book():
    selection = listbox.curselection()

    if not selection:
        messagebox.showwarning("No Selection", "Select a book first.")
        return

    result_label.config(text=f"Selected: {listbox.get(selection[0])}")

tk.Button(root, text="Show Book", command=show_book).pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.title("City Selector")
root.geometry("350x180")

city_var = tk.StringVar(value="Berlin")

city_combo = ttk.Combobox(
    root,
    textvariable=city_var,
    values=["Berlin", "Paris", "Rome", "Tokyo"],
    state="readonly"
)
city_combo.pack(padx=15, pady=10, fill="x")

result_label = tk.Label(root, text=f"Selected city: {city_var.get()}")
result_label.pack(pady=8)

def on_city_selected(event):
    result_label.config(text=f"Selected city: {city_combo.get()}")

city_combo.bind("<<ComboboxSelected>>", on_city_selected)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Item Cost")
root.geometry("350x180")

unit_price = 25

quantity_spinbox = tk.Spinbox(root, from_=1, to=50, width=10)
quantity_spinbox.pack(pady=10)

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

def calculate_cost():
    try:
        quantity = int(quantity_spinbox.get())
        total = quantity * unit_price
        result_label.config(text=f"Total cost: {total}")
    except ValueError:
        messagebox.showerror("Invalid Input", "Choose a valid quantity.")

tk.Button(root, text="Calculate", command=calculate_cost).pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("To-Do List")
root.geometry("380x350")

task_entry = tk.Entry(root)
task_entry.pack(fill="x", padx=15, pady=10)

task_list = tk.Listbox(root, height=10)
task_list.pack(fill="both", expand=True, padx=15, pady=5)

def add_task():
    task = task_entry.get().strip()

    if not task:
        messagebox.showwarning("Missing Task", "Enter a task first.")
        return

    task_list.insert(tk.END, task)
    task_entry.delete(0, tk.END)

def delete_task():
    selection = task_list.curselection()

    if not selection:
        messagebox.showwarning("No Selection", "Select a task to delete.")
        return

    task_list.delete(selection[0])

button_frame = tk.Frame(root)
button_frame.pack(pady=10)

tk.Button(button_frame, text="Add", command=add_task).grid(row=0, column=0, padx=5)
tk.Button(button_frame, text="Delete Selected", command=delete_task).grid(row=0, column=1, padx=5)

root.mainloop()
```

## ۱۳. جمع‌بندی و چک‌لیست

- [ ] ساخت فهرست با `Listbox`
- [ ] افزودن و حذف گزینه‌ها
- [ ] دریافت اندیس‌های انتخاب‌شده با `curselection()`
- [ ] دریافت متن گزینه با `get(index)`
- [ ] استفاده از `ttk.Combobox`
- [ ] محدود کردن انتخاب با `state="readonly"`
- [ ] واکنش به تغییر انتخاب با `bind()`
- [ ] دریافت مقدار عددی از `Spinbox`
- [ ] اعتبارسنجی ورودی و نمایش خطا
- [ ] پیاده‌سازی یک فرم انتخاب ساده

## ۱۴. آمادگی برای جلسه بعد

در جلسه پنجم، روی رویدادها و تعامل پیشرفته‌تر کار می‌کنیم: تفاوت `command` و `bind()`، رویدادهای صفحه‌کلید و ماوس، فعال و غیرفعال کردن ویجت‌ها و اعتبارسنجی بهتر ورودی‌ها.

پیوند پیشنهادی در Obsidian: `[[Tkinter - جلسه 05 - رویدادها و اعتبارسنجی]]`
