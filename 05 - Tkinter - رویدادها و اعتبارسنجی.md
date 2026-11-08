---
title: "Tkinter - جلسه 05 - رویدادها و اعتبارسنجی"
aliases:
  - "Tkinter Session 5"
  - "رویدادها و اعتبارسنجی ورودی در Tkinter"
tags:
  - Python
  - Tkinter
  - GUI
  - Teaching
level: Beginner-to-Intermediate
status: Ready
language: Persian
session: 5
---

# جلسه ۵ — رویدادها و اعتبارسنجی ورودی

> [!abstract] هدف جلسه
> در این جلسه با سیستم رویدادهای Tkinter آشنا می‌شویم، تفاوت `command` و `bind()` را یاد می‌گیریم، رویدادهای ماوس و صفحه‌کلید را مدیریت می‌کنیم و با روش‌های ساده و عملی اعتبارسنجی ورودی آشنا می‌شویم.

## ۱. رویداد چیست؟

در برنامه‌های گرافیکی، برنامه معمولاً منتظر می‌ماند تا کاربر کاری انجام دهد؛ برای مثال:

- روی دکمه کلیک کند.
- کلیدی از صفحه‌کلید را فشار دهد.
- ماوس را روی یک ویجت حرکت دهد.
- یک گزینه را از فهرست انتخاب کند.
- پنجره را ببندد.

هرکدام از این اتفاق‌ها می‌توانند یک **رویداد (Event)** باشند. برنامه می‌تواند تابعی را به رویداد متصل کند تا هنگام وقوع آن اجرا شود.

## ۲. تفاوت `command` و `bind()`

### روش اول: `command`

`command` برای اجرای تابع در پاسخ به عملیات مشخص یک ویجت، به‌خصوص کلیک روی `Button`، ساده و رایج است.

```python
import tkinter as tk

root = tk.Tk()
root.title("Command Example")
root.geometry("300x160")

def say_hello():
    result_label.config(text="Hello!")

tk.Button(root, text="Click Me", command=say_hello).pack(pady=15)

result_label = tk.Label(root, text="Click the button.")
result_label.pack(pady=5)

root.mainloop()
```

تابع را بدون پرانتز به `command` می‌دهیم؛ زیرا می‌خواهیم هنگام کلیک اجرا شود، نه هنگام ساخت دکمه.

### روش دوم: `bind()`

با `bind()` می‌توان یک تابع را به رویدادی مشخص متصل کرد. تابع رویداد معمولاً یک پارامتر به نام `event` دریافت می‌کند.

```python
import tkinter as tk

root = tk.Tk()
root.title("Bind Example")
root.geometry("350x180")

result_label = tk.Label(root, text="Click inside the window.")
result_label.pack(pady=40)

def handle_click(event):
    result_label.config(text=f"Mouse position: {event.x}, {event.y}")

root.bind("<Button-1>", handle_click)

root.mainloop()
```

در این مثال، کلیک چپ ماوس روی پنجره باعث اجرای `handle_click()` می‌شود. مقادیر `event.x` و `event.y` موقعیت ماوس را نسبت به ویجتی که رویداد به آن متصل شده است نشان می‌دهند.

### مقایسه

| ویژگی | `command` | `bind()` |
|---|---|---|
| کاربرد رایج | اجرای عملیات یک ویجت، مانند دکمه | اتصال به رویداد مشخص |
| دریافت شیء `event` | معمولاً خیر | معمولاً بله |
| مثال | `command=save_data` | `widget.bind("<Return>", handle_enter)` |
| انعطاف‌پذیری رویدادها | ساده و محدودتر | گسترده‌تر |

نکته: `command` و `bind()` جایگزین کامل یکدیگر نیستند؛ هرکدام برای کاربرد خاصی مناسب‌اند.

## ۳. رویدادهای رایج

| الگوی رویداد | معنی |
|---|---|
| `"<Button-1>"` | کلیک دکمه چپ ماوس |
| `"<Button-2>"` | کلیک دکمه میانی ماوس در سیستم‌هایی که آن را دارند |
| `"<Button-3>"` | کلیک دکمه راست ماوس |
| `"<Double-Button-1>"` | دوبار کلیک با دکمه چپ |
| `"<Enter>"` | ورود نشانگر ماوس به محدوده ویجت |
| `"<Leave>"` | خروج نشانگر ماوس از محدوده ویجت |
| `"<Motion>"` | حرکت نشانگر ماوس روی ویجت |
| `"<KeyPress>"` | فشردن یک کلید |
| `"<Return>"` | فشردن کلید Enter |
| `"<Escape>"` | فشردن کلید Escape |
| `"<FocusIn>"` | دریافت فوکوس توسط ویجت |
| `"<FocusOut>"` | از دست دادن فوکوس توسط ویجت |

رفتار بعضی رویدادهای ماوس، به‌ویژه شماره دکمه میانی، ممکن است بین سیستم‌عامل‌ها متفاوت باشد.

## ۴. مدیریت صفحه‌کلید

### اجرای تابع با فشردن Enter

```python
import tkinter as tk

root = tk.Tk()
root.title("Keyboard Event")
root.geometry("350x180")

name_entry = tk.Entry(root)
name_entry.pack(pady=12)

result_label = tk.Label(root, text="Type your name and press Enter.")
result_label.pack(pady=8)

def greet(event=None):
    name = name_entry.get().strip()

    if name:
        result_label.config(text=f"Hello, {name}!")

name_entry.bind("<Return>", greet)

root.mainloop()
```

در اینجا `event=None` باعث می‌شود تابع هم از طریق `bind()` و هم در صورت نیاز به شکل مستقیم و بدون آرگومان قابل فراخوانی باشد.

### تشخیص کلید فشرده‌شده

```python
import tkinter as tk

root = tk.Tk()
root.title("Key Detection")
root.geometry("350x180")

result_label = tk.Label(root, text="Press a key while the window is focused.")
result_label.pack(pady=40)

def show_key(event):
    result_label.config(text=f"Key pressed: {event.keysym}")

root.bind("<KeyPress>", show_key)

root.mainloop()
```

`event.keysym` نام نمادین کلید را نشان می‌دهد؛ مثلاً `Return` یا `Escape`. برای دریافت متن کلید در موارد مناسب می‌توان از `event.char` نیز استفاده کرد.

## ۵. رویدادهای ماوس و تغییر ظاهر ویجت

```python
import tkinter as tk

root = tk.Tk()
root.title("Mouse Events")
root.geometry("350x180")

status_label = tk.Label(
    root,
    text="Move the mouse over this label.",
    bg="lightgray",
    width=30,
    height=3
)
status_label.pack(pady=35)

def on_enter(event):
    status_label.config(bg="lightblue", text="Mouse entered")

def on_leave(event):
    status_label.config(bg="lightgray", text="Mouse left")

status_label.bind("<Enter>", on_enter)
status_label.bind("<Leave>", on_leave)

root.mainloop()
```

در این نمونه، `config()` تنظیمات ویجت را هنگام وقوع رویداد تغییر می‌دهد.

## ۶. فعال و غیرفعال کردن ویجت‌ها

ویجت‌هایی مانند `Button` و `Entry` دارای گزینه `state` هستند. این گزینه می‌تواند رفتار تعامل کاربر را تغییر دهد.

حالت‌های متداول:

- `"normal"`: ویجت فعال است.
- `"disabled"`: تعامل معمول کاربر با ویجت غیرفعال می‌شود.
- `"readonly"`: برای بعضی ویجت‌ها، مانند `ttk.Combobox`، امکان مشاهده و انتخاب محدود را فراهم می‌کند.

مثال:

```python
import tkinter as tk

root = tk.Tk()
root.title("Widget State")
root.geometry("350x220")

entry = tk.Entry(root)
entry.pack(pady=10)

def enable_entry():
    entry.config(state="normal")

def disable_entry():
    entry.config(state="disabled")

def clear_entry():
    entry.config(state="normal")
    entry.delete(0, tk.END)

button_frame = tk.Frame(root)
button_frame.pack(pady=8)

tk.Button(button_frame, text="Disable", command=disable_entry).grid(row=0, column=0, padx=4)
tk.Button(button_frame, text="Enable", command=enable_entry).grid(row=0, column=1, padx=4)
tk.Button(button_frame, text="Clear", command=clear_entry).grid(row=0, column=2, padx=4)

root.mainloop()
```

نکته: قبل از پاک کردن ورودی غیرفعال، آن را به حالت `normal` برمی‌گردانیم.

## ۷. اعتبارسنجی ورودی چیست؟

اعتبارسنجی یعنی بررسی کنیم داده واردشده برای کاری که می‌خواهیم انجام دهیم مناسب است یا نه.

نمونه‌ها:

- نام نباید خالی باشد.
- تعداد باید عدد صحیح مثبت باشد.
- سن باید در بازه منطقی قرار داشته باشد.
- ایمیل باید دست‌کم از نظر ساختار ساده بررسی شود.
- دو رمز عبور واردشده باید یکسان باشند.

اعتبارسنجی در رابط کاربری جای بررسی سمت سرور یا بررسی‌های امنیتی در سامانه‌های واقعی را نمی‌گیرد؛ اما از خطاهای معمول کاربر جلوگیری می‌کند.

## ۸. روش اول: اعتبارسنجی هنگام اجرای تابع

این روش برای برنامه‌های مقدماتی ساده و قابل فهم است.

```python
import tkinter as tk
from tkinter import messagebox

root = tk.Tk()
root.title("Input Validation")
root.geometry("350x200")

age_entry = tk.Entry(root)
age_entry.pack(pady=10)

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

def check_age():
    raw_age = age_entry.get().strip()

    if not raw_age:
        messagebox.showwarning("Missing Input", "Enter your age.")
        return

    try:
        age = int(raw_age)
    except ValueError:
        messagebox.showerror("Invalid Input", "Age must be a whole number.")
        return

    if age < 0 or age > 120:
        messagebox.showwarning("Invalid Age", "Enter an age between 0 and 120.")
        return

    result_label.config(text=f"Age accepted: {age}")

tk.Button(root, text="Check Age", command=check_age).pack(pady=5)

root.mainloop()
```

مراحل اعتبارسنجی:

1. مقدار ورودی را دریافت می‌کنیم.
2. خالی نبودن آن را بررسی می‌کنیم.
3. تبدیل نوع را داخل `try/except` انجام می‌دهیم.
4. محدودیت‌های منطقی را بررسی می‌کنیم.
5. فقط در صورت معتبر بودن، ادامه عملیات را انجام می‌دهیم.

## ۹. روش دوم: اعتبارسنجی هنگام تایپ با `validatecommand`

Tkinter امکان اعتبارسنجی در سطح `Entry` را نیز دارد. در این روش، تابع اعتبارسنجی هنگام تغییر متن اجرا می‌شود.

مثال: اجازه ورود فقط رقم‌ها و رشته خالی، تا کاربر بتواند ورودی را ویرایش یا پاک کند.

```python
import tkinter as tk

root = tk.Tk()
root.title("Live Validation")
root.geometry("350x180")

def validate_digits(proposed_value):
    return proposed_value.isdigit() or proposed_value == ""

validate_command = (root.register(validate_digits), "%P")

number_entry = tk.Entry(
    root,
    validate="key",
    validatecommand=validate_command
)
number_entry.pack(pady=15)

tk.Label(
    root,
    text="Digits only. The field may also be empty."
).pack(pady=5)

root.mainloop()
```

توضیح:

- `root.register(validate_digits)` تابع پایتون را برای استفاده در اعتبارسنجی ثبت می‌کند.
- `validate="key"` اعتبارسنجی را هنگام تغییر متن فعال می‌کند.
- `%P` به مقدار پیشنهادی جدید ورودی اشاره دارد؛ یعنی متنی که پس از تغییر قرار است در `Entry` باشد.
- تابع باید مقدار `True` یا `False` برگرداند.
- اجازه دادن به رشته خالی ضروری است تا کاربر بتواند محتوا را کامل پاک کند.

نکته آموزشی: اعتبارسنجی زنده برای محدودیت‌های ساده مانند فقط رقم مناسب است. برای بررسی‌های پیچیده‌تر، مانند بازه عددی یا ترکیب چند فیلد، معمولاً بررسی نهایی هنگام ثبت فرم نیز لازم است.

## ۱۰. پروژه کلاسی: فرم ثبت‌نام با اعتبارسنجی

### امکانات پروژه

- دریافت نام
- دریافت سن
- دریافت ایمیل
- انتخاب رشته با `Combobox`
- فعال شدن دکمه ثبت فقط پس از وارد شدن اطلاعات اولیه
- اعتبارسنجی نهایی هنگام ثبت
- نمایش خلاصه ثبت‌نام
- پاک کردن فرم

در این پروژه، فعال یا غیرفعال بودن دکمه بر اساس پر بودن فیلدهای اصلی کنترل می‌شود. بررسی نهایی همچنان در تابع ثبت انجام می‌شود.

```python
import tkinter as tk
from tkinter import ttk, messagebox

root = tk.Tk()
root.title("Student Registration")
root.geometry("450x380")
root.resizable(False, False)

name_var = tk.StringVar()
age_var = tk.StringVar()
email_var = tk.StringVar()
major_var = tk.StringVar(value="Software")
status_var = tk.StringVar(value="Enter the required information.")

tk.Label(root, text="Student Registration", font=("Arial", 15, "bold")).pack(pady=12)

form_frame = tk.Frame(root)
form_frame.pack(fill="x", padx=20)

tk.Label(form_frame, text="Name:").grid(row=0, column=0, padx=8, pady=8, sticky="w")
name_entry = tk.Entry(form_frame, textvariable=name_var)
name_entry.grid(row=0, column=1, padx=8, pady=8, sticky="ew")

tk.Label(form_frame, text="Age:").grid(row=1, column=0, padx=8, pady=8, sticky="w")
age_entry = tk.Entry(form_frame, textvariable=age_var)
age_entry.grid(row=1, column=1, padx=8, pady=8, sticky="ew")

tk.Label(form_frame, text="Email:").grid(row=2, column=0, padx=8, pady=8, sticky="w")
email_entry = tk.Entry(form_frame, textvariable=email_var)
email_entry.grid(row=2, column=1, padx=8, pady=8, sticky="ew")

tk.Label(form_frame, text="Major:").grid(row=3, column=0, padx=8, pady=8, sticky="w")
major_combo = ttk.Combobox(
    form_frame,
    textvariable=major_var,
    values=["Software", "Networking", "IT", "Database"],
    state="readonly"
)
major_combo.grid(row=3, column=1, padx=8, pady=8, sticky="ew")

form_frame.columnconfigure(1, weight=1)

def update_submit_state(*args):
    required_values = [
        name_var.get().strip(),
        age_var.get().strip(),
        email_var.get().strip()
    ]

    if all(required_values):
        submit_button.config(state="normal")
        status_var.set("Required fields are filled.")
    else:
        submit_button.config(state="disabled")
        status_var.set("Complete all required fields.")

def submit_form():
    name = name_var.get().strip()
    age_text = age_var.get().strip()
    email = email_var.get().strip()

    if not name:
        messagebox.showwarning("Missing Name", "Enter the student's name.")
        name_entry.focus_set()
        return

    try:
        age = int(age_text)
    except ValueError:
        messagebox.showerror("Invalid Age", "Age must be a whole number.")
        age_entry.focus_set()
        return

    if age < 1 or age > 120:
        messagebox.showwarning("Invalid Age", "Enter an age between 1 and 120.")
        age_entry.focus_set()
        return

    if "@" not in email or "." not in email.split("@")[-1]:
        messagebox.showwarning("Invalid Email", "Enter an email in a simple valid format.")
        email_entry.focus_set()
        return

    summary = (
        f"Name: {name}\n"
        f"Age: {age}\n"
        f"Email: {email}\n"
        f"Major: {major_var.get()}"
    )
    messagebox.showinfo("Registration Complete", summary)
    status_var.set("Registration data passed the basic checks.")

def reset_form():
    name_var.set("")
    age_var.set("")
    email_var.set("")
    major_var.set("Software")
    name_entry.focus_set()
    update_submit_state()

name_var.trace_add("write", update_submit_state)
age_var.trace_add("write", update_submit_state)
email_var.trace_add("write", update_submit_state)

button_frame = tk.Frame(root)
button_frame.pack(pady=12)

submit_button = tk.Button(
    button_frame,
    text="Submit",
    command=submit_form,
    state="disabled"
)
submit_button.grid(row=0, column=0, padx=6)

tk.Button(
    button_frame,
    text="Reset",
    command=reset_form
).grid(row=0, column=1, padx=6)

tk.Label(root, textvariable=status_var, wraplength=400).pack(padx=15, pady=5)

update_submit_state()
root.mainloop()
```

### تحلیل پروژه

1. `StringVar`ها مقدار فیلدها را نگه می‌دارند.
2. `trace_add("write", ...)` با تغییر مقدار متغیر، تابع `update_submit_state()` را اجرا می‌کند.
3. دکمه ثبت تا زمان پر شدن فیلدهای لازم غیرفعال است.
4. تابع `submit_form()` دوباره داده‌ها را بررسی می‌کند؛ غیرفعال بودن دکمه به‌تنهایی جای اعتبارسنجی را نمی‌گیرد.
5. سن به عدد صحیح تبدیل می‌شود و بازه آن بررسی می‌شود.
6. ایمیل با یک بررسی ساده ساختاری بررسی می‌شود؛ این روش اعتبارسنجی کامل ایمیل نیست.
7. `focus_set()` فوکوس را به فیلدی می‌برد که نیاز به اصلاح دارد.
8. تابع `reset_form()` فرم را به حالت اولیه برمی‌گرداند.

## ۱۱. خطاهای رایج

### خطای ۱: فراموش کردن پارامتر `event`

اشتباه:

```python
def handle_enter():
    print("Enter pressed")
```

در اتصال با `bind()`، این تابع باید رویداد را دریافت کند:

```python
def handle_enter(event):
    print("Enter pressed")
```

### خطای ۲: استفاده از پرانتز در `command`

اشتباه:

```python
button = tk.Button(root, text="Save", command=save_data())
```

در حالت معمول:

```python
button = tk.Button(root, text="Save", command=save_data)
```

### خطای ۳: اعتماد به بررسی دکمه به‌عنوان تنها اعتبارسنجی

حتی اگر دکمه ثبت غیرفعال باشد، تابع ثبت باید ورودی‌ها را بررسی کند؛ زیرا ممکن است وضعیت برنامه در آینده تغییر کند یا تابع از جای دیگری فراخوانی شود.

### خطای ۴: تبدیل عدد بدون `try/except`

اگر کاربر متن نامعتبر وارد کند، `int()` یا `float()` ممکن است `ValueError` ایجاد کند.

### خطای ۵: سخت‌گیری بیش از حد در اعتبارسنجی هنگام تایپ

اگر اعتبارسنجی اجازه خالی شدن فیلد را ندهد، ممکن است کاربر نتواند مقدار قبلی را به‌راحتی پاک کند و مقدار جدید وارد کند.

## ۱۲. تمرین‌های دانشجو

### تمرین ۱ — نمایش کلید

یک پنجره بسازید که با فشردن هر کلید، نام آن کلید را در یک `Label` نمایش دهد. از `bind("<KeyPress>", ...)` استفاده کنید.

### تمرین ۲ — دکمه فعال و غیرفعال

یک `Entry` و یک دکمه بسازید. تا وقتی ورودی خالی است، دکمه غیرفعال باشد. پس از وارد کردن متن، دکمه فعال شود. از `StringVar` و `trace_add()` استفاده کنید.

### تمرین ۳ — ورودی فقط عددی

یک `Entry` بسازید که هنگام تایپ فقط رقم‌ها را قبول کند و اجازه پاک کردن کامل محتوا را نیز بدهد. از `validatecommand` و `%P` استفاده کنید.

### تمرین ۴ — فرم محاسبه سن

سال تولد را از کاربر دریافت کنید. ورودی باید عدد صحیح باشد و در بازه‌ای منطقی قرار بگیرد. خطاها را با `messagebox` نمایش دهید و سن تقریبی را با استفاده از سال جاری محاسبه کنید.

---

## ۱۳. پاسخ تمرین‌ها

### پاسخ تمرین ۱

```python
import tkinter as tk

root = tk.Tk()
root.title("Key Viewer")
root.geometry("350x180")

result_var = tk.StringVar(value="Press any key.")

tk.Label(root, textvariable=result_var, font=("Arial", 14)).pack(pady=40)

def show_key(event):
    result_var.set(f"Key: {event.keysym}")

root.bind("<KeyPress>", show_key)
root.mainloop()
```

### پاسخ تمرین ۲

```python
import tkinter as tk

root = tk.Tk()
root.title("Enable Button")
root.geometry("350x180")

text_var = tk.StringVar()

entry = tk.Entry(root, textvariable=text_var)
entry.pack(pady=15)

result_label = tk.Label(root, text="")
result_label.pack(pady=5)

def submit():
    result_label.config(text=f"Submitted: {text_var.get().strip()}")

submit_button = tk.Button(root, text="Submit", command=submit, state="disabled")
submit_button.pack(pady=8)

def update_button(*args):
    if text_var.get().strip():
        submit_button.config(state="normal")
    else:
        submit_button.config(state="disabled")

text_var.trace_add("write", update_button)

root.mainloop()
```

### پاسخ تمرین ۳

```python
import tkinter as tk

root = tk.Tk()
root.title("Digits Only")
root.geometry("350x160")

def validate_digits(proposed_value):
    return proposed_value == "" or proposed_value.isdigit()

validate_command = (root.register(validate_digits), "%P")

number_entry = tk.Entry(
    root,
    validate="key",
    validatecommand=validate_command
)
number_entry.pack(pady=20)

tk.Label(root, text="Enter digits only.").pack(pady=5)

root.mainloop()
```

### پاسخ تمرین ۴

```python
import tkinter as tk
from tkinter import messagebox
from datetime import date

root = tk.Tk()
root.title("Age Calculator")
root.geometry("350x180")

birth_year_entry = tk.Entry(root)
birth_year_entry.pack(pady=12)

result_label = tk.Label(root, text="")
result_label.pack(pady=8)

def calculate_age():
    raw_year = birth_year_entry.get().strip()

    if not raw_year:
        messagebox.showwarning("Missing Input", "Enter your birth year.")
        return

    try:
        birth_year = int(raw_year)
    except ValueError:
        messagebox.showerror("Invalid Input", "Birth year must be a whole number.")
        return

    current_year = date.today().year

    if birth_year < 1900 or birth_year > current_year:
        messagebox.showwarning("Invalid Year", "Enter a valid birth year.")
        return

    approximate_age = current_year - birth_year
    result_label.config(text=f"Approximate age: {approximate_age}")

tk.Button(root, text="Calculate Age", command=calculate_age).pack(pady=5)

root.mainloop()
```

## ۱۴. جمع‌بندی و چک‌لیست

- [ ] مفهوم رویداد را توضیح می‌دهد.
- [ ] تفاوت `command` و `bind()` را می‌داند.
- [ ] رویدادهای ماوس و صفحه‌کلید را مدیریت می‌کند.
- [ ] از `event.keysym` و مختصات رویداد استفاده می‌کند.
- [ ] وضعیت ویجت را با `state` تغییر می‌دهد.
- [ ] ورودی‌ها را با `try/except` اعتبارسنجی می‌کند.
- [ ] کاربرد مقدماتی `validatecommand` و `%P` را می‌داند.
- [ ] از `trace_add()` برای واکنش به تغییر متغیر استفاده می‌کند.
- [ ] می‌داند که اعتبارسنجی هنگام ثبت نیز لازم است.

## ۱۵. آمادگی برای جلسه بعد

در جلسه ششم، ساخت چند پنجره‌ای را یاد می‌گیریم؛ شامل `Toplevel`، تفاوت آن با `Tk()`، انتقال اطلاعات بین پنجره‌ها، استفاده از `ttk` و یکپارچه‌سازی ظاهر فرم‌ها.

پیوند پیشنهادی در Obsidian: `[[Tkinter - جلسه 06 - چند پنجره‌ای و Toplevel]]`
